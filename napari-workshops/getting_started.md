# Getting started

This workshop will be a two-session introductory workshop to napari Python 
visualization software. The first session will start with an introduction 
to napari and the napari ecosystem. It will cover key concepts such as the 
GUI layout, viewer and layer models, as well as the ability to bidirectionally 
interact with napari from Python code. The second session will be focused on 
more advanced concepts, such as plugins and customization.

## Audience

The workshop is intended for people with some experience with (bio)image analysis. For the first session, minimal if any Python knowledge is required; however, for the second session and at least beginner experience with Jupyter notebooks and using the scientific Python ecosystem (numpy, scikit-image, etc.) is recommended.

## Goals
The aim of this workshop is to provide an introduction to bioimage analysis in
Python and `napari`. By the end of the workshop you should be able to
- use napari as an image viewer, including manipulating layers
- une napari to annotate structures of interest
- perform interactive analysis with napari in a
  [jupyter notebook](https://jupyter.org/)
- make use of the [napari plugin ecosystem](https://www.napari-hub.org/)

## Pre-workshop setup

Before the workshop, please ensure you have a running napari installation. We recommend an up-to-date `conda` installation and the proper environment for the workshop. Alternately, if you are not comfortable with Python and the command line, please install and use the napari bundled application. For both cases, see instructions [here](./installation.md).

## Workshop content

The workshop consists of [introductory slides](https://thejacksonlaboratory.github.io/intro-napari-slides/), as well as code examples in notebooks. The executed and rendered notebooks are [included in this repository](notebooks/index.md). We strongly encourage you to follow along with your own, fresh notebook. However, if you would like to down load and execute the completed notebooks locally, please see [the instructions here](notebook_setup.md). Alternately, you can install the `napari-workshop-browser` plugin and paste the following:  
`https://github.com/TheJacksonLaboratory/intro-napari-workshop/archive/refs/heads/main.zip`  

This will allow you to follow along with the notebooks used in the second session without needing to write code.  
Finally, the completed notebooks can also be run in the cloud [using Binder](launching_binder.md).