---
layout: post
title: "Part 4: Using generic matrix inverse when you don't need to"
date: 2026-05-26 00:00:00 +0530
permalink: /part-4-using-general-matrix-inverse-when-you-should-not/
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
.ida-code .kw    { color:#ff9e3b; } /* keywords (Bright Orange) -> Or use #f9d849 for Bright Yellow *
.ida-code .type  { color:#4ec9b0; } /* types */
.ida-code .fn    { color:#dcdcaa; } /* functions / intrinsics */
.ida-code .num   { color:#b5cea8; } /* numbers / hex */
.ida-code .var   { color:#9cdcfe; } /* variables */
.ida-code .const { color:#ce9178; } /* globals / constants */
.ida-code .comment{ color:#6a9955; font-style:italic; }
</style>


This is a small tangent I'm going to go on: It's about the amount of times I have seen a game engine using a generic matrix inverse function where it can be inlined to be way WAY faster!

Here is a generic matrix inverse using Cramer's Rule:

<div class="ida-code"><span class="type">_UNKNOWN</span> **<span class="kw">__fastcall</span> <span class="fn">sub_65C3E20</span>(<span class="type">__m128</span> *<span class="var">a1</span>)
{
  <span class="var">v2</span> = *<span class="var">a1</span>;
  <span class="var">v3</span> = <span class="var">a1</span>[<span class="num">1</span>];
  <span class="var">v4</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v2</span>, <span class="var">v2</span>, <span class="num">0xFF</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v5</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v3</span>, <span class="var">v3</span>, <span class="num">0xFF</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v40</span> = *<span class="var">a1</span>;
  <span class="var">v42</span> = <span class="var">v3</span>.<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v6</span> = <span class="var">a1</span>[<span class="num">2</span>];
  <span class="var">v7</span> = <span class="var">a1</span>[<span class="num">3</span>];
  <span class="var">v8</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v7</span>, <span class="var">v7</span>, <span class="num">0xFF</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v9</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v7</span>, <span class="var">v7</span>, <span class="num">0xAA</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v10</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v6</span>, <span class="var">v6</span>, <span class="num">0xFF</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v11</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v2</span>, <span class="var">v2</span>, <span class="num">0xAA</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v38</span> = <span class="var">v7</span>;
  <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v3</span>, <span class="var">v3</span>, <span class="num">0xAA</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v39</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v6</span>, <span class="var">v6</span>, <span class="num">0xAA</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v12</span> = <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v8</span>;
  <span class="var">v13</span> = <span class="var">v39</span> * <span class="var">v8</span>;
  <span class="var">v14</span> = <span class="var">v5</span> * <span class="var">v9</span>;
  <span class="var">v37</span> = <span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v41</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v3</span>, <span class="var">v3</span>, <span class="num">0x55</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v46</span> = <span class="var">v8</span>;
  <span class="var">v15</span> = <span class="var">v11</span> * <span class="var">v8</span>;
  <span class="var">v44</span> = <span class="var">v11</span>;
  <span class="var">v16</span> = <span class="var">v4</span> * <span class="var">v9</span>;
  <span class="var">v48</span> = <span class="var">v9</span>;
  <span class="var">v17</span> = <span class="var">v10</span> * <span class="var">v9</span>;
  <span class="var">v18</span> = <span class="var">v11</span> * <span class="var">v5</span>;
  <span class="var">v49</span> = <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] = <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v10</span>;
  <span class="var">v43</span> = <span class="var">v10</span>;
  <span class="var">v19</span> = <span class="var">v11</span> * <span class="var">v10</span>;
  <span class="var">v20</span> = <span class="var">v4</span> * <span class="var">v39</span>;
  <span class="var">v47</span> = <span class="var">v4</span>;
  <span class="var">v21</span> = <span class="var">v4</span> * <span class="var">v49</span>;
  <span class="var">v31</span> = <span class="var">v12</span>;
  <span class="var">v45</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v6</span>, <span class="var">v6</span>, <span class="num">0x55</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v50</span> = <span class="fn">_mm_shuffle_ps</span>(<span class="var">v38</span>, <span class="var">v38</span>, <span class="num">0x55</span>).<span class="var">m128_f32</span>[<span class="num">0</span>];
  <span class="var">v30</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v45</span> * <span class="var">v14</span>) + (<span class="kw">float</span>)(<span class="var">v41</span> * <span class="var">v13</span>)) + (<span class="kw">float</span>)(<span class="var">v50</span> * <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>]))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v45</span> * <span class="var">v12</span>) + (<span class="kw">float</span>)(<span class="var">v41</span> * <span class="var">v17</span>)) + (<span class="kw">float</span>)(<span class="var">v50</span> * (<span class="kw">float</span>)(<span class="var">v5</span> * <span class="var">v39</span>)));
  <span class="var">v32</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v45</span> * <span class="var">v15</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">1</span>] * <span class="var">v17</span>)) + (<span class="kw">float</span>)(<span class="var">v50</span> * <span class="var">v20</span>))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v45</span> * <span class="var">v16</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">1</span>] * <span class="var">v13</span>)) + (<span class="kw">float</span>)(<span class="var">v50</span> * <span class="var">v19</span>));
  <span class="var">v33</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v41</span> * <span class="var">v16</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">1</span>] * <span class="var">v12</span>)) + (<span class="kw">float</span>)(<span class="var">v50</span> * <span class="var">v18</span>))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v41</span> * <span class="var">v15</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">1</span>] * <span class="var">v14</span>)) + (<span class="kw">float</span>)(<span class="var">v50</span> * <span class="var">v21</span>));
  <span class="var">v3</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v41</span> * <span class="var">v19</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">1</span>] * (<span class="kw">float</span>)(<span class="var">v5</span> * <span class="var">v39</span>)))
                         + (<span class="kw">float</span>)(<span class="var">v45</span> * <span class="var">v21</span>))
                 - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v41</span> * <span class="var">v20</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">1</span>] * <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>])) + (<span class="kw">float</span>)(<span class="var">v45</span> * <span class="var">v18</span>));
  <span class="var">v22</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v12</span>) + (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v17</span>))
              + (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * (<span class="kw">float</span>)(<span class="var">v5</span> * <span class="var">v39</span>)))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v14</span>) + (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v13</span>)) + (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>]));
  <span class="var">v23</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v16</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v13</span>)) + (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v19</span>))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v15</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v17</span>)) + (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v20</span>));
  <span class="var">v24</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v15</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v14</span>)) + (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v21</span>))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v16</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v31</span>)) + (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v18</span>));
  <span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] = <span class="var">v42</span> * <span class="var">v19</span>;
  <span class="var">v25</span> = <span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="fn">COERCE_FLOAT</span>(<span class="fn">HIDWORD</span>(<span class="var">a1</span>-><span class="var">m128_u64</span>[<span class="num">0</span>]));
  <span class="var">v26</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v20</span>) + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v7</span>.<span class="var">m128_f32</span>[<span class="num">0</span>])) + (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v18</span>))
      - (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v6</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] + (<span class="kw">float</span>)(<span class="var">v40</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * (<span class="kw">float</span>)(<span class="var">v5</span> * <span class="var">v39</span>))) + (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v21</span>));
  <span class="var">v27</span> = <span class="fn">COERCE_FLOAT</span>(*<span class="var">a1</span>) * <span class="var">v50</span>;
  <span class="var">v29</span> = <span class="fn">COERCE_FLOAT</span>(*<span class="var">a1</span>) * <span class="var">v45</span>;
  <span class="var">v34</span> = <span class="var">v37</span> * <span class="fn">COERCE_FLOAT</span>(<span class="fn">HIDWORD</span>(<span class="var">a1</span>-><span class="var">m128_u64</span>[<span class="num">0</span>]));
  <span class="var">v35</span> = <span class="fn">COERCE_FLOAT</span>(*<span class="var">a1</span>) * <span class="var">v41</span>;
  <span class="var">v36</span> = <span class="var">v42</span> * <span class="fn">COERCE_FLOAT</span>(<span class="fn">HIDWORD</span>(<span class="var">a1</span>-><span class="var">m128_u64</span>[<span class="num">0</span>]));
  <span class="var">v28</span> = <span class="num">1.0</span>
      / (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v30</span> * <span class="fn">COERCE_FLOAT</span>(*<span class="var">a1</span>)) + (<span class="kw">float</span>)(<span class="var">v32</span> * <span class="var">v42</span>)) + (<span class="kw">float</span>)(<span class="var">v33</span> * <span class="var">v37</span>))
              + (<span class="kw">float</span>)(<span class="var">v3</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>]));
  <span class="var">a1</span>-><span class="var">m128_f32</span>[<span class="num">0</span>] = <span class="var">v30</span> * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">1</span>].<span class="var">m128_f32</span>[<span class="num">1</span>] = <span class="var">v28</span> * <span class="var">v23</span>;
  <span class="var">a1</span>[<span class="num">1</span>].<span class="var">m128_f32</span>[<span class="num">0</span>] = <span class="var">v28</span> * <span class="var">v22</span>;
  <span class="var">a1</span>[<span class="num">1</span>].<span class="var">m128_f32</span>[<span class="num">3</span>] = <span class="var">v28</span> * <span class="var">v26</span>;
  <span class="var">a1</span>[<span class="num">1</span>].<span class="var">m128_f32</span>[<span class="num">2</span>] = <span class="var">v28</span> * <span class="var">v24</span>;
  <span class="var">a1</span>-><span class="var">m128_f32</span>[<span class="num">3</span>] = <span class="var">v3</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v28</span>;
  <span class="var">a1</span>-><span class="var">m128_f32</span>[<span class="num">1</span>] = <span class="var">v32</span> * <span class="var">v28</span>;
  <span class="var">a1</span>-><span class="var">m128_f32</span>[<span class="num">2</span>] = <span class="var">v33</span> * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">2</span>].<span class="var">m128_f32</span>[<span class="num">0</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v41</span>) * <span class="var">v43</span>)
                                            + (<span class="kw">float</span>)(<span class="var">v5</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v50</span>)))
                                    + (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v45</span>) * <span class="var">v46</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v5</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v45</span>))
                                            + (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v50</span>) * <span class="var">v43</span>))
                                    + (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v41</span>) * <span class="var">v46</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">2</span>].<span class="var">m128_f32</span>[<span class="num">1</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v45</span>)) + (<span class="kw">float</span>)(<span class="var">v27</span> * <span class="var">v43</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v34</span> * <span class="var">v46</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v25</span> * <span class="var">v43</span>) + (<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v50</span>)))
                                    + (<span class="kw">float</span>)(<span class="var">v29</span> * <span class="var">v46</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">2</span>].<span class="var">m128_f32</span>[<span class="num">2</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v25</span> * <span class="var">v5</span>) + (<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v50</span>))) + (<span class="kw">float</span>)(<span class="var">v35</span> * <span class="var">v46</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v41</span>)) + (<span class="kw">float</span>)(<span class="var">v27</span> * <span class="var">v5</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v36</span> * <span class="var">v46</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">2</span>].<span class="var">m128_f32</span>[<span class="num">3</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v41</span>)) + (<span class="kw">float</span>)(<span class="var">v29</span> * <span class="var">v5</span>)) + (<span class="kw">float</span>)(<span class="var">v36</span> * <span class="var">v43</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v34</span> * <span class="var">v5</span>) + (<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v45</span>))) + (<span class="kw">float</span>)(<span class="var">v35</span> * <span class="var">v43</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">3</span>].<span class="var">m128_f32</span>[<span class="num">0</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v41</span>) * <span class="var">v48</span>) + (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v50</span>) * <span class="var">v39</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v49</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v45</span>)))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v45</span>) * <span class="var">v48</span>) + (<span class="kw">float</span>)(<span class="var">v49</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v50</span>)))
                                    + (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v41</span>) * <span class="var">v39</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">3</span>].<span class="var">m128_f32</span>[<span class="num">1</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v29</span> * <span class="var">v48</span>) + (<span class="kw">float</span>)(<span class="var">v44</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v50</span>)))
                                    + (<span class="kw">float</span>)(<span class="var">v25</span> * <span class="var">v39</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v34</span> * <span class="var">v48</span>) + (<span class="kw">float</span>)(<span class="var">v27</span> * <span class="var">v39</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v44</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v45</span>))))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">3</span>].<span class="var">m128_f32</span>[<span class="num">2</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v36</span> * <span class="var">v48</span>) + (<span class="kw">float</span>)(<span class="var">v27</span> * <span class="var">v49</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v44</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v41</span>)))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v35</span> * <span class="var">v48</span>) + (<span class="kw">float</span>)(<span class="var">v44</span> * (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v50</span>)))
                                    + (<span class="kw">float</span>)(<span class="var">v25</span> * <span class="var">v49</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">3</span>].<span class="var">m128_f32</span>[<span class="num">3</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v44</span> * (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v45</span>)) + (<span class="kw">float</span>)(<span class="var">v35</span> * <span class="var">v39</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v34</span> * <span class="var">v49</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v29</span> * <span class="var">v49</span>) + (<span class="kw">float</span>)(<span class="var">v36</span> * <span class="var">v39</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v44</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v41</span>))))
                    * <span class="var">v28</span>;
  <span class="kw">return</span> &<span class="var">retaddr</span>;
}</div>

