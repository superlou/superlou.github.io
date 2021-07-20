+++
title = "Embedded Cross-Compiled Test Driven Development with CGull"
date = 2021-07-18
[taxonomies]
tags = ["c", "embedded"]
+++

Testing bare-metal embedded C isn't straightforward. Someone will shout "stop using printf," and you'll yell back, "what's a printf?" They'll follow up with something about emulators. They're not wrong, but how do you efficiently regression test on the emulation target? Sure, you'll eventually have Requirements Based Tests (RBT) and "black box test procedures," but they are hardware-in-the-loop and tedious to run. A full RBT suite can take weeks to execute with a combination of manual and scripted tests. The pathway to happiness isn't simply a fast "edit-compile-run" development loop, but a "edit-compile-run-It Works and Didn't Break Something Else" cycle. Enter cross-compiled TDD.

<!-- more -->

# The Example Application

We have developed the following application for our hypothetical STM1769HC12 processor which is compiled inside a proprietary IDE that we must use per our approved tool-chain.

```c
#include "bsp.h"
#include "frame.h"

int main(void) {
  uint32_t discrete_inputs = 0;
  frame_init(10);

  while (1) {
    discrete_inputs = bsp_read_din();
    bsp_set_dout(0, discrete_inputs & (1 << 3) ? true : false);
    frame_wait_for_next();
  }

  return 0;
}
```

There is a timing mechanism that initializes a frame length to 10 milliseconds and delays the main loop so that the application logic runs every 10 milliseconds.

The Board Support Package (BSP) is responsible for reading a bank of discrete input circuits, with their values packed into a 32-bit wide value, 1 bit per channel. Logic polarity is already handled for us and we can trust that a bit being set means that the input circuit is "active." The value of discrete output 0 is then set based on discrete input channel 3.

Unfortunately, there is noise on the inputs, and the UX team doesn't like the flickering output this causes. There is now a requirement to debounce all inputs so that the state of an input must be held for 50 ms before it is considered changed.

You don't have access to hardware, and even if you did, verifying that you're debouncing for 50 ms and not 40 ms or 60 ms is going to require new test setups. They'll be built eventually, but for now, we want to not leave a time-bomb that is only caught during formal run-for-score.

# Test Driven Development

Test Driven Development simply means we write our test cases first, verify our current code fails these tests, then implement the simplest logic to make the test pass.

This is not to be confused with Development for Test, which I usually see used to mean developing software in such a way as to facilitate testing. This is important, too, but it misses the point of starting with the tests.

We're going to focus on testing a new debounce module. After talking with the developer responsible for the main loop logic, we come to an agreement on the interface.

```c
#include "bsp.h"
#include "frame.h"
#include "debouncer.h"

int main(void) {
  uint32_t discrete_inputs = 0;
  uint32_t debounced_inputs = 0;
  frame_init(10);

  Debouncer debouncer;
  debouncer_new(&debouncer, 5);

  while (1) {
    discrete_inputs = bsp_read_din();
    debounced_inputs = debouncer_update(&debouncer, discrete_inputs);
    bsp_set_dout(0, debounced_inputs & (1 << 3) ? true : false);
    frame_wait_for_next();
  }

  return 0;
}
```

We're pretty sure we'll need to keep track of some kind of state, since debouncing requires knowledge of past values. We'll need to check that each input has been active for 5 frames (50 ms ÷ 10 ms). Wihtout significantly affecting the flow of the main loop, we insert a `debouncer_update` function in between reading the raw discrete inputs and using that data to set outputs.

Before we write any code for the implementation of `debouncer_new` and `debouncer_update`, we're going to write tests.

# Unity and CGull

