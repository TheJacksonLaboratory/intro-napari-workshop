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

# From workflow to widget: customizing napari

The napari application uses a backend for the graphical user interface (GUI) called [Qt](https://doc.qt.io). A key feature of this framework is the use of [widgets](https://doc.qt.io/qt-6/qtwidgets-index.html), which are composable, basic UI elements. napari not only utilizes these for its UI, but also enables you can add your own as `dockable` elements. In fact, the layer controls, layer list, and napari console are all such dockable containers of Qt widgets.

There are a number of ways to go about creating your own widgets, you can see [an in-depth overview in the napari documentation](https://napari.org/dev/howtos/extending/magicgui.html). By far the simplest is to rely on the fact that napari supports the use of [`magicgui`](https://pyapp-kit.github.io/magicgui/), a Python library for quick and easy building of GUIs. A key feature of `magicgui` is autogeneration of GUIs from functions and dataclasses, by mapping Python type hints to widgets.

In this module, we will implement elements of our previous workflow as functions and then use [`magicgui.magicgui`](https://pyapp-kit.github.io/magicgui/api/magicgui/#magicguimagicgui) decorator on those functions to return us compound widgets that we can use to make exploring the parameters easier in the GUI

Let's get everything set up, based on the previous notebook:

```{code-cell} ipython3
from skimage import io

nuclei_url = 'https://raw.githubusercontent.com/kevinyamauchi/napari-spot-detection-tutorial/main/data/nuclei_cropped.tif'
nuclei = io.imread(nuclei_url)

spots_url = 'https://raw.githubusercontent.com/kevinyamauchi/napari-spot-detection-tutorial/main/data/spots_cropped.tif'
spots = io.imread(spots_url)
```

```{code-cell} ipython3
import napari
from napari.utils import nbscreenshot

# create the napari viewer
viewer = napari.Viewer()

# add the nuclei image to the viewer
viewer.add_image(nuclei, colormap = 'I Forest', blending = 'minimum')

# add the spots image to the viewer
viewer.add_image(spots, colormap = 'I Orange', blending='minimum')
```

Now let's write a function that takes an array and a `sigma` value and performs the 
high-pass operation.

```{code-cell} ipython3
import numpy as np
from scipy import ndimage as ndi

def gaussian_high_pass(image, sigma):
    low_pass = ndi.gaussian_filter(image, sigma)
    high_passed_im = (image - low_pass).clip(0)
    
    return high_passed_im
```

We can test our function, in similar fashion as before:

```{code-cell} ipython3
high_passed_spots = gaussian_high_pass(spots, 2)

viewer.add_image(high_passed_spots, colormap="I Blue", blending="minimum")
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Now lets modify the function slightly, by providing type annotations and a docstring, to 
leverage napari `magicgui` integration.

````{tip}
A brief note about type hints:  
Type hints are not enforced at runtime, **but** they can still raise `NameError` exceptions if not defined or imported.
To avoid that, we are putting the napari types in quotes to make them "forward references", because we have
not imported them. Alternatively, we could have imported them. A third option is to import:

```python
from __future__ import annotations 
```

This would permit us to drop the quotes from the type hints.  
For more information, see the official Python documentation for: 
[type hints in Python](https://peps.python.org/pep-0484/), [forward references](https://peps.python.org/pep-0484/#forward-references),and [annotations](https://peps.python.org/pep-0563/).

````

```{code-cell} ipython3
from magicgui import magicgui

@magicgui
def gaussian_high_pass(image: "napari.types.ImageData", sigma: float = 2)->"napari.types.ImageData":
    """Apply a gaussian high pass filter to an image.

    Parameters
    ----------
    image : np.ndarray
        The image to be filtered.
    sigma : float
        The sigma (width) of the gaussian filter to be applied.
        The default value is 2.
    
    Returns
    -------
    high_passed_im : np.ndarray
        The image with the high pass filter applied
    """
    low_pass = ndi.gaussian_filter(image, sigma)
    high_passed_im = (image - low_pass).clip(0)
    
    return high_passed_im
```

We have our `magicgui` decorated function and we've annotated it with the napari types.
Now, the object `gaussian_high_pass` is both a (compound) widget and a callable function. 
Let's add it to the viewer.

```{code-cell} ipython3
high_pass_widget
viewer.window.add_dock_widget(gaussian_high_pass)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Notice that because we told `magicgui` that our function will use not just any numpy array, but 
specifically `ImageData`—the data of an Image layer—and that it will also return that, `magicgui`
generated UI widgets for selecting an Image layer--if you add another layer type, it won't show
up in the dropdown!
Press the `Run` button and you will see that a new Image layer is added with the results of our
function—again thanks to autogeneration from `magicgui`.

```{code-cell} ipython3
# we'll call the widget to simulate clicking `Run`
gaussian_high_pass(viewer.layers['spots'].data)
```

Note that we are just returning `ImageData`, so there is no information passed about colormaps, blending, etc. If we want to specify that, we would need to annotate as [`LayerDataTuple`](https://napari.org/stable/guides/magicgui.html#returning-napari-types-layerdatatuple).
For now you will need to manually or programmatically set any colormap/blending settings. (Let's also hide the previous filtering output.)

```{code-cell} ipython3
viewer.layers[-1].blending = "minimum"
viewer.layers[-1].colormap = "I Blue"
viewer.layers['high_passed_spots'].visible = False
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

However, if you press `Run` again, the data for that layer will be updated in place, so
you can change the `sigma` value and see the updated result.

```{tip}
Hover over the labels `image` and `sigma` -- the names of the parameters we passed to the function.
You should see tooltips with the docstring information! How cool is that?
```

Our `gaussian_high_pass` object *is the widget*, so we can easily get the value of the current setting:

```{code-cell} ipython3
gaussian_high_pass.sigma.value
```

At the same time, `gaussian_high_pass` remains a callable function. Let's call it normally, to check 
that the function is still working as expected. Remember, type hints are not enforced by Python at runtime, 
so nothing should have changed.

```{code-cell} ipython3
test_output = gaussian_high_pass(spots, 2)
test_output.shape
```

This means that if you have a script or module you can import the function and use it as normally
*or* use it as a widget in napari.

Let's make the the widget more dynamic and user-friendly, by giving `magicgui` some extra information.
Let's ask for a slider for the `sigma` parameter and lets have the function be auto-called
when the slider is changed.

But first, lets remove the previous widget.

```{code-cell} ipython3
viewer.window.remove_dock_widget("all")
```

```{code-cell} ipython3
@magicgui(auto_call=True,
            sigma={"widget_type": "FloatSlider", "min": 0, "max": 20}
         )
def gaussian_high_pass(image: "napari.types.ImageData", sigma: float = 2)->"napari.types.ImageData":
    """Apply a gaussian high pass filter to an image.

    Parameters
    ----------
    image : np.ndarray
        The image to be filtered.
    sigma : float
        The sigma (width) of the gaussian filter to be applied.
        The default value is 2.
    
    Returns
    -------
    high_passed_im : np.ndarray
        The image with the high pass filter applied
    """
    low_pass = ndi.gaussian_filter(image, sigma)
    high_passed_im = (image - low_pass).clip(0)
    
    return high_passed_im
```

```{code-cell} ipython3
viewer.window.add_dock_widget(gaussian_high_pass)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Now you can play with the slider until you get the effect you want in the GUI and then return the value:

```{code-cell} ipython3
gaussian_high_pass.sigma.value
```

Or you can *set the value*:

```{code-cell} ipython3
gaussian_high_pass.sigma.value = 3
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Finally, lets make a widget for the whole workflow as a function. Now we are starting with
image layer (data), but then we want a Points layer with points. We could again return just the
layer data using `napari.types.PointsData`. But lets get a nicer Points layer instead, so we will return a LayerDataTuple.
If `detect_spots()` returns a `LayerDataTuple`, napari will add a *new layer* to
   the viewer using the data in the `LayerDataTuple`. For more information on
   the `LayerDataTuple` type, please see
   [the docs](https://napari.org/plugins/guides.html#the-layerdata-tuple).
    - The layer data tuple should be: `(layer_data, layer_metadata, layer_type)`
    - `layer_data`: the data to be displayed in the new layer (i.e., the points
      coordinates)
    - `layer_metadata`: the display options for the layer stored as a
      dictionary. Some options to consider: `symbol`, `size`, `face_color`
    - `layer_type`: the name of the layer type as a string—in this case `'Points'`  
Also let's change the `image` argument type hint to `ImageLayer` so that we can access more
properties if we'd like or be able to more easily set the value programmatically.

```{code-cell} ipython3
# again lets remove the previous widget
viewer.window.remove_dock_widget("all")
```

```{code-cell} ipython3
import numpy as np
from skimage.feature import blob_log

@magicgui
def detect_spots(
    image: "napari.layers.Image",
    high_pass_sigma: float = 2,
    spot_threshold: float = 0.01,
    blob_sigma: float = 2
)->"napari.types.LayerDataTuple":
    """Apply a gaussian high pass filter to an image.

    Parameters
    ----------
    image : np.ndarray
        The image in which to detect the spots.
    high_pass_sigma : float
        The sigma (width) of the gaussian filter to be applied.
        The default value is 2.
    spot_threshold : float
        The threshold to be passed to the blob detector.
        The default value is 0.01.
    blob_sigma: float
        The expected sigma (width) of the spots. This parameter
        is passed to the "max_sigma" parameter of the blob
        detector.
    
    Returns
    -------
    points_coords : np.ndarray
        An NxD array with the coordinate for each detected spot.
        N is the number of spots and D is the number of dimensions.
    sizes : np.ndarray
        An array of size N, where N is the number of detected spots
        with the diameter of each spot.
    
    """
    # filter the image
    filtered_spots = gaussian_high_pass(image.data, high_pass_sigma)

    # detect the spots on the filtered image
    blobs_log = blob_log(
        filtered_spots,
        max_sigma=blob_sigma,
        num_sigma=1,
        threshold=spot_threshold
    )
    
    # convert the output of the blob detector to the 
    # desired points_coords and sizes arrays
    # (see the docstring for details)
    points_coords = blobs_log[:, 0:2]
    sizes = 2 * np.sqrt(2) * blobs_log[:, 2]

    return (points_coords, {"size": sizes, "face_color": "red"}, "Points")
```

```{code-cell} ipython3
viewer.window.add_dock_widget(detect_spots)
```

```{code-cell} ipython3
# let's call the widget/function to simulate pressing run
detect_spots(viewer.layers['spots'])
```

```{code-cell} ipython3
# lets set the dropdown value for the screenshot
detect_spots.image.value = viewer.layers['spots']
nbscreenshot(viewer)
```
