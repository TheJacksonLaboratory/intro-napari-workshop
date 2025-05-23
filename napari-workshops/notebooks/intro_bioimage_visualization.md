---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.16.1
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Bioimage visualization with napari

In this notebook, we will begin exploring the use of napari for bioimage visualization. 
We will exploit the bidirectional communication between the napari viewer and the Python kernel
in Jupyter notebook to have a documented and reproducible workflow.

## (Optional) Preparing to run this notebook on mybinder.org

```{code-cell} ipython3
:tags: [remove-output]

# this cell is required to run these notebooks on Binder. Make sure that you also have a desktop tab open.
import os
if 'BINDER_SERVICE_HOST' in os.environ:
    os.environ['DISPLAY'] = ':1.0'
```

## Visualizing data with napari

Let's start by importing napari and creating an empty [napari viewer](https://napari.org/tutorials/fundamentals/viewer.html)

```{code-cell} ipython3
:tags: [remove-output]

import napari
viewer = napari.Viewer()
```

Hopefully when you ran the above command a new empty napari viewer appeared in a separate window.

Unlike other jupyter widgets, napari is not embedded inside the jupyter notebook. This is because the graphical parts of napari are written in [Qt](https://www.qt.io/), making it hard to embed on the web.

Instead, we can take a screenshot of the current state of napari viewer and embed that in the notebook. This can be useful for teaching or sharing purposes where you might want to share key steps in an analysis which makes use of interactive components.

To do this, we use the `nbscreenshot` utility function

```{code-cell} ipython3
from napari.utils import nbscreenshot

nbscreenshot(viewer)
```

```{note}
Unfortunately, in contrast with the real napari viewer, these screenshots won't be interactive!
```

### Seeing our first image

There are a few different ways to load images to into our `viewer`.

- By `dragging and dropping` image files onto the viewer
- By selecting image files from the `Open File(s)` menu option
- Using the `viewer.open` command with a file path from within the notebook
- Loading the image data into an array-like object and then passing that array using the `viewer.add_image` command

For the first three options the file path will get passed through our Reader plugin interface, allowing you to easily leverage highly customized plugins for your diverse needs. The fourth option allows you complete control over loading and visualization and is most suited for when you have data already loaded into your notebook from other sources.

For example, you could explicitly load a 3D image using the `tifffile` library and then use the `add_image()` method of our existing `Viewer` object named `viewer`.

```Python
import napari
from tifffile import imread

# load the image data using tifffile
nuclei = imread('data/nuclei.tif')
viewer.add_image(nuclei)
```

However, for simplicity, we will use the [`cells3d` dataset, provided by scikit-image](https://scikit-image.org/docs/stable/api/skimage.data.html#skimage.data.cells3d). 

```{code-cell} ipython3
from skimage.data import cells3d

image_data = cells3d()  # shape (60, 2, 256, 256)

membranes = image_data[:, 0, :, :]
nuclei = image_data[:, 1, :, :]
```

Now that we have the data arrays loaded, we can directly add the data we want to visualize to the viewer.

```{code-cell} ipython3
## directly adding image data to the napari viewer
viewer.add_image(nuclei)
```

Don't forget to change windows so you can now see the viewer. By default you'll just be looking at the middle
slice of the leading axis (index 0). You should see a single slider at the bottom of the viewer that will
allow you to scroll through the rest of the z-stack. You can programmatically set the slice to display 
to the 27 slice using the API as follows:

```{code-cell} ipython3
viewer.dims.current_step = (26, 0, 0)
```

Let's take a screenshot to record this in our notebook:

```{code-cell} ipython3
nbscreenshot(viewer)
```

One of the real strengths of napari is that you have full control over all the critical layer properties both programmatically and via the GUI.

Each `layer` that is added to the `viewer` can be found in the `viewer.layers` list.

```{code-cell} ipython3
# let's see what the layers list has in it right now
print(viewer.layers)
```

The layer list can be indexed either numerically or by the layer name, which is visible in the panel in the bottom left of the viewer. This layer has the name `nuclei`, which was automatically imputed because we originally named the variable we loaded from disk `nuclei`. Pretty cool!

```{code-cell} ipython3
nuclei_layer = viewer.layers['nuclei']
first_layer = viewer.layers[0]
nuclei_layer, first_layer
```

### Color channels and blending

In the top left hand corner of the viewer we now have a control panel with controls that cover all our layers, and those that are specific to images like contrast limits and colormap.

Right clicking on the **contrast limits** slider pulls up an elongated version of the slider which you can type specific numbers into. Let's give that a try to adjust the contrast limits to `(4000, 23000)`, and let's change the colormap to `blue` using the drop down menu.

```{code-cell} ipython3
:tags: [remove-cell]

layer = viewer.layers[0]
layer.contrast_limits = (4000, 23000)
layer.colormap = 'blue'
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

If we go in and get the `nuclei` layer from our layer list we can now see and edit the values of some of the properties that we can control in the GUI. First, let's print the values of some layer visualization properties.

```{code-cell} ipython3
# let's look at the values of some of the properties on the `nuclei` layer
print('Colormap: ', viewer.layers['nuclei'].colormap.name)
print('Contrast limits: ', viewer.layers['nuclei'].contrast_limits)
print('Opacity: ', viewer.layers['nuclei'].opacity)
```

Now let's change some of them. Note that the viewer GUI will update in real time as you run this code in the notebook.

```{code-cell} ipython3
viewer.layers['nuclei'].colormap = 'red'
viewer.layers['nuclei'].contrast_limits = [26000, 40000]
viewer.layers['nuclei'].opacity = 0.9

# We can even rename the layer, but note that from now on you'll need to refer to if with its new name
viewer.layers['nuclei'].name = 'division'
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

We could have actually passed these parameters as key-word arguments to during the first `add_image` call. For example we can add another copy of the data as follows:

```{code-cell} ipython3
viewer.add_image(nuclei, contrast_limits=[1600, 50000], colormap='blue', blending='additive')
```

Setting the `blending` of the second layer to `additive` now lets us see both together, which could be useful for understanding how parts of the image relate to each other.

```{code-cell} ipython3
nbscreenshot(viewer)
```

Let's now load in an additional channel of data containing a stain for cell membranes and add them to the viewer as a new layer.

```{code-cell} ipython3
viewer.add_image(membranes, contrast_limits=[1000, 24000], colormap='green', blending='additive');
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

### Orthogonal slicing and 3D rendering
So far we've only looked at one slice. Use the slider at the bottom of the viewer to scroll through to another slice 
and see what the image looks like. For example the 27th slice looks like:

```{code-cell} ipython3
:tags: [remove-cell]

viewer.dims.current_step = (26, 0, 0)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

As this is a 3D volume, we can also use napari to look at 2D slices along other axes. If we click the `roll dimensions` 
button in the bottom left hand corner of the viewer (which looks like a 3D cube with an arrow, 3rd from the left), 
and then scrolling through to the 60th slice. We can also accomplish this using the API.

```{tip}
Remember that the 2 right-most array dimensions will be the displayed ones and that we use `zyx` convention with 0-based indexing.
```

```{code-cell} ipython3
viewer.dims.order = (2, 0, 1)  # z, y will be displayed
viewer.dims.current_step = (25, 0, 60)  # this tuple is still in the zyx order
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

In addition to doing 2D rendering, napari can also do full 3D rendering. To enable 3D mode click on the 3D rendering 
button (which looks like a wireframe of a cube, second from the left). Now the slider has disappeared as the full 3D 
volume is being rendered at the same time. If the original dataset had been 4D, say for a volumetric timeseries then 
when using 2D rendering we would have initially seen two sliders, one for Z and one for Time, and 3D rendering would 
have switched us to only seeing one slider, for Time. In this way napari can be used to visualize either 2D or 3D 
slices of full n-Dimensional data with multiple color channels.

Try playing around with some of the 3D rendering modes and parameters in the gui to get a nice 3D visualization.

You can also programmatically set the camera to a specific view. So if you
manually find a view you like, you can print the values and then set them
explicitly in the notebook. For example:

```{code-cell} ipython3
viewer.dims.ndisplay = 3
viewer.camera.angles = (-10, 10, 150)
viewer.camera.zoom = 1.75
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

```{code-cell} ipython3
:tags: [remove-cell]

# sleep to allow screenshot to finish before notebook closes
from time import sleep
sleep(3)
```

```{tip}
If you want to save the rendered canvas view to a file, instead of having it displayed in the notebook, you can use the `viewer.screenshot` method instead of using `nbscreenshot`. This is equivalent to the GUI option in **File**>**Save Screenshot...**. You can specify the path to the output file, with the extension determining the file format. By default, the `canvas_only` argument is `True`, which means that only the canvas will be saved, without any of the viewer GUI elements.
```

## Conclusions

We've now seen how to use napari to visualize 3D images, including looking at 2D slices and the full 3D image. We've also learnt how to change properties of an image layer both from the GUI and from the jupyter notebook.

The next lessons will teach us how to create napari layers of other types, perform manual annotations, interactive analyses and more!
