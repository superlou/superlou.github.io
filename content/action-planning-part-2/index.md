+++
title = "Action Planning Part 2: Rewrite It In Rust"
date = 2021-10-20
[taxonomies]
tags = ["ai", "rust", "python"]
+++

Back in [Action Planning in Python](/action-planning-in-python), we wrote a simple A* based planner that could solve the problem of boiling water, given a scenario with initial conditions, actions the AI agent can take, and preconditions for those actions. The result was:

```
Path:
pick up pot
move to sink
turn on faucet
wait
turn off faucet
move to stove
put down pot
turn on stove
goal
```

In Part 2, we're going to rewrite it in Rust.

<!-- more -->

Why? One thing I noticed was that I was using a lot of [string-typing](https://wiki.c2.com/?StringlyTyped) to prototype something as fast as possible. I could do make objects and use Python type hints to cut down on my mistakes, but I want to see if I can write something as readable in Rust. Given the algorithm developed earlier was very functional, I think a lot of the implementation should map very cleanly. Additionally, the end goal of this series is going to be using the action planner in a game, and I'm planning to hook it into the Godot game engine via network calls. Creating a single executable planning server will make it easier to distribute.

# The State of the World

To represent the state of the world, we will hold everything in a `State` struct. The possible values of state parameters are strongly typed using enums.

```rust
#[derive(Debug, PartialEq, Eq, Hash, Clone, Copy)]
pub enum Pos {
    Sink,
    Counter,
    Stove,
}

#[derive(Debug, PartialEq, Eq, Hash, Clone)]
pub enum Holdable {
    Pot,
}

#[derive(Debug, PartialEq, Eq, Hash, Clone)]
pub enum Activatable {
    Stove,
    Faucet,
}

#[derive(Debug, PartialEq, Eq, Hash, Clone)]
pub struct State {
    pub pos: Pos,
    pub pot_pos: Pos,
    pub pot_filled: bool,
    pub faucet_on: bool,
    pub stove_on: bool,
    pub holding: Option<Holdable>,
}
```

Remember that back in Python, the state variables were often set to strings (yes, Python has `Enum`):

```python
s0 = State(pos='counter', pot_pos='counter', pot_filled=False,
           faucet_on=False, stove_on=False, holding=None)
```

The Rust approach will cause compilation to fail if position is set to an undefined value. During the migration from Rust to Python, I found that it definitely took more effort to get the program running, but once it ran, there were rarely logic errors. Some parts of the code did have to be more explicit. In Python, I was able to simply count how many elements in the state `namedtuple` where different from the goal, in Rust, the heuristic looks like the following:

```rust
fn heuristic(state: &State) -> f32 {
    let mut distance = 0.0;

    if state.pot_pos != Pos::Stove {
        distance += 1.0;
    }

    if !state.stove_on {
        distance += 1.0;
    }

    if !state.pot_filled {
        distance += 1.0;
    }

    if state.holding != None {
        distance += 1.0;
    }

    if state.faucet_on {
        distance += 1.0;
    }

    distance
}
```

Notice that the heuristic function doesn't receive a goal node as a parameter to compare against: it estimates how far the state is from the hard-coded expectations. This also avoids needing to create a `Goal` type. I initially set out to use a new `State` struct with nulled fields like in the Python implementation. There was a  problem, though: `State` must have definitions for every value in it (that isn't explicitly able to be `None`). However, the goal state often has "don't care" conditions for elements. What if I don't care about the final state of `faucet_on`? In the world represented by `State`, the faucet *must* be on or off. Since the goal doesn't change during the graph search, we'll just hard-code it into the heuristic.

# Changing the World

In Python, actions the AI agent could take were represented with functions that returned `None` if they failed preconditions, and otherwise returned a tuple with the new world state, the cost of the action, and a text description. This function moved the agent:

```python
def move(state, to):
    if state.pos == to:
        return None

    state = state._replace(pos=to)

    if state.holding is not None:
        moving_object = {state.holding + '_pos': to}
        state = state._replace(**moving_object)

    return state, 1, 'move to ' + to
```

The same action in Rust is very similar:

```rust
fn move_actor(state: &State, to: Pos) -> Option<Neighbor<State>> {
    if state.pos == to {
        return None;
    }

    let mut new_state = state.clone();
    new_state.pos = to;

    if new_state.holding == Some(Holdable::Pot) {
        new_state.pot_pos = to;
    }

    let action = match to {
        Pos::Sink => "Move to sink",
        Pos::Counter => "Move to counter",
        Pos::Stove => "Move to stove",
    };

    Some(Neighbor::new(new_state, 1.0, action.to_owned()))
}
```

As expected, things are more explicit. The `to` parameter must be one of the `Pos` enum values instead of a string. The function is guaranteed to always return `None` or `Some(Neighbor<State>)`. A `Neighbor` holds onto the information we used to return in the tuple:

