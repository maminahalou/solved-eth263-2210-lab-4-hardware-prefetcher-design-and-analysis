Download Link: https://assignmentchef.com/product/solved-eth263-2210-lab-4-hardware-prefetcher-design-and-analysis
<br>
<h1></h1>

In this lab you will implement and evaluate multiple different prefetchers using ChampSim [1], a trace-based microarchitectural simulator written in C++. ChampSim models a modern high-performance out-of-order (OoO) core and enables evaluating ideas on classical problems of microarchitectures like data prefetching, branch prediction and cache replacement policies. Unlike the timing simulator you used in previous labs, ChampSim does not natively run an executable/assembly program but uses an instruction trace, extracted out from an application, to simulate the processor microarchitecture model.

We will provide you the program instruction traces from SPEC CPU 2006 and 2017 [2, 3] as well as ChampSim’s source code with a baseline branch predictor and cache replacement policy. We will also provide a high-level prefetcher API. Your job is to use this API to implement different prefetching algorithms and evaluate the impact of your prefetcher implementations on the supplied program traces. An efficient prefetcher will reduce the number of cycles a program trace takes to execute.

<h1>2.      Lab Resources</h1>

<h2>2.1.      How To Get Started</h2>

Clone the ChampSim repository.

$ git clone https://gitlab.ethz.ch/rahbera/champsim.git

You need to run <sub>build champsim.sh</sub>, which is a shell script to build the simulator. The script takes a single input parameter that is the prefetching algorithm name. We included three sample L2 prefetchers to help you get started. Compile ChampSim using the following command. This should create the ChampSim executable inside <sub>bin/ </sub>directory.

$ ./build champsim.sh next line

You need program instruction traces to run ChampSim. You can download all the traces in your local machine by executing the following command (Note: You need <sub>∼2 </sub>GB of free space to download the traces). This script takes <sub>lab traces.txt </sub>as input to download the traces. Windows users can easily find a work around of the bash script.

$ cd scripts/

$ ./download traces.sh

Your prefetchers will be evaluated using all 16 traces from <sub>lab traces.txt </sub>only. However, there is an additional trace list <sub>lab traces extended.txt </sub>with 10 more traces. Feel free to test your prefetchers on these traces too.

To run ChampSim, execute run champsim.sh.

$ ./run champsim.sh &lt;BINARY&gt; &lt;WARMUP INST&gt; &lt;SIM INST&gt; &lt;TRACE&gt;

This script takes five arguments:

<table width="41">

 <tbody>

  <tr>

   <td width="14">BI</td>

   <td width="27">NARY</td>

  </tr>

 </tbody>

</table>

1.: The full path of the executable

<ol start="2">

 <li><sub>WARMUP INST</sub>: Number of instructions (in millions) to warmup (Default: 1) 3. <sub>SIM INST</sub>: Number of instructions (in millions) to simulate (Default: 10) 4. <sub>TRACE</sub>: Full path of the trace file to execute.</li>

</ol>

Unless otherwise specified, you will always use 100 as <sub>WARMUP INST </sub>(i.e., warming up for <sub>100 </sub>M instructions) and 500 as <sub>SIM INST </sub>(i.e., simulating <sub>500 </sub>M instructions) for all simulations.

<table width="34">

 <tbody>

  <tr>

   <td width="14">Eu</td>

   <td width="20">ler</td>

  </tr>

 </tbody>

</table>

NOTE: Each trace simulation might take around an hour to get completed. Hence, you might use the ETH cluster [4] for launching your runs. Euler uses <sub>bsub </sub>to launch and manage jobs. Please go through the Euler website to learn more.

ChampSim metrics. ChampSim reports an array of metrics at the end of simulation. Some metrics to look for while evaluating the L2 prefetcher are: number of cycles, L1D average miss latency, number of L2 loads/hits/misses, number of L2 prefetch requests and useful prefetch requests.

<h2>2.2.      Source Code</h2>

We will briefly walk you through the major directories of ChampSim and explain their functionality. Do <sub>NOT </sub>modify any of these files or directories unless explicitly mentioned.

