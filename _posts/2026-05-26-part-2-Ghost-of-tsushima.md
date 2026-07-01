---
layout: post
title: "Part 2: Ghost Of Tsushima (Vector Extraction Through Multiplication)"
date: 2026-05-26 00:00:00 +0530
permalink: /part-2-Ghost-of-tsushima/
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

### Case 1: Vector Extraction Through Multiplication?

In Ghost of Tsushima while i was looking at how the View-Projection Matrix was being constructed i came across a common pattern where they do a full Row to column multiplication that could be replaced by a simple `movaps` instruction.

Take this Example (IDA pseudo code):

 <div class="ida-code"> <span class="var">v16</span>[<span class="num">3</span>] = <span class="fn">_mm_add_ps</span>(
             <span class="fn">_mm_add_ps</span>(
               <span class="fn">_mm_add_ps</span>(
		 <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>((<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, (<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, <span class="num">0</span>), <span class="var">ProjMat_0</span>), 
		 <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>((<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, (<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, <span class="num">0x55</span>), <span class="var">ProjMat_1</span>)),
               <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>((<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, (<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, <span class="num">0xAA</span>), <span class="var">ProjMat_2</span>)),
             <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>((<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, (<span class="type">__m128</span>)<span class="var">xmmword_1138D10</span>, <span class="num">0xFF</span>), <span class="var">ProjMat_3</span>));
</div>

> This is part of the construction of the View-Projection Matrix where the translation row of the matrix needed to be zeroed out (likely for skybox rendering).
> Multiply View with Projection only using directional vectors while zeroing out Translation.

The logic here is simply:

**Step 1: Shuffle**

- _mm_shuffle_ps(someRow1, someRow1, imm) selects one component of someRow1 and replicates it across all 4 slots of a new __m128.
- The different imm values pick different elements:
	0x00-> picks element 0 (X)
	0x55-> picks element 1 (Y)
	0xAA-> picks element 2 (Z)
	0xFF-> picks element 3 (W)

After shuffling, each __m128 looks like [X,X,X,X], [Y,Y,Y,Y], etc.

**Step 2: Multiply with Projection Matrix**

- Each shuffled vector is multiplied component-wise with a column of the projection matrix:

<div class="ida-code"><span class="fn">_mm_mul_ps</span>(<span class="var">shuffledRow</span>, <span class="var">ProjMat_n</span>)
</div>

- This performs 4 parallel multiplications of the same row component with each element in the projection matrix column.

**Step 3: Sum the results**

- The _mm_add_ps calls sum all four products together:

<div class="ida-code">(<span class="var">X</span> * <span class="var">ProjMat_0</span>) + (<span class="var">Y</span> * <span class="var">ProjMat_1</span>) + (<span class="var">Z</span> * <span class="var">ProjMat_2</span>) + (<span class="var">W</span> * <span class="var">ProjMat_3</span>)</div>

- The result is a single row of the final View-Projection matrix.

##### The Problem:

The problem here is that `xmmword_1138D10` has a value of: (0.0, 0.0, 0.0, 1.0).

Since the first three components are zero, those multiplications with ProjMat_0, ProjMat_1, and ProjMat_2 drop out. The only one left is the last one, where w = 1.0. Which means you’re just selecting the last row of the projection matrix (ProjMat_3).

So this whole instruction chain simplifies to basically 1 `movaps` instruction:

<div class="ida-code"> <span class="var">v16</span>[<span class="num">3</span>] = <span class="var">ProjMat_3</span></div>

### Case 2: Even More Vector Extraction Through Multiplication??

![ESP-Image1](/Redundancy-seen-in-AAA-game-engines/assets/images/part-2/ida_view_matrix_math.png)

This is the later stage where translation is added back into the View-Projection Matrix where it was previously zeroed out, and it is done in a very confusing way.

<div class="ida-code"><span class="var">v17</span> = <span class="fn">_mm_add_ps</span>(
        <span class="fn">_mm_add_ps</span>(
          <span class="fn">_mm_add_ps</span>(
	    <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">CamPos_Negated_w1_dup</span>, <span class="var">CamPos_Negated_w1_dup</span>, <span class="num">0x55</span>), <span class="var">VP_NoTrans_Row1</span>), 
	    <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">CamPos_Negated_w1_dup</span>, <span class="var">CamPos_Negated_w1_dup</span>, <span class="num">0</span>), *<span class="var">VP_NoTrans</span>)),
          <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">CamPos_Negated_w1_dup</span>, <span class="var">CamPos_Negated_w1_dup</span>, <span class="num">0xAA</span>), <span class="var">VP_NoTrans_Row2</span>)),
        <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">CamPos_Negated_w1_dup</span>, <span class="var">CamPos_Negated_w1_dup</span>, <span class="num">0xFF</span>), <span class="var">VP_NoTrans_Row3</span>));
</div>

This is the only Vector Multiplication that matters, the one where it's adding back the translation into the VP matrix.

Here is the biggest reduction:

<div class="ida-code"> <span class="var">v18</span> = <span class="fn">_mm_add_ps</span>(
	 <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0100</span>, <span class="var">Mask_0100</span>, <span class="num">0x55</span>), <span class="var">VP_NoTrans_Row1</span>), 
	 <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0100</span>, <span class="var">Mask_0100</span>, <span class="num">0</span>), *<span class="var">VP_NoTrans</span>));
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x260</span>) = <span class="fn">_mm_add_ps</span>(
                              <span class="fn">_mm_add_ps</span>(
                                <span class="fn">_mm_add_ps</span>(
                                  <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0010</span>, <span class="var">Mask_0010</span>, <span class="num">0x55</span>), <span class="var">VP_NoTrans_Row1</span>),
                                  <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0010</span>, <span class="var">Mask_0010</span>, <span class="num">0</span>), *<span class="var">VP_NoTrans</span>)),
                                <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0010</span>, <span class="var">Mask_0010</span>, <span class="num">0xAA</span>), <span class="var">VP_NoTrans_Row2</span>)),
                              <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0010</span>, <span class="var">Mask_0010</span>, <span class="num">0xFF</span>), <span class="var">VP_NoTrans_Row3</span>));
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x270</span>) = <span class="var">v17</span>;
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x250</span>) = <span class="fn">_mm_add_ps</span>(
                              <span class="fn">_mm_add_ps</span>(<span class="var">v18</span>, <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0100</span>, <span class="var">Mask_0100</span>, <span class="num">0xAA</span>), <span class="var">VP_NoTrans_Row2</span>)),
                              <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_0100</span>, <span class="var">Mask_0100</span>, <span class="num">0xFF</span>), <span class="var">VP_NoTrans_Row3</span>));
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x240</span>) = <span class="fn">_mm_add_ps</span>(
                              <span class="fn">_mm_add_ps</span>(
                                <span class="fn">_mm_add_ps</span>(
                                  <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_1000</span>, <span class="var">Mask_1000</span>, <span class="num">0x55</span>), <span class="var">VP_NoTrans_Row1</span>),
                                  <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_1000</span>, <span class="var">Mask_1000</span>, <span class="num">0</span>), <span class="var">VP_NoTrans_Row0</span>)),
                                <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_1000</span>, <span class="var">Mask_1000</span>, <span class="num">0xAA</span>), <span class="var">VP_NoTrans_Row2</span>)),
                              <span class="fn">_mm_mul_ps</span>(<span class="fn">_mm_shuffle_ps</span>(<span class="var">Mask_1000</span>, <span class="var">Mask_1000</span>, <span class="num">0xFF</span>), <span class="var">VP_NoTrans_Row3</span>));

