---
layout: page
title: Master Semester Project
description: Evaluation of NPU Acceleration for Offloading 5G Software Wordloads
img: assets/img/npu.jpg
importance: 1
category: work
related_publications: true
---

Neural Processing Units are recent hardware-specialized chips. Primarily composed of AI Engines and commonly incorporated in modern microprocessors, they serve as fast compute units for specific applications like Digital Signal Processing and Machine Learning. The idea of the project was to explore the possibility to offload software 5G workloads into a NPU with XDNA AIE-ML architecture from AMD/Xilinx using MLIR-AIE, an open-source toolkit that targets AI Engines in both Versal and Ryzen AI products. Evaluating the performance compared to CPU-based implementations was a second objective. I took the liberty to benchmark against srsRAN, a 5G open-source stack, and benchamrked on Discrete Fourier Transforms (DFTs). In summary, the project outcomes were details on (i) how MLIR-AIE enables developers to efficiently program the AI Engines, (ii) the encountered issues while programming in this architecture and toolchain, and (iii) the necessary documentation for new researchers who would like to dig into the software paradigm of AI Engines.

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

To efficiently program a NPU, I had to first experiment some developement tools, such as Ryzen AI Software and Riallto. This new paradigm brought me learning a lot on hardware-specialized chips in general, and the process of creating Kernels, which are actually functions executed on the compute units of the NPU. After weeks of learning, I chose to use MLIR-AIE, which is a MLIR-based toolchain that target the architecture of my NPU. But the main reason why I chose this one is because I needed C++ host code. The host code is the "client" code where you actually execute Kernels, or said differently, when you move data in and out of the NPU. In fact srsRAN is built on C++, meaning my complex data has to be handled by C++ code. 

MLIR-AIE brought many challenges: it is based on a open-source compiler called Peano, coming from the fork of LLVM LLVM-AIE, targeting AIE architectures. After weeks of issues to compile kernels requiring complex data types such as complex bfloat16, and after having searched every projects for any piece of code that would explain these troubles, I finally got the answer that Peano does not handle complex data types for the AIE-ML architecture (yes really...). This was a pain because how could I compute DFTs from srsRAN without complex numbers? Fortunately, AMD provides another compiler, closed-source, called Vitis AI that provides this support. I forgot to mention that AIE and AIE-ML are 2 kind of architecture, both coming from the XDNA design architecture, but the second is more recent and targets AI Workloads, not Signal Processing like the first one. 

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

Developing Kernels is not an easy task as it requires the developers to be aware of the hardware details of the NPU. I want to express my gratitude for AMD that provides such as huge documentation. Projects such as Riallto does not highlight enough the necessity to read them. They think developers will always use the minimal amount of stuff of their library to create kernels, just infering models for examples. But the key issue is that when you deviate just a bit from the standard things that can be achieved with these projects, you end up with messy code that does not work, and you don't understand why? Don't know that the maximum channels in and out of a compute tile is 4, and you are sending 3 types of data to it? Then you can just pray that chatGPT will find the answer for you, because the only place where this kind of things are written is in the docs of AMD. 

The next table summarizes the important limitations that I encountered while working on this NPU with AIE-ML architecture. You may think its not that much, but with the lack of documentation provided by MLIR-AIE, I hope new developers will read my report with attention to not fall into these issues. You can download my full report HERE if you want more details, and a text with less mess that this one!

<h1>What has been the final setup?</h1>

With MLIR-AIE, I got the chance to finally finish my DFT kernel for DFTs of size 512. The next image is the final schema of my setup

<div class="mt-3 mt-md-0">
  {% include figure.liquid
      loading="eager"
      path="assets/img/global_arch.jpg"
      title="image"
      class="img-fluid rounded mx-auto d-block z-depth-1" %}
</div>
<div class="caption">
    General overview of how MLIR-AIE and XRT have been used to run a DFT kernel. In the Host, the configuration of the tiles & the kernel are processed by MLIR-AIE, resulting in multiple files. The C++ Host code using XRT (Xilinx runtime) synchronizes the data to be processed by the kernel. The data is managed by two ObjectFIFOs of_in and of_out.
</div>