```rust
pub struct Neighbor<S> {
    state: S,
    cost: f32,
    action: String,
}
```

`Neighbor` takes as type parameter `S`. This makes the `Neighbor` flexible enough to handle different world representations and not just our specific `State`. In Rust, the fields of `State` are fixed. If I want to separate my A\* functions from my specific scenario, I don't want to hardcode this one type of state into the A\* module, but I want the concept of Neighbors to be in the A* module. Using the type parameter lets `Neighbor` be generic to the exact type of state. Similarly, our specific `move_actor` action is part of the scenario and is only appropriate for this specific type of state.

The other actions from the Python version are defined similarly and are all in [main.rs](https://github.com/superlou/action_planning/blob/main/step5-rs/src/main.rs).

## A* is Reborn

Other than hiding the goal inside the `heuristic` function, the interface is the same as the Python version.

```
pub fn a_star<S>(
    start: &S,
    heuristic: &dyn Fn(&S) -> f32,
    neighbors: &dyn Fn(&S) -> Vec<Neighbor<S>>,
) -> Result<Vec<(S, String)>, ()>
where
    S: Clone + PartialEq + Eq + std::hash::Hash + std::fmt::Debug,
{
 ...
}
```

Probably the single greatest thing making the port to Rust easy is the `HashMap`, which can hash any struct that implements the `std::hash::Hash` trait, just like our `namedtuple`. The trait bounds on the type parameter `S` allow `a_star` to path plan using any neighbors where the state is hashable. Thanks to the `HashMap` providing the ability to look-up g-scores and f-scores from a state that's been visited, the body is an almost line for line translation from Python:

```rust
...
{
    let mut open_set: Vec<S> = vec![start.clone()];

    let mut came_from = HashMap::new();

    // Cost of path from start to node
    let mut g_score = HashMap::new();
    g_score.insert(start.clone(), 0.0);

    // Estimated cost of path from start through node to goal
    // This is an estimate of the total path cost.
    let mut f_score = HashMap::new();
    f_score.insert(start.clone(), heuristic(&start));

    while open_set.len() > 0 {
        open_set.sort_by(|a, b| {
            f_score
                .get(a)
                .unwrap_or(&f32::INFINITY)
                .partial_cmp(f_score.get(b).unwrap_or(&f32::INFINITY))
                .unwrap()
        });

        let current = &open_set.remove(0);

        if heuristic(current) == 0.0 {
            return Ok(reconstruct_path(came_from, current));
        }

        for neighbor in neighbors(current) {
            let tentative_g_score = g_score.get(current).unwrap_or(&f32::INFINITY) + neighbor.cost;
            if tentative_g_score < *g_score.get(&neighbor.state).unwrap_or(&f32::INFINITY) {
                // This path to the neighbor is the best one seen so far
                came_from.insert(
                    neighbor.state.clone(),
                    (current.clone(), neighbor.action.to_owned()),
                );
                g_score.insert(neighbor.state.clone(), tentative_g_score);
                f_score.insert(
                    neighbor.state.clone(),
                    tentative_g_score + heuristic(&neighbor.state),
                );

                if !open_set.contains(&&neighbor.state) {
                    open_set.push(neighbor.state.clone())
                }
            }
        }
    }

    return Err(());
}
```

In my [original Python implementation](https://github.com/superlou/action_planning/blob/main/step4/a_star.py#L22-L52), the function has no explicit return if it fails to construct a path. I should've written tests to expose this scenario, but in Rust, the compiler enforces a return of `Result<Vec<(S, String)>, ()>`. When successful, the function returns `Ok<Vec<(S, String)>`, which is a vector containing tuples of the states and actions to complete the path. If a path is not found, the result is `Err(())`, which is simply an error with an empty tuple.

# Running It

As with the Python version, the planner is run by declaring an initial state and then running the A* search of actions until the heuristic indicates we are at the goal state.

```rust
fn main() {
    let s0 = State {
        pos: Pos::Counter,
        pot_pos: Pos::Counter,
        pot_filled: false,
        faucet_on: false,
        stove_on: false,
        holding: None,
    };

    let result = a_star(&s0, &heuristic, &neighbors);

    match result {
        Ok(path) => {
            for (_, action) in path {
                println!("{}", action);
            }
        }
        Err(_) => println!("Unable to find solution!"),
    };
}
```

The result of compiling and running is the same as in Python.

```
Pick up pot
Move to sink
Turn on faucet
wait
Turn off faucet
Move to stove
Put down pot
Turn on stove
goal
```

The full Rust code is available [here](https://github.com/superlou/action_planning/tree/main/step5-rs). The biggest surprise I had was how straightforward it was to port the Python code to Rust. The next step in this series will be rewriting the planner to use dynamically generated scenarios and goals.