<ul>

 <li><strong><sub>inc</sub></strong>: This directory contains all the header files. These headers will automatically get included during the compilation. <em>Do not change any parameters in these files</em>. If you create any header file by yourself while implementing prefetchers, you need to put it here.</li>

 <li><strong><sub>src</sub></strong>: This directory contains files that define the microarchitecture of an OoO core, uncore, cache and DRAM controller. You might skim through these files to get a better understanding of ChampSim.</li>

</ul>

<table width="74">

 <tbody>

  <tr>

   <td width="14"><strong>re</strong></td>

   <td width="33"><strong>place</strong></td>

   <td width="27"><strong>ment</strong></td>

  </tr>

 </tbody>

</table>

<ul>

 <li><strong><sub>branch </sub></strong>and: These directories contain the source code of different branch predictors and last level cache (LLC, in our case it is L3) management policies. You will use the perceptron branch predictor [5] and SHiP [6] cache management policy for the LLC in this assignment.</li>

 <li><strong><sub>prefetcher</sub></strong>: This is the main directory you will work with. ChampSim is extensible to implement prefetchers at all three cache levels, but you will only focus on implementing prefetchers at L2. The L2 prefetcher API is defined in <sub>l2c prefetcher.cc </sub> The API consists of five key functions:

  <ol>

   <li><strong><sub>l2c prefetcher initialize</sub></strong>: This function is called when the cache gets created and should be used to initialize any internal data structures of the prefetcher.</li>

   <li><strong><sub>l2c prefetcher final stats</sub></strong>: This is the final function that is called at the end of simulation. This can be a good place to print overall statistics of the prefetcher.</li>

   <li><strong>l2c prefetcher operate</strong>: This function is called for <em>each L2 lookup operation</em>. This means it is called for both L2 load and store accesses that can either hit or miss in the cache. The third and fourth arguments of this function helps in identifying the type of lookup. In this lab, you will focus on finding patterns only in <em><sub>L2 load accesses</sub></em>. The first and second arguments provide the cacheline aligned address and the PC of the memory access, respectively.</li>

   <li><strong>l2c prefetcher cache fill</strong>: This function is called for <em>each L2 fill operation</em>. The function argument names are self explanatory.</li>

  </ol></li>

</ul>

<table width="81">

 <tbody>

  <tr>

   <td colspan="2" width="81">pr fill level</td>

  </tr>

  <tr>

   <td width="54">FILL LLC</td>

   <td width="28"> </td>

  </tr>

 </tbody>

</table>

<ol start="5">

 <li><strong><sub>prefetch line</sub></strong>: You do <em><sub>NOT </sub></em>need to implement this function, but to use it when a prefetch request needs to be injected into the memory hierarchy. (Tip: see how how this function is used in next line.l2c perf or ip stride.l2c perf). The first two arguments are the PC and the cacheline address of the access that triggers this prefetch request. The third argument is the cacheline address of the prefetch request. By default, a prefetch request generated by the prefetcher at a cache level <em><sub>N </sub></em>first looks up the <em><sub>N</sub></em><em><sub>th</sub></em>-level cache. On a miss, the prefetch request looks up the next cache levels (<em><sub>N </sub></em><sub>+1<em>,N </em>+2 </sub>etc) and eventually goes to main memory if it misses in the LLC. Once the memory supplies the corresponding data, the prefetched cachelines gets filled in all cache levels from the LLC to the <em><sub>N</sub></em><em><sub>th </sub></em> However, you can modify the prefetcher to send a hint to the memory subsystem to refrain from filling the cacheline in all cache levels if you think filling a prefetched line in all cache levels might hurt performance. This hint is passed by the fourth argument,. For an L2 prefetcher, this argument can take either of the two values FILL <sub>L2 </sub>or . For Task 1 and 2, you will set this to <sub>FILL L2</sub>, i.e., the prefetched cacheline will get filled in both in L2 and LLC. You are free to use this parameter as you deem fit in Task 3.</li>

</ol>

<h2>2.3.      Implementing a Prefetcher</h2>

To implement your own prefetcher, create a new file with <sub>&lt;prefetcher name&gt;.l2c pref </sub>naming format and define the four API functions in C++ in your own way. If you do not need to use any specific function, simply define the function with an empty body. <em>Please do not change the file </em>l2c prefetcher.cc <em>by yourself</em>. The build script will automatically generate the appropriate <sub>l2c prefetcher.cc </sub>file using your l2c pref file and link it with the ChampSim model.