For matrices like the Projection Matrix, a large number of values are simply 0. If it's a well-known matrix, it can be inlined without using generic inversions. It would be a huge drop in CPU cycles!

In the dunia engine they have inlined the inverse Projection Matrix calculation by saving important variables while constructing the Projection Matrix and rearranging it.

The instruction count for this generic Cramer's Rule inverse is roughly 470, not to mention instruction count alone isn't enough to showcase the inefficiency 

### 1. SIMD to Scalar

The matrix rows are loaded into 128-bit wide registers, but get immediately unpacked to do scalar calculations 32 bits at a time.

### 2. Register Spilling

<div class="ida-code">  <span class="kw">float</span> <span class="var">v29</span>; <span class="comment">// [rsp+4h] [rbp-1C4h]</span>
  <span class="kw">float</span> <span class="var">v30</span>; <span class="comment">// [rsp+8h] [rbp-1C0h]</span>
  <span class="kw">float</span> <span class="var">v31</span>; <span class="comment">// [rsp+Ch] [rbp-1BCh]</span>
  <span class="kw">float</span> <span class="var">v32</span>; <span class="comment">// [rsp+10h] [rbp-1B8h]</span>
  <span class="kw">float</span> <span class="var">v33</span>; <span class="comment">// [rsp+14h] [rbp-1B4h]</span>
  <span class="kw">float</span> <span class="var">v34</span>; <span class="comment">// [rsp+1Ch] [rbp-1ACh]</span>
  <span class="kw">float</span> <span class="var">v35</span>; <span class="comment">// [rsp+20h] [rbp-1A8h]</span>
  <span class="kw">float</span> <span class="var">v36</span>; <span class="comment">// [rsp+24h] [rbp-1A4h]</span>
