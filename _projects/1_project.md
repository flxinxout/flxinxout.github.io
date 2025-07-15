---
layout: page
title: Master Semester Project
description: Evaluation of NPU Acceleration for Offloading 5G Software Wordloads
img: assets/img/npu.jpg
importance: 1
category: work
related_publications: true
---

Neural Processing Units are recent hardware-specialized chips. Primarily composed of AI Engines and commonly incorporated in modern microprocessors, they serve as fast compute units for specific applications like Digital Signal Processing (DSP) and Machine Learning. The idea of the project was to explore the possibility to offload software 5G workloads into a NPU with XDNA AIE-ML architecture from AMD/Xilinx using MLIR-AIE, an open-source toolkit that targets AI Engines in both Versal and Ryzen AI products. Evaluating the performance compared to CPU-based implementations was a second objective. I took the liberty to benchmark against srsRAN, a 5G open-source stack, and benchmarked on Discrete Fourier Transforms (DFTs). In addition, I tried to explain to new developers how to efficiently program the AI Engines using MLIR-AIE, but also highlighted the encountered issues while programming in this architecture and toolchain. To finish, because it has been a project at the frontier of the research (very recent piece of hardware with no real projects), I've pointed the necessary documentation for new researchers who would like to dig into the software paradigm of AI Engines.

<div class="mt-3 mt-md-0">
  {% include figure.liquid
      loading="eager"
      path="assets/img/npu_arch.png"
      title="example image"
      class="img-fluid rounded mx-auto d-block z-depth-1" %}
</div>
<div class="caption">
    NPUs refer for neural processing units, and they are represented by a spatial architecture. You can see on the right an example of what is an NPU. Its a kind of array where small compute units sits waiting for computing something. They are named compute tile. They share data all over the graph, and to create a bridge with the external memory, interface or shim tiles are created. This architecture differs completely from a CPU architecture.
</div>

<h1>What about the details?</h1>

