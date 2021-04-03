+++
title = "Smoothing Rates of Change"
date = 2013-12-23
[taxonomies]
tags = ["dsp", "math"]

[extra]
katex = true
+++

This is probably something that would have been obvious to everyone else, but stumped me for a bit when I came across it the other day.  I was looking at the rate of change of a sequence, and my smoothing algorithm wasn’t doing any better than just subtracting two points in the sequence farther apart.  In fact, the results were identical.  This won’t be particularly rigorous, but I believe the concept is valid.

<!-- more -->

# Difference Between Points in a Sequence

We are looking at a set of samples $d[n]$, sampled at some set frequency.  Let’s create a new series which is the difference (delta) between every two consecutive points:

$$\Delta d[n] = d[n] - d[n-1]$$

$\Delta d[n]$ is the rate of change from one point to another, or can be thought of as the slope between the points.  Let’s generalize this for the case where we take the difference not between consecutive points, but between points spaced $s$ apart:

$$\Delta_s d[n] = \frac{1}{s} \left( d[n] - d[n-s] \right)$$

The $\frac{1}{s}$ is because, if you think in terms of slope, the spacing between your points has widened.  I’ll refer to this operation as taking the “spaced delta” of the sequence.  There’s probably a proper word for this, and I’m hoping someone will comment with it.

# The Moving Average

In my application, the sequence $d[n]$ was the sampled positions of a device, and since I was more concerned with how fast it was moving, I was looking at $\Delta_1 d[n]$, the rate of change.  Since this will be noisy when using real-world data, it needs to be smoothed.  An easy goto in the digital signal processing toolbox is the moving average.  You just average the past $w$ samples.  It’s relatively cheap so long as $w$ isn’t too large, and it is actually ideal for removing random noise while preserving step response.  Let’s define a moving average of our original sequence $d[n]$ as:

$$M_w d[n] = \frac{1}{w} \left( d[n] + d[n-1] + d[n-2] + \ldots + d[n-w] \right)$$

# Smoothing the Rate of Change

I want to smooth out our rate-of-change sequence, which looks like:

$$\Delta_1 d[n] = \left[ \ldots, d[n+2]-d[n+1], d[n+1]-d[n], d[n]-d[n-1], d[n-1]-d[n-2], d[n-2]-d[n-3], \ldots  \right]$$

If we plug these points into $M_w d[n]$ to average out the most recent $w$ points, we have:

$$M\_w \left( \Delta_1 d[n] \right) = \frac{1}{w} \left( (d[n]-d[n-1]) + (d[n-1]-d[n-2]) + (d[n-2]-d[n-3]) + \ldots + (d[n-w+1] – d[n-w]) \right)$$

The neat thing is that all of the inner terms cancels out, leaving:

$$M\_w \left( \Delta_1 d[n] \right) = \frac{1}{w} \left( d[n] - d[n-w] \right)$$

This is identical to the spaced delta with a spacing equal to the length of the moving average:

$$M\_w \left( \Delta_1 d[n] \right) = \Delta\_w d[n]$$

# Why This is Important

Trying to keep my terminology consistent, the special case of a spaced delta where the spacing is one is the rate of change of the system.  If you want to run a moving average on the rate of change of a sequence to smooth it, you can instead simply take the spaced delta of the original sequence with the spacing equal to what your moving average width would be.  This saves a large number of operations, and has less opportunity for errors to accumulate when you are working on systems with limited precision.

To note, I assumed that all of the operations needed to be causal, but I believe the result holds true without this so long as you are consistent.