<span class="comment">etc...</span></div>

16 XMM registers are available for floating-point math. This function defines over 50 individual float variables.

The variables written with annotations like `[rsp+Ch]` indicates that the CPU ran out of hardware registers and was forced to "spill" intermediate calculations to the stack.

### 3. Dependency Chains

look at:

<div class="ida-code">  <span class="var">v28</span> = <span class="num">1.0</span>
      / (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v30</span> * <span class="fn">COERCE_FLOAT</span>(*<span class="var">a1</span>)) + (<span class="kw">float</span>)(<span class="var">v32</span> * <span class="var">v42</span>)) + (<span class="kw">float</span>)(<span class="var">v33</span> * <span class="var">v37</span>))
              + (<span class="kw">float</span>)(<span class="var">v3</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>]));</div>

which represents `1.0/determinant`. Its calculation requires v30, v32, v33, and others to be completely finished.

Subsequently, every single element written back to the final matrix depends on v28.

<div class="ida-code">  <span class="var">a1</span>[<span class="num">2</span>].<span class="var">m128_f32</span>[<span class="num">1</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v45</span>)) + (<span class="kw">float</span>)(<span class="var">v27</span> * <span class="var">v43</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v34</span> * <span class="var">v46</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v25</span> * <span class="var">v43</span>) + (<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v37</span> * <span class="var">v50</span>)))
                                    + (<span class="kw">float</span>)(<span class="var">v29</span> * <span class="var">v46</span>)))
                    * <span class="var">v28</span>;
  <span class="var">a1</span>[<span class="num">2</span>].<span class="var">m128_f32</span>[<span class="num">2</span>] = (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v25</span> * <span class="var">v5</span>) + (<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v42</span> * <span class="var">v50</span>))) + (<span class="kw">float</span>)(<span class="var">v35</span> * <span class="var">v46</span>))
                            - (<span class="kw">float</span>)((<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">v47</span> * (<span class="kw">float</span>)(<span class="var">v38</span>.<span class="var">m128_f32</span>[<span class="num">0</span>] * <span class="var">v41</span>)) + (<span class="kw">float</span>)(<span class="var">v27</span> * <span class="var">v5</span>))
                                    + (<span class="kw">float</span>)(<span class="var">v36</span> * <span class="var">v46</span>)))
                    * <span class="var">v28</span>;
