---
layout: page
title: Master Semester Project
description: Evaluation of NPU Acceleration for Offloading 5G Software Wordloads
img: assets/img/npu.jpg
importance: 1
category: work
related_publications: true
---

Neural Processing Units are recent hardware-specialized chips. Primarily composed of AI Engines and commonly incorporated in modern microprocessors, they serve as fast compute units for specific applications like Digital Signal Processing and Machine Learning. I wanted to explore the possibility to offload software 5G workloads into a NPU with XDNA AIE-ML architecture from AMD/Xilinx using MLIR-AIE, an open-source toolkit that targets AI Engines in both Versal and Ryzen AI products. This project focuses on offloading Discrete Fourier Transforms from srsRAN, an open-source 5G stack, and run its benchmarks to compete with the CPU implementation. The idea is to discuss the results and the limitations encountered. In summary, the project highlights (i) how MLIR-AIE enables developers to efficiently program the AI Engines, (ii) the encountered issues while programming in this architecture and toolchain, and (iii) the necessary documentation for new researchers who would like to dig into the software paradigm of AI Engines. 

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

<h1>Introduction</h1>

Ryzen AI and Versal Adaptive SoC are two platforms that integrate NPUs. While the first one is a brand of microprocessors, and the second a general software-programmable and heterogeneous compute platform, both of them can incorporate a NPU, and more specifically AI Engines. These small units are the core part of the different NPU architectures, forming an array of compute units and capable of sharing data. Microprocessors such as Ryzen AI incorporate a small amount of these AI Engines, while the second can contain hundreds of them Taka et al., 2023. Figure below shows a spatial architecture AMD/Xilinx, 2024, representing the XDNA. It’s a high-performance architecture that uses control and data flow graphs as the computational model. It contains two different
entities: Compute Tiles (CTs) and Interface/Shim Tiles (STs). This architecture reflects a flow graph, where data moves inside it in a synchronous manner. Each CT has its own vector processor unit (VPU) and an internal memory space, allowing rapid accumulation of results in the processor’s registers. STs are gateways with the
external memory, optimizing the transfer of data inside and outside the NPU.

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

<h2>bla</h2>

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}

```html
<div class="row justify-content-sm-center">
  <div class="col-sm-8 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm-4 mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
```

{% endraw %}
