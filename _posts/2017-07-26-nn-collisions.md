---
layout: post
title: Deep Name Collisions
---

As the field of deep learning has matured, it has incorporated advancements
in many fields of scientific inquiry (ie. it has eaten them whole).  As such, practitioners
have *helpfully* integrated the language of many sub-fields, to ease the process of
learning and enhance cross-domain understanding.  

Consider for example a deep neural network algorithm.  There are some values that
are produced during the process of inference (which change based on the inputs), and other 
values that are trained ahead of time to describe the problem (which do not change).


<img src="{{ site.baseurl }}/img/nn.svg" alt="Neural Network" />

The following table describes possible names for each of these, and the originating
field.

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;}
.tg .tg-yw4l{vertical-align:top}
</style>
<table class="tg">
  <tr>
    <th class="tg-yw4l">Things that change on each input</th>
    <th class="tg-yw4l">Things that don't change on each input</th>
    <th class="tg-yw4l">Where does this shit come from?</th>
  </tr>
  <tr>
    <td class="tg-yw4l">Neurons</td>
    <td class="tg-yw4l">Synapses</td>
    <td class="tg-yw4l">Biology</td>
  </tr>
  <tr>
    <td class="tg-yw4l">Variables/Features</td>
    <td class="tg-yw4l">Parameters</td>
    <td class="tg-yw4l">Statistics</td>
  </tr>
  <tr>
    <td class="tg-yw4l">Matrix</td>
    <td class="tg-yw4l">Kernels</td>
    <td class="tg-yw4l">Linear Algebra</td>
  </tr>
  <tr>
    <td class="tg-yw4l">Image/Pixels</td>
    <td class="tg-yw4l">Filter</td>
    <td class="tg-yw4l">Image Processing</td>
  </tr>
  <tr>
    <td class="tg-yw4l">Activations</td>
    <td class="tg-yw4l">Weights</td>
    <td class="tg-yw4l">#@&amp;% you</td>
  </tr>
</table>

Because newcomers may know only some of the terminology, it is convention
that these terms may be used interchangeably;  eg. features and filters, or activations and kernels. 
Or they can be combined for enhanced readability; eg.
"neuron-feature-activations" and "kernel-parameter-filter-weights".
This makes deep learning easily accessible ... and fun!.

This is really just an example, other helpful terminology equivalences
include feature "maps" and channels, tensors and ... multidimensional arrays.

Thanks science, and happy name colliding!