The [Unity Test Framework](https://www.throwtheswitch.org/unity) is an open source library that lets you write `TEST_ASSERT_EQUAL_INT(5, some_variable)` and receive a report in a terminal if the test passes or fails. Unity is only a couple source and header files, and is as at home running directly on bare-metal embedded (with `putchar` defined) as it is on a desktop.

Since we don't have hardware available to test on, we are going to wrap Unity with [CGull](https://github.com/superlou/cgull). CGull includes the Unity libraries, and uses Python and GCC to build source files in a directory separate from the tests. It also can compile in test "helpers" to provide stubs to functions that would normally be included from elsewhere in the embedded project.

CGull compiles and runs the application source files locally, relying on the assumption that most C code that works on a desktop will behave the same on the microcontroller. This sounds like a bold assumption, but so long as the architecture has the same data widths, it's not bad. We can also segregate our modules to avoid hardware-dependent functions. Note how in this example, nothing in the debounce logic needs to understand an embedded environment: integers go in and integers come out.

# Envrionment Setup

We have two folders, one for our existing embedded application, and another for our tests.

```
embedded_project/
 ├ config.proj
 ├ bsp.c
 ├ bsp.h
 ├ debouncer.c
 ├ debouncer.h
 ├ main.c
 └ main.h
embedded_project_cgull_tests/
 ├ helpers/
 ├ tests/
 └ config.toml
```

In `embedded_project_cgull_tests/config.toml`, we specify where the project source code is and add whatever gcc flags we need to make the compiled code reasonably similar to the embedded processor. We also need to define a test for our debouncer module.

```
target_inc = "../embedded_project"
target_src = "../embedded_project"
gcc_flags = ["-m32"]

[test.debouncer]
depends = ["debouncer.c"]
```

CGull will automatically look for a file named `test_debouncer.c` in the tests folder based on the section name `[test.debouncer]`. The bare minimum for this file is:

```c
#include "unity.h"
#include "debouncer.h"

int main() {
  UNITY_BEGIN();
  UNITY_END();
  return 0;
}
```

Each module we are testing will be compiled into an executable. Running `cgull` from the `embedded_project_cgull_tests` directory should result in:

```
$ cgull

-----------------------
0 Tests 0 Failures 0 Ignored
OK
```

We haven't written any tests yet, so this isn't surprising. Lets start with a simple test that gives a single input changing over time, and expects the debounced output for all inputs to go high after the fifth occurrence.

```c
// test_debouncer.c
#include "unity.h"
#include "debouncer.h"

void test_basic_debouncing(void) {
  Debouncer debouncer;
  debouncer_new(&debouncer, 5);

  uint32_t in = 0xffffffff;

  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, in));
}

int main() {
  UNITY_BEGIN();
  RUN_TEST(test_basic_debouncing);
  UNITY_END();
  return 0;
}
```

Running cgull again, the output reminds us that we haven't written any implementation for our debounce module.

```
$ cgull
tests/test_debouncer.c: In function ‘test_basic_debouncing’:
tests/test_debouncer.c:5:3: error: unknown type name ‘Debouncer’
    5 |   Debouncer debouncer;
      |   ^~~~~~~~~
tests/test_debouncer.c:6:3: warning: implicit declaration of function ‘debouncer_new’ [-Wimplicit-function-declaration]
    6 |   debouncer_new(&debouncer, 0);
      |   ^~~~~~~~~~~~~
In file included from /home/lsimons/workspace/cgull/cgull/unity/unity.h:16,
                 from tests/test_debouncer.c:1:
tests/test_debouncer.c:12:40: warning: implicit declaration of function ‘debouncer_update’ [-Wimplicit-function-declaration]
   12 |   TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
      |                                        ^~~~~~~~~~~~~~~~
```

We now write code until the errors go away.

```c
// debouncer.h
#ifndef DEBOUNCER_H_
#define DEBOUNCER_H_

#include <stdint.h>

typedef struct {
} Debouncer;

void debouncer_new(Debouncer *debouncer, uint8_t threshold);
uint32_t debouncer_update(Debouncer *debouncer, uint32_t input);

#endif

// debouncer.c
#include "debouncer.h"

void debouncer_new(Debouncer *debouncer, uint8_t threshold) {

}

uint32_t debouncer_update(Debouncer *debouncer, uint32_t input) {

}
```

The cgull result is:

```
$ cgull
tests/test_debouncer.c:12:test_basic_debouncing:FAIL: Expected 0 Was 1449119692

-----------------------
1 Tests 1 Failures 0 Ignored
FAIL
```

We can compile successfully, but we haven't implemented any debouncing logic, so the single test case fails. Our first stab will be to use a counter for each input bit and simply increment until the bit has been set the right number of times.

```c
// debouncer.h
...
typedef struct {
  uint8_t count[32];
  uint32_t threshold;
} Debouncer;
...

// debouncer.c
#include <stdbool.h>
#include "debouncer.h"

#define NUM_DEBOUNCE_INPUTS 32

void debouncer_new(Debouncer *debouncer, uint8_t threshold) {
  uint32_t i;

  for (i = 0; i < NUM_DEBOUNCE_INPUTS; i++) {
    debouncer->count[i] = 0;
  }

  debouncer->threshold = threshold;
}

uint32_t debouncer_update(Debouncer *debouncer, uint32_t input) {
  uint32_t i;
  uint32_t result = 0;
  bool input_is_active;

  for (i = 0; i < NUM_DEBOUNCE_INPUTS; i++) {
    input_is_active = (input & (1 << i)) != 0;

    if (input_is_active) {
      debouncer->count[i]++;
    }

    if (debouncer->count[i] >= debouncer->threshold) {
      result += (1 << i);
    }
  }

  return result;
}
```

Now, when we run cgull we get:

```
$ cgull
tests/test_debouncer.c:19:test_basic_debouncing:PASS

-----------------------
1 Tests 0 Failures 0 Ignored
OK
```

We've done the bare minimum to pass the test. Well, it's not malicious bare minimum: we've identified that we need to treat each input separately. Lets add another test case to be sure.

```c
//test_debouncer.c
...
void test_debouncing_of_some_inputs(void) {
  Debouncer debouncer;
  debouncer_new(&debouncer, 5);

  uint32_t in = 0xAAAA5555;

  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0xAAAA5555, debouncer_update(&debouncer, in));
}

int main() {
  UNITY_BEGIN();
  RUN_TEST(test_basic_debouncing);
  RUN_TEST(test_debouncing_of_some_inputs);
  UNITY_END();
  return 0;
}
```

```
lsimons@bestever:~/workspace/cgull/examples/embedded_project_cgull_tests$ cgull
tests/test_debouncer.c:32:test_basic_debouncing:PASS
tests/test_debouncer.c:33:test_basic_debouncing_of_some_inputs:PASS

-----------------------
2 Tests 0 Failures 0 Ignored
OK
```

We haven't checked what happens when an input is deasserted. We expect (per hypothetical requirements) that the debounce is symmetrical. 5 frames of deasserted inputs should cause the debounced value to go to 0.

```c
// test_debouncer.c
...
void test_debouncing_on_and_off(void) {
  Debouncer debouncer;
  debouncer_new(&debouncer, 5);

  uint32_t in = 0xffffffff;

  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, 0));
}

int main() {
  UNITY_BEGIN();
  RUN_TEST(test_basic_debouncing);
  RUN_TEST(test_debouncing_of_some_inputs);
  RUN_TEST(test_debouncing_on_and_off);
  UNITY_END();
  return 0;
}
```

```
$ cgull
tests/test_debouncer.c:50:test_basic_debouncing:PASS
tests/test_debouncer.c:51:test_debouncing_of_some_inputs:PASS
tests/test_debouncer.c:45:test_debouncing_on_and_off:FAIL: Expected 0 Was 4294967295

-----------------------
3 Tests 1 Failures 0 Ignored
FAIL
```

And we're failing again. Since we're counting in both directions, we now need some state to keep track of which way each input was at.

```c
// debouncer.h
...
typedef struct {
  uint8_t count[32];
  uint8_t threshold;
  uint32_t state;
} Debouncer;
...

// debouncer.c
...
void debouncer_new(Debouncer *debouncer, uint8_t threshold) {
  uint32_t i;

  for (i = 0; i < NUM_DEBOUNCE_INPUTS; i++) {
    debouncer->count[i] = 0;
  }

  debouncer->threshold = threshold;
  debouncer->state = 0;
}

uint32_t debouncer_update(Debouncer *debouncer, uint32_t input) {
  uint32_t i;
  uint32_t new_state = 0;
  bool input_is_active;
  bool debounce_was_active;

  for (i = 0; i < NUM_DEBOUNCE_INPUTS; i++) {
    input_is_active = (input & (1 << i)) != 0;
    debounce_was_active = (debouncer->state & (1 << i)) != 0;

    if (input_is_active) {
      debouncer->count[i]++;
    } else {
      if (debouncer->count[i] > 0) {
        debouncer->count[i]--;
      }
    }

    if (debouncer->count[i] >= debouncer->threshold) {
      new_state += (1 << i);
    } else if (debounce_was_active && (debouncer->count[i] != 0)) {
      new_state += (1 << i);
    }
  }

  debouncer->state = new_state;
  return debouncer->state;
}
```

```
$ cgull
tests/test_debouncer.c:50:test_basic_debouncing:PASS
tests/test_debouncer.c:51:test_debouncing_of_some_inputs:PASS
tests/test_debouncer.c:52:test_debouncing_on_and_off:PASS

-----------------------
3 Tests 0 Failures 0 Ignored
OK
```

There are also some tests that are much easier to run in this fashion than on real hardware. We know that this code is time dependent, and we are suspicious about what happens if the inputs are in the same state for many calls. This would be incredibly time prohibitive in an emulator.

```c
// test_debouncer.c
void test_debouncing_for_a_while(void) {
  Debouncer debouncer;
  debouncer_new(&debouncer, 5);

  uint32_t in = 0xffffffff;

  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));
  TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, in));

  for (uint64_t i = 0; i < 10000; i++) {
    TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, in));
  }

  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));
  TEST_ASSERT_EQUAL_UINT32(0xffffffff, debouncer_update(&debouncer, 0));

  for (uint64_t i = 0; i < 10000; i++) {
    TEST_ASSERT_EQUAL_UINT32(0x00000000, debouncer_update(&debouncer, 0));
  }
}
```

```
$ cgull
tests/test_debouncer.c:75:test_basic_debouncing:PASS
tests/test_debouncer.c:76:test_debouncing_of_some_inputs:PASS
tests/test_debouncer.c:77:test_debouncing_on_and_off:PASS
tests/test_debouncer.c:60:test_debouncing_for_a_while:FAIL: Expected 4294967295 Was 0

-----------------------
4 Tests 1 Failures 0 Ignored
FAIL
```

This test exposed an issue when counting upwards. Just like we need to not count below zero, we want to limit the counter once it hits the threshold.

```c
...
// debouncer.c
uint32_t debouncer_update(Debouncer *debouncer, uint32_t input) {
...

  for (i = 0; i < NUM_DEBOUNCE_INPUTS; i++) {
    ...

    if (input_is_active) {
      if (debouncer->count[i] < debouncer->threshold) {
        debouncer->count[i]++;
      }
    } else {
      if (debouncer->count[i] > 0) {
        debouncer->count[i]--;
      }
    }

    ...
}
```

```
$ cgull
tests/test_debouncer.c:75:test_basic_debouncing:PASS
tests/test_debouncer.c:76:test_debouncing_of_some_inputs:PASS
tests/test_debouncer.c:77:test_debouncing_on_and_off:PASS
tests/test_debouncer.c:78:test_debouncing_for_a_while:PASS

-----------------------
4 Tests 0 Failures 0 Ignored
OK
```

You can create all kinds of tests around staggering the activation of inputs and watching for unexpected effects in the outputs. This process is the basic idea of TDD. As our test definition gets better, our code gets better. If a fix breaks an existing test in your suite, you'll know immediately.

# But You Made Fun of `printf`

One cool thing with this cross-compiled approach is that if you don't have a debugger available on the desktop, you can use `printf` in the module you are developing. Since it's compiled for your desktop, it will simply print in your terminal. Just remember to remove them when done.

# Do I Really Need CGull?

No. It's a half-baked Python module I threw together that composes `gcc` command-line arguments. You could use `make` or other tools. However, if you're not the only developer on a project, it's nice to use the same tool as everyone else. Agreeing on a framework like CGull makes it easier to collaborate.

# Regulatory Caveat

Sometimes regulatory standards have very strict specifications on what kind of tests can be used for credit, and how they can be developed in the software life cycle. The technique of cross-compilation used here pretty much guarantees you can't take credit for the tests executed. They don't run on the target hardware. They're not strictly traceable to formal requirements. They don't even generate the same instruction sets. So, if that's the industry you're in, think of this technique as a tool for code development and informal regression test. That said, you'll be surprised the bugs you find that the black-box verification team missed.