<span class="comment">etc...</span></div>

I would be very surprised if it isn't stalled at least somewhere in the pipeline

The dunia engine inlined this to about 15-20 instructions by saving required values into registers or the stack and constructing it at the end of projection matrix construction.
More info on my previous write-up! [Part 6: Reversing Construction of the Inverse Projection Matrix](https://zero-irp.github.io/Proj-Blog/part-6-reversing-inverse-projection-matrix/)

This also ties into the Camera Matrix and honestly various other matrices!

> Reminder!  
> Camera Matrix^-1 = View Matrix

Let's take the view matrix for example and how to inline it from my previous blog [Reversing The ViewProjection Matrix (Part 4.2: Reversing SIMD Instructions for Matrix Math - Fast inverse for orthonormal Matrices)](https://zero-irp.github.io/ViewProj-Blog/part-4.2-reversing-simd-instrustions/#fast-inverse-for-orthonormal-matrices)

### Fast inverse for orthonormal Matrices

If R is a pure rotation matrix meaning:

- No scaling,
- No shear,
- It’s orthonormal (columns are perpendicular and unit-length)

then $$R^{-1} = R^T$$

Suppose a 4x4 matrix with homogenous coordinates:  

$$
C_{world} =
\begin{bmatrix}
R_{00} & R_{01} & R_{02} & 0 \\
R_{10} & R_{11} & R_{12} & 0 \\
R_{20} & R_{21} & R_{22} & 0 \\
T_x & T_y & T_z & 1.0
\end{bmatrix}
$$

Here:  
- R (upper 3×3) is the orientation of the camera in world space.  
- T (bottom row, first 3 values) is the position of the camera in world space.

To get $$C_{world}^{-1}$$ we can separate the matrix like so:  

$$
C_{world} =
\begin{bmatrix}
R & 0 \\
T & 1
\end{bmatrix}
$$

and we want its inverse.

The block matrix inverse formula for this special form  is:

$$
\begin{bmatrix} 
A & 0 \\ 
B & 1 
\end{bmatrix}^{-1}
=
\begin{bmatrix} 
A^{-1} & 0 \\ 
-BA^{-1} & 1 
\end{bmatrix}
$$

*(See [Wikipedia: Blockwise inversion](https://en.wikipedia.org/wiki/Invertible_matrix#Blockwise_inversion) for the general derivation)*

Applying The Formula we get:  

- A = R
- B = T

So:  

$$
C_{world}^{-1} =
\begin{bmatrix}
R^{-1} & 0 \\
-TR^{-1} & 1
\end{bmatrix}
$$

Since R is orthonormal ($$R^{-1} = R^T$$):  

$$
C_{world}^{-1} =
\begin{bmatrix}
R^T & 0 \\
-TR^T & 1
\end{bmatrix}
$$

> Exponent "T" represents the Transpose and Regular "T" represents the Translation

Now Expand  $$−TR^T$$ into its dot products:  

if:  

$$
R =
\begin{bmatrix}
R_{0x} & R_{0y} & R_{0z} \\
R_{1x} & R_{1y} & R_{1z} \\
R_{2x} & R_{2y} & R_{2z} \\
\end{bmatrix}
$$

and $$T = [T_x, T_y, T_z],$$  

So:  

$$-TR^T=
\begin{bmatrix} 
-T_x & -T_y & -T_z \\
\end{bmatrix}
\times
\begin{bmatrix}
R_{0x} & R_{1x} & R_{2x} \\
R_{0y} & R_{1y} & R_{2y} \\
R_{0z} & R_{1z} & R_{2z} \\
\end{bmatrix}
$$

then:  

$$-TR^T = [-dot(T,R_0), -dot(T,R_1), -dot(T,R_2)]$$  

So the last row becomes:  

$$[-dot(T,R_0), -dot(T,R_1), -dot(T,R_2)]$$

And Expanding $$R^T$$ is just the Transpose of the Rotation, thus completing the inverse:  

$$
C_{world}^{-1} =
\begin{bmatrix}
R^T & 0 \\
-TR^T & 1
\end{bmatrix}
$$

This is also seen in the dunia engine and honestly in most game engines.

### Dunia Engine Example for View Matrix Fast Inverse:

<div class="ida-code">  <span class="comment">// Dot Product of: Right • CamPos</span>
  <span class="var">rightTrans</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">rightY</span> * <span class="var">CamPos_XYZ</span>[<span class="num">1</span>]) + (<span class="kw">float</span>)(<span class="var">rightX</span> * *<span class="var">CamPos_XYZ</span>))
             + (<span class="kw">float</span>)(<span class="var">rightZ</span> * <span class="var">CamPos_XYZ</span>[<span class="num">2</span>]);
  <span class="var">forwardZ</span> = *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x88</span>);
  <span class="var">upX</span> = *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x90</span>);
  <span class="var">upY</span> = *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x94</span>);

  <span class="comment">// Dot Product of: Forward • CamPos</span>
  <span class="var">forwardTrans</span> = (<span class="kw">float</span>)((<span class="kw">float</span>)(<span class="var">forwardY</span> * <span class="var">CamPos_XYZ</span>[<span class="num">1</span>]) + (<span class="kw">float</span>)(<span class="var">forwardX</span> * *<span class="var">CamPos_XYZ</span>))
               + (<span class="kw">float</span>)(<span class="var">forwardZ</span> * <span class="var">CamPos_XYZ</span>[<span class="num">2</span>]);
  <span class="var">upZ</span> = *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x98</span>);

  <span class="comment">// Dot product of: up • CamPos (upTrans + v18)</span>
  *(<span class="kw">float</span> *)&<span class="var">v18</span> = <span class="var">upZ</span> * <span class="var">CamPos_XYZ</span>[<span class="num">2</span>];
  <span class="var">upTrans</span> = (<span class="kw">float</span>)(<span class="var">upY</span> * <span class="var">CamPos_XYZ</span>[<span class="num">1</span>]) + (<span class="kw">float</span>)(<span class="var">upX</span> * *<span class="var">CamPos_XYZ</span>);</div>