To help you get started, we have provided two simple prefetcher implementations: (1) next-line prefetcher and (2) table-based IP-stride prefetcher. These are taken from the 3rd Data Prefetching Championship [7] and can be found in next line.l2c pref and ip stride.l2c pref. These files should give you a brief understanding of how to implement a prefetcher. We have also provided an empty L2 prefetcher in <sub>no.l2c pref </sub>to simulate a system without any prefetcher.

<h1>3.      Basics of Prefetching</h1>

As we discussed in depth in Lecture 18, prefetching is a speculation technique to predict a future memory request and fetch it into the caches before the processor core actually demands it. This prediction helps to <em>hide </em>the long latency of a DRAM-based main memory access and makes the program run faster. The effectiveness of a prefetcher can be measured using four metrics:

<ul>

 <li>Performance: The number of cycles saved by the prefetcher (the higher the better)</li>

 <li>Coverage: The fraction of the memory accesses saved by the prefetcher (the higher the better)</li>

 <li>Accuracy: The fraction of prefetched addresses that are actually needed later by the program (the higher the better)</li>

 <li>Timeliness: The fraction of the latency of memory accesses hidden by the prefetcher (the higher the better)</li>

</ul>

In a processor core with a traditional three level cache hierarchy, a prefetcher can be employed at any level. For example, a prefetcher at the L2 cache level tries to capture the program access pattern by observing the accesses coming out from the L1 data cache and fills the prefetched cachelines up to the L2 cache level. Many works propose prefetching algorithms [8, 9, 10, 11, 12, 13, 14, 15, 16, 17] to accurately predict future program accesses. Commercial processors use multiple prefetchers in their cache hierarchies to improve performance [18]. To learn more in depth about prefetching, please watch Professor Mutlu’s lecture video on the topic [19].

<h1>4.      Task 1/3: Implement GHB-based Stride Prefetcher</h1>

Your goal is to <em><sub>implement </sub></em>a Global History Buffer (GHB) based stride prefetcher at the L2 cache by using the prefetcher API. The prefetcher learns access patterns on cacheline addresses of loads that miss in the L1 data cache and prefetches predicted cachelines into the L2 cache. We provide the detail design of the prefetcher microarchitecture in the next subsection.

<h2>4.1.      GHB Prefetcher</h2>

The GHB prefetcher [10] was introduced in 2004. It comprises of two major structures:

<ul>

 <li>Index Table (IT): A table that is indexed by program properties, like PC (program counter), and that stores a pointer to a GHB entry.</li>

 <li>Global History Buffer (GHB): A circular queue that stores time-ordered sequence of the observed cacheline addresses. Each GHB entry also stores a pointer (called <sub>prev ptr</sub>) that points to the last cacheline address that has the same IT index. By traversing <sub>prev ptr</sub>, one can get the temporal sequence of cacheline addresses that points to the same IT entry.</li>

</ul>

Stride Prefetching Algorithm. For each L2 cache access (both hit and miss), the algorithm uses the PC of the access to index into the IT and insert the cacheline address (say <em><sub>A</sub></em>) into the GHB. Using the PC and the link pointers in GHB entries, the algorithm retrieves the sequence of last 3 addresses by this PC that accessed L2 cache. The stride is computed by taking the difference between two consecutive addresses in the sequence.

If two strides match (say <em><sub>d</sub></em>), the prefetcher simply issues prefetch requests to cachelines <em><sub>A </sub></em><sub>+ <em>ld,A </em>+ (<em>l </em>+ </sub>1)<em>d,A </em>+(<em>l </em>+2)<em>d,…,A </em>+(<em>l </em>+ <em>n</em>)<em>d</em>, where <em>l </em>is the <em>prefetch look-ahead </em>and <em>n </em>is the <em>prefetch degree</em>. For your design, please <em><sub>statically </sub></em>set both <em><sub>l </sub></em>and <em><sub>n </sub></em>to 4. Please also size the IT and GHB so that they are 256 entries each. For more detail on the GHB based stride prefetcher implementation, please read the original GHB paper [10]. Doing so will be important for you to implement Task 1 correctly.

