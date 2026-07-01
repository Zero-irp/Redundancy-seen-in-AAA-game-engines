---
layout: post
title: "Part 3: Avalanche Engine (Matrix Multiplication Replaces Vector Addition)"
date: 2026-05-26 00:00:00 +0530
permalink: /part-3-Avalanche-engine/
---

<style>
.post-nav {
  display: flex;
  justify-content: space-between;
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid #444;
  font-family: Consolas, "Liberation Mono", Menlo, monospace;
  font-size: 15px;
}
.post-nav a {
  color: #569cd6;
  text-decoration: none;
  padding: 10px 16px;
  background: #1e1e1e;
  border-radius: 6px;
  transition: background 0.2s ease;
}
.post-nav a:hover {
  background: #2d2d2d;
}
</style>

<style>
.ida-code{
  background:#1e1e1e;
  color:#dcdcdc;
  padding:12px;
  border-radius:8px;
  font-family: Consolas, "Liberation Mono", Menlo, monospace;
  font-size:14px;
  line-height:1.4;
  overflow-x:auto;
  white-space: pre; /* preserve spaces and linebreaks */
}

/* token classes you can use inside the div */
.ida-code .kw    { color:#569cd6; } /* keywords */
.ida-code .type  { color:#4ec9b0; } /* types */
.ida-code .fn    { color:#dcdcaa; } /* functions / intrinsics */
.ida-code .num   { color:#b5cea8; } /* numbers / hex */
.ida-code .var   { color:#9cdcfe; } /* variables */
.ida-code .const { color:#ce9178; } /* globals / constants */
.ida-code .comment{ color:#6a9955; font-style:italic; }
</style>

<style>
.asm64-wrap { display: flex; flex-direction: column; gap: 18px; padding: 4px 0; }
.asm64-section-label {
  font-size: 13px;
  color: var(--color-text-secondary);
  margin-bottom: 4px;
  font-family: var(--font-sans);
}
.asm64-code {
  background: #1e1e1e;
  color: #c5c8c6;
  padding: 12px;
  border-radius: 8px;
  font-family: "Cascadia Code", Consolas, "Liberation Mono", Menlo, monospace;
  font-size: 13px;
  line-height: 1.45;
  overflow-x: auto;
  white-space: pre;
  margin: 0;
}
.asm64-code .kw     { color: #d78700; font-weight: 600; }
.asm64-code .reg    { color: #5fafaf; }
.asm64-code .mem    { color: #af87d7; }
.asm64-code .num    { color: #b5cea8; }
.asm64-code .label  { color: #ffaf5f; }
.asm64-code .comment{ color: #6a9955; font-style: italic; }
.asm64-code .const  { color: #ce9178; }
</style>


### Temporal Anti-Aliasing (TAA): Matrix Multiplication Replaces Vector Addition

![ESP-Image1](/Redundancy-seen-in-AAA-game-engines/assets/images/part-3/jitter.png)

Here we see a very "Textbook" way of adding jitters to the projection matrix for `SMAA_T2X` in the Avalanche Engine.

The classic textbook way being:

$$
\begin{bmatrix}
x_{scale} & 0 & 0 & 0 \\
0 & y_{scale} & 0 & 0 \\
0 & 0 & \dfrac{z_{far}}{z_{far}-z_{near}} & 1 \\
0 & 0 & -\dfrac{z_{near}z_{far}}{z_{far}-z_{near}} & 0
\end{bmatrix}
\times
\begin{bmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
j_x & j_y & 0 & 1
\end{bmatrix}
=
\begin{bmatrix}
x_{scale} & 0 & 0 & 0 \\
0 & y_{scale} & 0 & 0 \\
j_x & j_y & \dfrac{z_{far}}{z_{far}-z_{near}} & 1 \\
0 & 0 & -\dfrac{z_{near}z_{far}}{z_{far}-z_{near}} & 0
\end{bmatrix}
$$

This looks Clean when looking at the source code, but in a low-level CPU render loop, it is inefficient.

Probably would look something like this in C++:

<div class="ida-code"><span class="type">Matrix4x4</span> <span class="var">jitterMatrix</span> = <span class="var">Matrix4x4</span>::<span class="fn">Identity</span>();
<span class="var">jitterMatrix</span>.<span class="var">m</span>[<span class="num">3</span>][<span class="num">0</span>] = <span class="var">jX</span>;
<span class="var">jitterMatrix</span>.<span class="var">m</span>[<span class="num">3</span>][<span class="num">1</span>] = <span class="var">jY</span>;


<span class="var">projMatrix</span> = <span class="var">projMatrix</span> * <span class="var">jitterMatrix</span>;</div>

Again: **Clean C++ Code ≠ Clean Compiled Code**

- First we need to construct an entire 4x4 Identity Matrix on the stack just to hold two float values.
- Then load the matrices as arguments into the function.
- Inside the function do stack allocation, set up security cookies, load registers etc.
- Multiply all rows to columns (repeated 4 times)
- Finally end the function by deallocating stack, verify the security cookie, loading result into memory and registers.

> Note: Calculating `curFrame & 1`, selecting jitters, scaling them down to sub-pixel space are all mathematically necessary steps and are not over-engineered.

The easier way to do it would be:

- Take the scaled down jitters.
- Take the 2nd Row (counting from 0) of the Projection Matrix and do a very simple `addps`.

Example (where the jitters were already scaled down):

<div class="asm64-wrap">
  <pre class="asm64-code"><span class="kw">movaps</span> <span class="reg">xmm0</span>, <span class="const">projMat_2</span>  <span class="comment">; Load 2nd Row [0, 0, Z_scale, 1]</span>
<span class="kw">movaps</span> <span class="reg">xmm1</span>, <span class="const">jitter_Row</span> <span class="comment">; [jitX, jitY, 0, 0]</span>

<span class="kw">addps</span> <span class="reg">xmm0</span>, <span class="reg">xmm1</span> <span class="comment">; result: [jX, jY, Z_scale, 1]</span></pre>
</div>

That's about a 120 instruction count drop to 3.

<div class="post-nav">
  <a href="{{ site.baseurl }}/part-2-Ghost-of-tsushima/">&laquo; Part 2: Ghost Of Tsushima</a>
  <a href="{{ site.baseurl }}/part-4-using-general-matrix-inverse-when-you-should-not/">Part 4: Using generic matrix inverses &raquo;</a>
</div>
 