Standard Camera Matrix layout being (Memory Layout):

$$
C_{world} =
\begin{bmatrix}
r_x & r_y & r_z & 0 \\
u_x & u_y & u_z & 0 \\
f_x & f_y & f_z & 0 \\
p_x & p_y & p_z & 1.0
\end{bmatrix}
$$

here the right vector would be stored like so:

`*(float *)(a1 + 0x30) = rightX;   *(float *)(a1 + 0x34) = rightY;   *(float *)(a1 + 0x38) = rightZ;`

The dunia engine transposes it like so and adds the dot products.

<div class="ida-code">  <span class="comment">// Fast inverse for orthonormal matrices (View Matrix Construction)</span>
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x30</span>) = <span class="var">rightX</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x40</span>) = <span class="var">rightY</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x50</span>) = <span class="var">rightZ</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x34</span>) = <span class="var">forwardX</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x44</span>) = <span class="var">forwardY</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x54</span>) = <span class="var">forwardZ</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x58</span>) = <span class="var">upZ</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x38</span>) = <span class="var">upX</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x48</span>) = <span class="var">upY</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x60</span>) = -<span class="var">rightTrans</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x64</span>) = -<span class="var">forwardTrans</span>;
  *(<span class="kw">float</span> *)(<span class="var">a1</span> + <span class="num">0x68</span>) = -(<span class="kw">float</span>)(<span class="var">upTrans</span> + *(<span class="kw">float</span> *)&<span class="var">v18</span>);</div>