<h1>5.      Task 2/3: Implement Feedback-directed Prefetching</h1>

Your second task is to incorporate feedback information to evaluate the effectiveness of the prefetcher in the system and accordingly adjust the prefetcher’s aggressiveness. Srinath et al. [11] propose a mechanism that incorporates dynamic feedback into the design of the prefetcher to increase the performance improvement provided by prefetching as well as to reduce the negative performance and bandwidth impact of prefetching. In Task 2, you need to extend the prefetcher you designed in Task 1 using a simplified version of Srinath et al.’s feedback directed mechanism [11]. The two dynamic feedback metrics you need to incorporate into the GHB prefetcher are described below:

<ul>

 <li><sub>Prefetch Accuracy</sub>: Prefetch accuracy is a measure of how accurately the prefetcher can predict the memory addresses that will be accessed by the program.</li>

</ul>

where Number of Useful Prefetches is the number of prefetched cache blocks that are used by demand requests while they are resident in the L2 cache. Your goal is to implement the mechanism that calculates the prefetch accuracy as described in Section 3.1.1 of [11].

<ul>

 <li><sub>Prefetch Lateness</sub>: Prefetch lateness is a measure of how timely the prefetch requests generated by the prefetcher are with respect to the demand accesses that need the prefetched data. A prefetch is defined to be late if the prefetched data has not yet returned from main memory by the time a load or store instruction requests the prefetched data.</li>

</ul>

The prefetch lateness is defined as:

In general, prefetch lateness decreases as the prefetcher becomes more aggressive. Your goal is to implement the mechanism that calculates the prefetch lateness as described in Section 3.1.2 of [11].

<ul>

 <li><sub>DO NOT </sub>implement the cache pollution metric proposed in [11]. You can do it, if you wish, as a part of Task 3.</li>

</ul>

The feedback information should be collected <sub>every 1000 instructions</sub>. Based on the collected feedback you need to update a counter that tunes the aggressiveness of the GHB prefetcher based on the information provided in Tables 1 and 2 below. The <em><sub>prefetch distance </sub></em>is defined in [11] in Section 2.1. The threshold values which define the accuracy and the lateness of the prefetcher can be found in Section 4.3 in [11]. <sub>DO NOT </sub>implement the changes in the cache insertion policy proposed in [11]. You can do it, if you wish, as a part of Task 3.

Table 1. Prefetcher aggressiveness configurations

<table width="316">

 <tbody>

  <tr>

   <td width="75">Counter</td>

   <td width="122">Aggressiveness</td>

   <td width="68">Distance</td>

   <td width="51">Degree</td>

  </tr>

  <tr>

   <td width="75">1</td>

   <td width="122">Very Conservative</td>

   <td width="68">4</td>

   <td width="51">1</td>

  </tr>

  <tr>

   <td width="75">2</td>

   <td width="122">Conservative</td>

   <td width="68">8</td>

   <td width="51">1</td>

  </tr>

  <tr>

   <td width="75">3</td>

   <td width="122">Middle-of-the-Road</td>

   <td width="68">16</td>

   <td width="51">2</td>

  </tr>

  <tr>

   <td width="75">4</td>

   <td width="122">Aggressive</td>

   <td width="68">32</td>

   <td width="51">4</td>

  </tr>

  <tr>

   <td width="75">5</td>

   <td width="122">Very Aggressive</td>

   <td width="68">48</td>

   <td width="51">4</td>

  </tr>

 </tbody>

</table>

Table 2. Feedback counter updates based on prefetch accuracy and lateness.

<table width="407">

 <tbody>

  <tr>

   <td width="54">Case</td>

   <td width="126">Prefetch Accuracy</td>

   <td width="122">Prefetch Lateness</td>

   <td width="105">Counter Update</td>

  </tr>

  <tr>

   <td width="54">1</td>

   <td width="126">High</td>

   <td width="122">Late</td>

   <td width="105">Increment</td>

  </tr>

  <tr>

   <td width="54">2</td>

   <td width="126">High</td>

   <td width="122">Not-Late</td>

   <td width="105">No Change</td>

  </tr>

  <tr>

   <td width="54">3</td>

   <td width="126">Medium</td>

   <td width="122">Late</td>

   <td width="105">Increase</td>

  </tr>

  <tr>

   <td width="54">4</td>

   <td width="126">Medium</td>

   <td width="122">Not-Late</td>

   <td width="105">No Change</td>

  </tr>

  <tr>

   <td width="54">5</td>

   <td width="126">Low</td>

   <td width="122">Late</td>

   <td width="105">Decrement</td>

  </tr>

  <tr>

   <td width="54">6</td>

   <td width="126">Low</td>

   <td width="122">Not-Late</td>

   <td width="105">No Change</td>

  </tr>

 </tbody>

