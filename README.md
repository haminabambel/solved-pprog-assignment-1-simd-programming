Download Link: https://assignmentchef.com/product/solved-pprog-assignment-1-simd-programming
<br>
5/5 - (1 vote)







The purpose of this assignment is to familiarize yourself with SIMD (single instruction, multiple data) programming. Most modern processors include some form of vector operations (i.e., SIMD instructions), and some applications may take advantage of SIMD instructions to improve performance through vectorization. Although modern compilers support automatic vectorization optimizations, the capabilities of compilers to fully auto-vectorize a given piece of code are often limited. Fortunately, almost all compilers (targeted to processors with SIMD extensions) provide SIMD intrinsics to allow programmers to vectorize their code explicitly.




<hr>

<h2 id="evaluationplatform">Evaluation Platform</h2>

Your program should be able to run on UNIX-like OS platforms. We will evaluate your programs on the workstations dedicated for this course. You can access these workstations by <code>ssh</code> with the following information. (To learn how to use <code>ssh</code> and <code>scp</code>, you can refer to the video listed in the <a href="#References">references</a>)

The workstations are based on Ubuntu 20.04 with Intel(R) Core(TM) i5-7500 CPU @ 3.40GHz processors. <code>g++-10</code>(used in part1) and <code>clang++-11</code>(used in part2) have been installed.

<table>

 <thead>

  <tr>

   <th></th>

   <th>Port</th>

   <th>User Name</th>

   <th>Password</th>

  </tr>

 </thead>

 <tbody>

  <tr>

   <td>140.113.215.195</td>

   <td>37072 ~ 37080</td>

   <td>{student_id}</td>

   <td>{student_id}</td>

  </tr>

 </tbody>

</table>

Login example:

<pre><code class="shell language-shell">$ ssh &lt;student_id&gt;@140.113.215.195 -p &lt;port&gt;</code></pre>

Please download the code and unzip it:

<pre><code class="shell language-shell">$ wget https://nycu-sslab.github.io/PP-f21/HW1/HW1.zip$ unzip HW1.zip -d HW1$ cd HW1</code></pre>

<h2 id="1part1vectorizingcodeusingfakesimdintrinsics">1. Part 1: Vectorizing Code Using Fake SIMD Intrinsics</h2>

Take a look at the function <code>clampedExpSerial</code> in <code>part1/main.cpp</code> of the Assignment I code base. The <code>clampedExp()</code> function raises <code>values[i]</code> to the power given by <code>exponents[i]</code> for all elements of the input array and clamps the resulting values at 9.999999. Your job is to vectorize this piece of code so it can be run on a machine with SIMD vector instructions.

Please enter the <code>part1</code> folder:

<pre><code class="shell language-shell">$ cd part1</code></pre>

<strong>Rather than</strong> craft an implementation using <em>SSE</em> or <em>AVX2</em> vector intrinsics that map to real SIMD vector instructions on modern CPUs, to make things a little easier, we’re asking you to <strong><em>implement your version using PP’s “fake vector intrinsics” defined in <code>PPintrin.h</code>.</em></strong> The <code>PPintrin.h</code> library provides you with a set of vector instructions that operate on vector values and/or vector masks. (These functions don’t translate to real CPU vector instructions, instead we simulate these operations for you in our library, and provide feedback that makes for easier debugging.)

As an example of using the PP intrinsics, a vectorized version of the <code>abs()</code> function is given in <code>main.cpp</code>. This example contains some basic vector loads and stores and manipulates mask registers. Note that the <code>abs()</code> example is only a simple example, and in fact the code does not correctly handle all inputs! (We will let you figure out why!) You may wish to read through all the comments and function definitions in <code>PPintrin.h</code> to know what operations are available to you.

Here are few hints to help you in your implementation:

<ul>

 <li>Every vector instruction is subject to an optional mask parameter. The mask parameter defines which lanes whose output is “masked” for this operation. A 0 in the mask indicates a lane is masked, and so its value will not be overwritten by the results of the vector operation. If no mask is specified in the operation, no lanes are masked. (Note this equivalent to providing a mask of all ones.)</li>

 <li><em>Hint:</em> Your solution will need to use multiple mask registers and various mask operations provided in the library.</li>

 <li><em>Hint:</em> Use <code>_pp_cntbits</code> function helpful in this problem.</li>

 <li>Consider what might happen if the total number of loop iterations is not a multiple of SIMD vector width. We suggest you test your code with <code>./myexp -s 3</code>.</li>

 <li><em>Hint:</em> You might find <code>_pp_init_ones</code> helpful (use it to initialize any mask!).</li>

 <li><em>Hint:</em> Use <code>./myexp -l</code> to print a log of executed vector instruction at the end. Use function <code>addUserLog()</code> to add customized debug information in log. Feel free to add additional <code>PPLogger.printLog()</code> to help you debug.</li>

</ul>

The output of the program will tell you if your implementation generates correct output. If there are incorrect results, the program will print the first one it finds and print out a table of function inputs and outputs. Your function’s output is after “output = “, which should match with the results after “gold = “. The program also prints out a list of statistics describing utilization of the PP fake vector units. You should consider the performance of your implementation to be the value “Total Vector Instructions”. (You can assume every PP fake vector instruction takes one cycle on the PP fake SIMD CPU.) “Vector Utilization” shows the percentage of vector lanes that are enabled.

See the <a href="#requirements">requirements</a> to finish this part.

<blockquote>

 The following part is not required for this assignment, but you are encouraged to do it for practice.

 Once you have finished part 1, it is time for vectorizing the code using real SIMD intrinsics and see if the program can really get the benefits from vectorization. Vectorize the same piece of code in part 1 so it can be run on a machine with SIMD vector instructions.

 Intrinsics are exposed by the compiler as (inline) functions that are not part of any library. Of course the SIMD intrinsics depend on the underlying architecture, and may differ from one compiler to other even for a same SIMD instruction set. Fortunately, compilers tend to standardize intrinsics prototype for a given SIMD instruction set, and we only have to handle the differences between the various SIMD instruction sets.

</blockquote>

<h2 id="2part2vectorizingcodewithautomaticvectorizationoptimizations">2. Part 2: Vectorizing Code with Automatic Vectorization Optimizations</h2>

Take the exercises below and answer questions <a href="#Q2-1">Q2-1</a>, <a href="#Q2-2">Q2-2</a>, and <a href="#Q2-3">Q2-3</a>.

We are going to start from scratch and try to let the compiler do the brunt of the work. You will notice that this is not a “flip a switch and everything is good” exercise, but it also requires effort from the developer to write the code in a way that the compiler knows it can do these optimizations. The goal of this assignment is to learn how to fully exploit the optimization capabilities of the compiler such that in the future when you write code, you write it in a way that gets you the best performance for the least amount of effort.

Please enter the <code>part2</code> folder:

<pre><code class="shell language-shell">$ cd part2</code></pre>

Auto-vectorization is enabled by default at optimization levels <code>-O2</code> and <code>-O3</code>. We first use <code>-fno-vectorize</code> to disable automatic vectorization, and start with the following simple loop (in <code>test1.cpp</code>):

<pre><code class="cpp language-cpp">void test1(float* a, float* b, float* c, int N) {  __builtin_assume(N == 1024);  for (int i=0; i&lt;I; i++) {    for (int j=0; j&lt;N; j++) {      c[j] = a[j] + b[j];    }  }}</code></pre>

We have added an outer loop over <code>I</code> whose purpose is to eliminate measurement error in <code>gettime()</code>. Notice that <code>__builtin_assume(N == 1024)</code> tells the compiler more about the inputs of the program—say this program is used in a mobile phone and always has the same input size—so that it can perform more optimizations.

You can compile this C++ code fragment with the following command and see the generated assembly code in <code>assembly/test1.novec.s</code>.

<pre><code class="shell language-shell">$ make clean; make test1.o ASSEMBLE=1</code></pre>