### Closing thoughts:

I'm all out of rants and tangents to go on about, here are some main takeaways:

**1. Clean C++ Code ≠ Clean Compiled Code**

*Abstraction is a luxury where the cost is performance.* Generalized math wrappers and trusting the compiler to "figure it out" is how you end up with 133 instructions instead of 3.

**2. Profilers won't save you**

If the entire foundation is bloated, the baseline execution cost of every function is artificially raised. *Profiler-Invisible Waste / Death by a Thousand Cuts*

**3. Following the textbook perfectly**  

Those matrix inversions, matrix multiplications, identity matrices all look great on the whiteboard but in a low-level CPU pipeline it is going about it in a really roundabout way.

**4. The "Main Path" Contagion**  

This is not even a niche, unimportant function. This is the main rendering function preparing many different matrices bound for the GPU for calculations.  
You can guarantee this exact same philosophy infests every other system in the engine.

**5. Who was the common antagonist anyway?**

You might have already guessed! It's the `MatrixMultiply4x4()` but really that's just the narrative for this write-up.

The true antagonist is the codebase culture itself. It is the "Clean C++" philosophy that prioritizes developer convenience and generic abstractions over CPU execution realities. This exact same over-engineering bleeds into entirely different mathematical primitives across the entire engine.

And it probably doesn't even stop at just math libraries, probably every other library is also abused like this.

In my next write-up, we are going to look at the exact opposite problem. We are going to explore the Compatibility Tax. The ghost of a 12-year-old CPU that keeps modern games from utilizing instructions that could theoretically yield 5x speedups.

*Until then, worship the IDA goddess!*

<div class="post-nav">
  <a href="{{ site.baseurl }}/part-3-Avalanche-engine/">&laquo; Part 3: Avalanche Engine (Depth)</a>
</div>