</table>

<h1>6.      Task 3/3: Design Your Own Prefetcher</h1>

Your goal is to come up with an L2 prefetcher implementation of your choice that beats the existing prefetchers. You might want to take a look at the prior prefetcher literature to select one for implementation, or bring your new ideas on prefetcher design. (Note: you can take inspiration from algorithms proposed in prior data prefetching championships. But <sub>DO NOT </sub>use the source code provided by them. We will specifically look

for any abuse of this constraint and you will get penalized for plagiarism.)

Implement and evaluate your prefetcher in the similar way of Task 1 and 2. Please submit a brief write-up in PDF (no more than two A4 pages excluding references), clearly describing the prefetching algorithm including references to the relevant prefetching works.

NOTE: The designers of the <em>topthree </em>highest-performing prefetching implementations will be rewarded 1.5% of their entire course grade, provided the prefetching technique is of their own. You can be inspired by past works, of course.

<h1>7.      Submission and Evaluation</h1>

<h2>7.1.      Submission</h2>

Use the corresponding assignment in Moodle (<a href="https://moodle-app2.let.ethz.ch/">https://moodle-app2.let.ethz.ch/</a><a href="https://moodle-app2.let.ethz.ch/">)</a> to submit your work.

Please prepare your submission in the following way:

<table width="92">

 <tbody>

  <tr>

   <td width="92">task3.l2c pref</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>Rename the prefetcher files from Task 1, 2 and 3 to <sub>l2c pref</sub>, <sub>fdp.l2c pref </sub>and respectively. Put them inside <sub>prefetcher </sub>directory.</li>

</ul>

<table width="67">

 <tbody>

  <tr>

   <td width="14">re</td>

   <td width="53">port.pdf</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>Include a brief write-up in PDF format as mentioned in Task 3, describing your prefetching algorithm. Rename it asand put it in the ChampSim home directory. Optionally, you may also include another short writeup describing any surprising observations that you think we should be aware of while grading your submission.</li>

 <li>Prepare a single tarball of your ChampSim repository excluding the traces directory and rename the tarball to lab4 <em>YourSurname YourName</em>.tar.gz</li>

</ul>

<h2>7.2.      Evaluation</h2>

Your prefetchers will be evaluated based on performance improvement over a baseline system without any prefetchers. We will individually compute the performance improvement for each of the 10 traces. The overall improvement will be calculated as the geometric mean of individual improvements. For the prefetcher implementation from Task 3, we will release a leaderboard, where you can check how your proposal performs as compared to other proposals from your peers.

<h1>8.      Tips</h1>

<ul>

 <li>If needed, please ask questions to the TAs via the online Q&amp;A forum in Moodle.</li>

 <li>When you encounter a technical problem, please first read the error messages. A search on the web can usually solve many debugging issues, and error messages.</li>

</ul>

<table width="94">

 <tbody>

  <tr>

   <td width="73">462.libquan</td>

   <td width="20">tum</td>

  </tr>

 </tbody>

</table>

<ul>

 <li>One easy        way         to            debug     your        prefetcher            is             to            test         it              using       the<sub>–</sub></li>

</ul>

<table width="67">

 <tbody>

  <tr>

   <td width="47">libquan</td>

   <td width="20">tum</td>

  </tr>

 </tbody>

</table>

714<sub>B.champsimtrace.xz </sub>trace.a library for the simulation of a quantum computer and is part of SPEC CPU 2006 workload suite. It provides a structure for representing a quantum register and some elementary gates. Programmatically, it iterates over a 32 MB integer array and this access pattern generates cacheline addresses with a constant stride of +1 over a physical page. You can use this workload to test your prefetcher.