You are recommended to try out Compiler Explorer, a nifty online tool that provides an “interactive compiler”. <a href="https://godbolt.org/z/5PE6x9" target="_blank" rel="noopener"><u>This link</u></a> is pre-configured for 10.0.1 version of <em>clang</em> and compiler flags from the makefile. (To manually configure yourself: select language C++, compiler version x86-64 <em>clang</em> 10.0.1 and enter flags <code>-O3 -std=c++17 -Wall -fno-vectorize</code>. A screenshot is shown below.

<img decoding="async" alt="fno-vectorize" data-recalc-dims="1" data-src="https://i0.wp.com/user-images.githubusercontent.com/18013815/93843680-3ec7d400-fccd-11ea-9c62-007db8c2f569.png?w=980&amp;ssl=1" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

 <noscript>

  <img decoding="async" src="https://i0.wp.com/user-images.githubusercontent.com/18013815/93843680-3ec7d400-fccd-11ea-9c62-007db8c2f569.png?w=980&amp;ssl=1" alt="fno-vectorize" data-recalc-dims="1">

 </noscript>

<h3 id="21turningonautovectorization">2.1 Turning on auto-vectorization</h3>

Let’s turn the compiler optimizations on and see how much the compiler can speed up the execution of the program.

We remove <code>-fno-vectorize</code> from the compiler option to turn on the compiler optimizations, and add <code>-Rpass=loop-vectorize -Rpass-missed=loop-vectorize -Rpass-analysis=loop-vectorize</code> to get more information from <em>clang</em> about why it does or does not optimize code. This was done in the makefile, and you can enable auto-vectorization by typing the following command, which generates <code>assembly/test1.vec.s</code>.

<pre><code class="shell language-shell">$ make clean; make test1.o ASSEMBLE=1 VECTORIZE=1</code></pre>

You should see the following output, informing you that the loop has been vectorized. Although <em>clang</em> does tell you this, you should always look at the assembly to see exactly how it has been vectorized, since it is not guaranteed to be using the vector registers optimally.

<pre><code class="log language-log">test1.cpp:14:5: remark: vectorized loop (vectorization width: 4, interleaved count: 2) [-Rpass=loop-vectorize]    for (int j=0; j&lt;N; j++) {    ^</code></pre>

You can observe the difference between <code>test1.vec.s</code> and <code>test1.novec.s</code> with the following command or by changing the compiler flag on Compiler Explorer.

<pre><code class="shell language-shell">$ diff assembly/test1.vec.s assembly/test1.novec.s</code></pre>

<h3 id="22addingthe__restrictqualifier">2.2 Adding the <code>__restrict</code> qualifier</h3>

Now, if you inspect the assembly code—actually, you don’t need to do that, which is out of the scope of this assignment—you will see the code first checks if there is a partial overlap between arrays <code>a</code> and <code>c</code> or arrays <code>b</code> and <code>c</code>. If there is an overlap, then it does a simple non-vectorized code. If there is no overlap, it does a vectorized version. The above can, at best, be called partially vectorized.

The problem is that the compiler is constrained by what we tell it about the arrays. If we tell it more, then perhaps it can do more optimization. The most obvious thing is to inform the compiler that no overlap is possible. This is done in standard C by using the <code>restrict</code> qualifier for the pointers. By adding this type qualifier, you can hint to the compiler that for the lifetime of the pointer, only the pointer itself or a value directly derived from it (such as <code>pointer + 1</code>) will be used to access the object to which it points.

C++ does not have standard support for <code>restrict</code>, but many compilers have equivalents that usually work in both C++ and C, such as the <em>GCC</em>‘s and <em>clang</em>‘s <code>__restrict__</code> (or <code>__restrict</code>), and Visual C++’s <code>__declspec(restrict)</code>.

The code after adding the <code>__restrict</code> qualifier is shown as follows.

<pre><code class="cpp language-cpp">void test(float* __restrict a, float* __restrict b, float* __restrict c, int N) {  __builtin_assume(N == 1024);  for (int i=0; i&lt;I; i++) {    for (int j=0; j&lt;N; j++) {      c[j] = a[j] + b[j];    }  }}</code></pre>

Let’s modify <code>test1.cpp</code> accordingly and recompile it again with the following command, which generates <code>assembly/test1.vec.restr.s</code>.

<pre><code class="shell language-shell">$ make clean; make test1.o ASSEMBLE=1 VECTORIZE=1 RESTRICT=1</code></pre>

Now you should see the generated code is better—the code for checking possible overlap is gone—but it is assuming the data are <strong>NOT</strong> 16 bytes aligned (<code>movups</code> is unaligned move). It also means that the loop above can not assume that the arrays are aligned.

If <em>clang</em> were smart, it could test for the cases where the arrays are either all aligned, or all unaligned, and have a fast inner loop. However, it is unable to do that currently.&#x1f641;

<h3 id="23addingthe__builtin_assume_alignedintrinsic">2.3 Adding the <code>__builtin_assume_aligned</code> intrinsic</h3>

In order to get the performance we are looking for, we need to tell <em>clang</em> that the arrays are aligned. There are a couple of ways to do that. The first is to construct a (non-portable) aligned type, and use that in the function interface. The second is to add an intrinsic or three within the function itself. The second option is easier to implement on older code bases, as other functions calling the one to be vectorized do not have to be modified. The intrinsic has for this is called <code>__builtin_assume_aligned</code>:

<pre><code class="cpp language-cpp">void test(float* __restrict a, float* __restrict b, float* __restrict c, int N) {  __builtin_assume(N == 1024);  a = (float *)__builtin_assume_aligned(a, 16);  b = (float *)__builtin_assume_aligned(b, 16);  c = (float *)__builtin_assume_aligned(c, 16);  for (int i=0; i&lt;I; i++) {    for (int j=0; j&lt;N; j++) {      c[j] = a[j] + b[j];    }  }}</code></pre>

Let’s modify <code>test1.cpp</code> accordingly and recompile it again with the following command, which generates <code>assembly/test1.vec.restr.align.s</code>.

<pre><code class="shell language-shell">$ make clean; make test1.o ASSEMBLE=1 VECTORIZE=1 RESTRICT=1 ALIGN=1</code></pre>

Let’s see the difference:

<pre><code class="shell language-shell">$ diff assembly/test1.vec.restr.s assembly/test1.vec.restr.align.s</code></pre>

Now finally, we get the nice tight vectorized code (<code>movaps</code> is aligned move.) we were looking for, because <em>clang</em> has used packed <em>SSE</em> instructions to add 16 bytes at a time. It also manages <code>load</code> and <code>store</code> two at a time, which it did not do last time. The question is now that we understand what we need to tell the compiler, how much more complex can the loop be before auto-vectorization fails.

<h3 id="24turningonavx2instructions">2.4 Turning on AVX2 instructions</h3>

Next, we try to turn on AVX2 instructions using the following command, which generates <code>assembly/test1.vec.restr.align.avx2.s</code>

<pre><code class="shell language-shell">$ make clean; make test1.o ASSEMBLE=1 VECTORIZE=1 RESTRICT=1 ALIGN=1 AVX2=1</code></pre>

Let’s see the difference:

<pre><code class="shell language-shell">$ diff assembly/test1.vec.restr.align.s assembly/test1.vec.restr.align.avx2.s</code></pre>

We can see instructions with prefix <code>v*</code>. That’s good. We confirm the compiler uses AVX2 instructions; however, this code is still not aligned when using <em>AVX2</em> registers.

<blockquote>

 <strong><em><a name="Q2-1"></a>Q2-1:</em></strong> Fix the code to make sure it uses aligned moves for the best performance.

 Hint: we want to see <code>vmovaps</code> rather than <code>vmovups</code>.

</blockquote>

<h3 id="25performanceimpactsofvectorization">2.5 Performance impacts of vectorization</h3>

Let’s see what speedup we get from vectorization. Build and run the program with the following configurations, which run <code>test1()</code> many times, and record the elapsed execution time.

<pre><code class="shell language-shell"># case 1$ make clean &amp;&amp; make &amp;&amp; ./test_auto_vectorize -t 1# case 2$ make clean &amp;&amp; make VECTORIZE=1 &amp;&amp; ./test_auto_vectorize -t 1# case 3$ make clean &amp;&amp; make VECTORIZE=1 AVX2=1 &amp;&amp; ./test_auto_vectorize -t 1</code></pre>

Note that you may wish to use the workstations provided by this course, which support <em>AVX2</em>; otherwise, you may get a message like “Illegal instruction (core dumped)”. You can check whether or not a machine supports the <em>AVX2</em> instructions by looking for <code>avx2</code> in the flags section of the output of <code>cat /proc/cpuinfo</code>.

<pre><code class="shell language-shell">$ cat /proc/cpuinfo | grep avx2</code></pre>

<blockquote>

 <strong><em><a name="Q2-2"></a>Q2-2:</em></strong> What speedup does the vectorized code achieve over the unvectorized code? What additional speedup does using <code>-mavx2</code> give (<code>AVX2=1</code> in the <code>Makefile</code>)? You may wish to run this experiment several times and take median elapsed times; you can report answers to the nearest 100% (e.g., 2×, 3×, etc). What can you infer about the bit width of the default vector registers on the PP machines? What about the bit width of the <em>AVX2</em> vector registers.

 Hint: Aside from speedup and the vectorization report, the most relevant information is that the data type for each array is <code>float</code>.

</blockquote>

You may also run <code>test2()</code> and <code>test3()</code> with <code>./test_auto_vectorize -t 2</code> and <code>./test_auto_vectorize -t 2</code>, respectively, before and after fixing the vectorization issues in Section 2.6.

<h3 id="26moreexamples">2.6 More examples</h3>

<h4 id="261example2">2.6.1 Example 2</h4>

Take a look at the second example below in <code>test2.cpp</code>:

<pre><code class="cpp language-cpp">void test2(float *__restrict a, float *__restrict b, float *__restrict c, int N){  __builtin_assume(N == 1024);  a = (float *)__builtin_assume_aligned(a, 16);  b = (float *)__builtin_assume_aligned(b, 16);  c = (float *)__builtin_assume_aligned(c, 16);  for (int i = 0; i &lt; I; i++)  {    for (int j = 0; j &lt; N; j++)    {      /* max() */      c[j] = a[j];      if (b[j] &gt; a[j])        c[j] = b[j];    }  }}</code></pre>

Compile the code with the following command:

<pre><code class="shell language-shell">make clean; make test2.o ASSEMBLE=1 VECTORIZE=1</code></pre>

Note that the assembly was not vectorized. Now, change the function with a patch file (<code>test2.cpp.patch</code>), which is shown below, by running <code>patch -i ./test2.cpp.patch</code>.

<pre><code class="diff language-diff">--- test2.cpp+++ test2.cpp@@ -14,9 +14,8 @@     for (int j = 0; j &lt; N; j++)     {       /* max() */-      c[j] = a[j];-      if (b[j] &gt; a[j])-        c[j] = b[j];+      if (b[j] &gt; a[j]) c[j] = b[j];+      else c[j] = a[j];     }   }</code></pre>

Now, you actually see the vectorized assembly with the <code>movaps</code> and <code>maxps</code> instructions.

<blockquote>

 <strong><em><a name="Q2-3"></a>Q2-3:</em></strong> Provide a theory for why the compiler is generating dramatically different assembly.

</blockquote>

<h4 id="262example3">2.6.2 Example 3</h4>

Take a look at the third example below in <code>test3.cpp</code>:

<pre><code class="cpp language-cpp">double test3(double* __restrict a, int N) {  __builtin_assume(N == 1024);  a = (double *)__builtin_assume_aligned(a, 16);  double b = 0;  for (int i=0; i&lt;I; i++) {    for (int j=0; j&lt;N; j++) {      b += a[j];    }  }  return b;}</code></pre>

Compile the code with the following command:

<pre><code class="shell language-shell">$ make clean; make test3.o ASSEMBLE=1 VECTORIZE=1</code></pre>

You should see the non-vectorized code with the <code>addsd</code> instructions.

Notice that this does not actually vectorize as the <em>xmm</em> registers are operating on 8 byte chunks. The problem here is that <em>clang</em> is not allowed to re-order the operations we give it. Even though the the addition operation is associative with real numbers, they are not with floating point numbers. (Consider what happens with signed zeros, for example.)

Furthermore, we need to tell <em>clang</em> that reordering operations is okay with us. To do this, we need to add another compile-time flag, <code>-ffast-math</code>. Compile the program again with the following command:

<pre><code class="shell language-shell">$ make clean; make test3.o ASSEMBLE=1 VECTORIZE=1 FASTMATH=1</code></pre>