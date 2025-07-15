---
layout: page
title: Bachelor Semester Project
description: Speeding up the Quantification of Data Staleness in Dynamic Bayesian Optimization
img: assets/img/bayesian.png
importance: 2
category: work
related_publications: true
---

Some algorithms have been proposed to extend Bayesian Optimization (BO) to time-varying functions. This adaptation is known as Dynamic Bayesian Optimization (DBO). To achieve great performance, W-DBO from [3] uses a criterion that quantifies how relevant an observation is for the future predictions of the Gaussian Process. By evolving in time, the optimum of the function changes, and observations of the function become less relevant due to their lack of information on its future values. In a long-period time optimization, keeping all of them will impact the performance of the algorithm. In fact, the sampling frequency will decrease due to the growth of the Gaussian Process’ inference time. To remove them rapidly, the criterion should be calculated efficiently in
a low-level programming language. By speeding up the computations using C++ to calculate the criterion, W-DBO shown great improvements over state of the art solutions. We present in this work (i) the C++ implementation, (ii) the performance of this implementation compared to a Python implementation, (iii) the performance of W-DBO compared to the state of the art. Additionally, we develop Python packages for W-DBO and the criterion.

<div class="mt-3 mt-md-0">
  {% include figure.liquid
      loading="eager"
      path="assets/img/1.png"
      title="example image"
      class="img-fluid rounded mx-auto d-block z-depth-1" %}
</div>
<div class="caption">
    
</div>


<div class="mt-3 mt-md-0">
  {% include figure.liquid
      loading="eager"
      path="assets/img/2.png"
      title="image"
      class="img-fluid rounded mx-auto d-block z-depth-1" %}
</div>
<div class="caption">
    
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