</div>

This is just extracting the values stored in the VP rows using Masks.

Mask_1000 is (1, 0, 0, 0)
Mask_0100 is (0, 1, 0, 0)
Mask_0010 is (0, 0, 1, 0)

Multiplying a unit vector by a matrix simply extracts the corresponding row. The original code was laboriously performing this extraction manually for each axis:

- The calculation for 0x240 used Mask_1000 to extract Row0.
- The calculation for 0x250 used Mask_0100 to extract Row1.
- The calculation for 0x260 used Mask_0010 to extract Row2.

So it just collapses into 3 `movaps` instructions and 1 vector multiplication.

<div class="ida-code">  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x240</span>) = <span class="var">VP_NoTrans_Row0</span>
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x250</span>) = <span class="var">VP_NoTrans_Row1</span>
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x260</span>) = <span class="var">VP_NoTrans_Row2</span>
  *(<span class="type">__m128</span> *)(<span class="var">a1</span> + <span class="num">0x270</span>) = <span class="var">v17</span>
</div>

Again: I was not looking specifically for over-engineered code, this just stood out a lot.

>I have also optimized this on my previous blog in assembly (for fun):  
[Reversing The ViewProjection Matrix - Part 4.5: Detour Hooking to Optimize SIMD Operations](https://zero-irp.github.io/ViewProj-Blog/part-4.5-detour-hooking-simd-operations/)

<div class="post-nav">
  <a href="{{ site.baseurl }}/part-1-dunia/">&laquo; Part 1: Dunia Engine</a>
  <a href="{{ site.baseurl }}/part-3-Avalanche-engine/">Part 3: Avalanche Engine &raquo;</a>
</div>