To efficiently program a NPU, I had to first experiment some developement tools, such as Ryzen AI Software and Riallto. This new paradigm brought me learning a lot on hardware-specialized chips in general, and the process of creating Kernels, which are actually functions executed on the compute units of the NPU. After weeks of learning, I chose to use MLIR-AIE, which is a MLIR-based toolchain that can target the architecture of my NPU, the XDNA/AIE-ML, specialized for ML workloads. You may ask why did I chose this architecture and not the XNDA/AIE, specialized for DSP applications? I was just not aware that Ryzen AI processors incorporate only the AIE-ML architecture (unfortunately in the ML area, that's what people want in their processors). Still, I had to continue my project with that 😅.

Now, the main reason why I chose MLIR-AIE is because I needed C++ host code. The host code is the code used at runtime that start workloads on the NPU, where you actually execute Kernels, or said differently, when you move data in and out of the NPU. In fact srsRAN is built on C++, meaning my complex data has to be handled by C++ code. Keep in mind this "complex data" because its more important than you think!

Speaking of complex data type, srsRAN uses complex floats to represent its data. The AIE-ML architecture is specialized for low-precision arithmetics, presenting only bfloat16 as floating point data type available. Its a 8b mantissa float that can be used by the toolchain but has to be handled on the srsRAN side by casting the float's in bfloat's using libraries that implements this type: tatata I present Eigen 😂. Even if its a linear algebra library, it implements a bfloat on C++17, the version of srsRAN (C++ introduced it only on C++23).

<div class="mt-3 mt-md-0">
  {% include figure.liquid
      loading="eager"
      path="assets/img/aie_arch.png"
      title="image"
      class="img-fluid rounded mx-auto d-block z-depth-1" %}
</div>
<div class="caption">
    XDNA spatial architecture, where CTs (with the AIE-ML type of AI Engine) are shown interconnected by different communication systems.
</div>

<h2>Challenges</h2>

MLIR-AIE brought many challenges: it is based on a open-source compiler called Peano, coming from a fork of LLVM, LLVM-AIE, targeting AIE architectures. After weeks of issues to compile kernels requiring complex data types such as complex bfloat16, and after having searched every projects for any piece of code that would explain these troubles, I finally got the answer that Peano does not handle complex data types for the AIE-ML architecture (yes really...). This was a pain because how could I compute DFTs from srsRAN without complex numbers? Fortunately, AMD provides another compiler, closed-source, called Vitis AI (and initially supporting AIE architecture) that provides this support. 

Now, developing Kernels is not an easy task as it requires the developers to be aware of the hardware details of the NPU. I want to express my gratitude for AMD that provides such as huge documentation (even if its messy). Projects such as Riallto does not highlight enough the necessity to read them. They think developers will always use the minimal amount of stuff of their libraries to create kernels, to just infer models for examples, or run simple matrix multiplications. But the key issue is that when you deviate just a bit from the standard things that can be achieved with these projects, you end up with messy code that does not work, and you don't understand why. Don't know that the maximum channels in and out of a compute tile is 4, and you are sending 3 types of data to it? Then you can just pray that chatGPT will find the answer for you, because the only place where this kind of things are written is in the docs of AMD. 

The next table summarizes the important limitations that I encountered while working on this NPU with AIE-ML architecture. You may think its not that much, but with the lack of documentation provided by MLIR-AIE, I hope new developers will read my report with attention to not fall into these issues, each one takes several hours to solve. You can download my full report [📄 here](/assets/pdf/Master_Semester_Project_Report.pdf) if you want more details, and a text with less mess that this one!


<table style="width:100%; border-collapse: collapse; font-family: sans-serif;">
  <thead>
    <tr style="background-color: #004080; color: white;">
      <th style="padding: 10px 8px; text-align: left;">Name</th>
      <th style="padding: 10px 8px; text-align: left;">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>Twiddles</strong></td>
      <td style="padding: 8px;">
        Numerical values have to be computed prior to invocation of the AIE‑API’s FFT algorithm because of the strict requirement for bfloat16 data type.
      </td>
    </tr>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>Lack of radix‑based FFT</strong></td>
      <td style="padding: 8px;">
        The lack of radix‑3 and radix‑5 implementation for AIE‑ML architecture lets us compute N‑point DFTs only for N = 2ᵏ, for k < 2. Back‑to‑back ssrRAN uses N = 754, advanced considerations have to be done. 
      </td>
    </tr>
     <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>Floating point data type</strong></td>
      <td style="padding: 8px;">
        XDNA AIE‑ML architecture does not support float32, but only bfloat16, which has lower precision. Float32 can be emulated in software, but slows down the computations.
      </td>
    </tr>
     <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>Complex data type</strong></td>
      <td style="padding: 8px;">
        LLVM‑AIE does not support FFT intrinsics of AIE‑API for complex types. The use of xchesscc coming with the Vitis Tools from AMD/Xilinx is mandatory. 
      </td>
    </tr>
     <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>XRT’s kernel overhead</strong></td>
      <td style="padding: 8px;">
       After experimentation, XRT has an overhead between 30 and 60𝜇s when running a single kernel. The CPU takes around 3𝜇s to run one DFT. Running one DFT at a time was necessary to benchmark against srsRAN, as they compute one DFT at a time. Several solutions are discussed in the Result section, involving OpenCL for example.
      </td>
    </tr>
     <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>Trace in MLIR‑AIE</strong></td>
      <td style="padding: 8px;">
        Tracing is not implemented in IRON API. Any IRON file has to be converted using CTM implementation.              
      </td>
    </tr>
     <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>xchesscc & instructions file</strong></td>
      <td style="padding: 8px;">
        When switching from Peano to xchesscc compiler, we highlight that the instructions file for the NPU is not hexadecimal, a requirement from XRT.          
      </td>
    </tr>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong>DMA channels in CTs</strong></td>
      <td style="padding: 8px;">
        The number of input and output DMA channels of the tiles are an absolute hardware limitation. It limits the number of ObjectFIFOs, taking each a channel. As mentioned in the introduction, two in and two out DMA channels are available.        
      </td>
    </tr>
  </tbody>
</table>

<h1>What has been the final setup?</h1>

With MLIR-AIE, I got the chance to finish my kernel for computing DFTs of size N=512. The next image is the final schema of my setup

<div class="mt-3 mt-md-0">
  {% include figure.liquid
      loading="eager"
      path="assets/img/global_arch.jpg"
      title="image"
      class="img-fluid rounded mx-auto d-block z-depth-1" %}
</div>
<div class="caption">
    General overview of how MLIR-AIE and XRT have been used to run a DFT kernel. In the Host, the configuration of the tiles & the kernel are processed by MLIR-AIE, resulting in multiple files. The C++ Host code using XRT (Xilinx Runtime) synchronizes the data to be processed by the kernel. The data is managed by two ObjectFIFOs of_in and of_out.
</div>

I don't want to give too much details here, but the idea is pretty simple when you said it loud. The confguration of the tiles is done via MLIR-AIE. It setups an IT to bridge with external memory, and a CT to compute a DFT. They are linked with ObjectFIFOs, abstract elements implementing first-in first-out buffers to send data across the NPU. In addition, the toolchain outputs a binary file (xclbin) and instructions file that are used by a runtime called XRT to let the host code actually execute the DFT kernel on the NPU.