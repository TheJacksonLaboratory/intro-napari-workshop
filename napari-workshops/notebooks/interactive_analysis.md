---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.1
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# Interactive analysis with napari

One need for bioimage analysts is to interactively perform analysis on images. This interaction could be manual parameter tuning, such as adjusting thresholds, or performing human-in-the-loop analysis through clicking on specific regions of an image.

**[napari](https://napari.org/)** makes such interactive analyses easy because of its easy coupling with Python and Scientific Python ecosystem, including tools like **[numpy](https://numpy.org/)** and **[scikit-image](https://scikit-image.org/)**.

## Setup

```{code-cell} ipython3
:tags: [remove-output]

# this cell is required to run these notebooks on Binder. Make sure that you also have a desktop tab open.
import os
if 'BINDER_SERVICE_HOST' in os.environ:
    os.environ['DISPLAY'] = ':1.0'
```

As previously, we start by importing `napari`, our `nbscreenshot` utility, and instantiating an empty viewer.

```{code-cell} ipython3
import napari
from napari.utils import nbscreenshot

# Create an empty viewer
viewer = napari.Viewer()
```

Let's read the same `cells3d` image from previous lessons and view it in napari:

```{code-cell} ipython3
from skimage.data import cells3d

image_data = cells3d()  # shape (60, 2, 256, 256)

membranes = image_data[:, 0, :, :]
nuclei = image_data[:, 1, :, :]

viewer.add_image(nuclei, colormap="green")
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

## Segmentation workflow

Let's develop a segmentation workflow for the nuclei using processing utilities from scikit-image.

```{code-cell} ipython3
from skimage import feature
from skimage import filters
from skimage import measure
from skimage import morphology
from skimage import segmentation
from skimage import util
from scipy import ndimage
import numpy as np
```

First let's try and separate background from foreground using a threshold. Here we'll use an 
automatically calculated threshold using the [Li method](https://scikit-image.org/docs/dev/auto_examples/developers/plot_threshold_li.html), but feel free to substitute a different method if you prefer, like Otsu. For more information about available `skimage` thresholding methods, see the [thresholding documentation](https://scikit-image.org/docs/stable/auto_examples/applications/plot_thresholding_guide.html).

```{code-cell} ipython3
foreground = nuclei >= filters.threshold_li(nuclei)
viewer.add_labels(foreground, name='foreground')
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

```{tip}
We applied the threshold to the `nuclei` array, but we could have also used the data backing the `nuclei` Image layer in the viewer: `viewer.layers['nuclei'].data`. Further, when adding a layer using one of the `add` methods, we can assign the return to a variable to make accessing the layer easier in the future. For example, for the case of the "foreground" labels layer we could use, e.g. `fg_layer = viewer.add_labels(foreground)`. Then we can use `fg_layer.data` to access the data in the layer.
```

Now we can see that the image has some noise, which is reflected in the threshold output so lets apply a Gaussian blur to smooth it. You can of course use any other smoothing filter, such as a median filter. `napari` makes it easy to compare multiple outputs by toggling the visibility of the layers in the viewer.

```{code-cell} ipython3
blur = filters.gaussian(nuclei, sigma=10, preserve_range=True)
viewer.add_image(blur)
foreground_blur = nuclei >= filters.threshold_li(blur)
viewer.add_labels(foreground_blur, name='foreground_blur')
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

```{tip}
If you get overwhelmed, remember that you can toggle the visibility of layers! 
In the GUI you can click the eye icon to show/hide that layer or Alt/Option-click to toggle showing *only* that layer.  
You can also hide layers programmatically by setting the `visible` attribute of the layer to `False`, e.g. `viewer.layers['foreground'].visible = False`.  
Finally, you can also remove layers from the viewer programmatically using the `remove` method and passing a layer you want to remove, e.g. `viewer.layers.remove(viewer.layers['foreground'])`, or by index using the `pop` method, e.g. `viewer.layers.pop(1)` to remove the second layer from the bottom (remember that the first layer is at index 0!).
```

Notice that there is still some signal from outside the nuclei and there are some holes located 
inside the nuclei. We can use some morphological operators to clean this up. First we will do an
opening operation, which will first erode the image and then dilate it. Then, we will fill the 
holes using a hole filling algorithm and then remove any small objects. 

```{tip}
We could update the layer data in the viewer *in place* if we didn't want to create another 
object or another layer.
```

```{code-cell} ipython3
foreground_processed = morphology.binary_opening(foreground_blur)
foreground_processed = morphology.remove_small_holes(foreground_processed, area_threshold = 20**3)
foreground_processed = morphology.remove_small_objects(foreground_processed, min_size=20**3)    
viewer.add_labels(foreground_processed)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Feel free to play around with the parameters of the morphological operations to see how they affect the segmentation.  

We will now convert this binary mask into an **instance segmentation** where each nuclei is assigned a unique label by using connected-components labeling, as implemented in [`skimage`](https://scikit-image.org/docs/dev/api/skimage.measure.html#skimage.measure.label).

```{code-cell} ipython3
from skimage import measure

labels = measure.label(foreground_processed)

viewer.add_labels(
    labels,
    opacity=0.5,
)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

As you can see, we have all the objects labeled, but touching objects are not separated. To do this, 
we can use a [**marker controlled watershed** approach](https://scikit-image.org/docs/dev/api/skimage.segmentation.html#skimage.segmentation.watershed). But to use this we will need to choose some locations for the markers.

There are a number of strategies that you can use to choose marker locations for the watershed step. Here we will use the distance transform of the binary mask and place markers at the local maxima of that. 

```{note}
Instead of computing the marker locations programmatically, you could instead use the GUI tools to add points to a Points layer.
```

The first step in this procedure is to calculate a distance transform on the binary mask as follows.

```{code-cell} ipython3
distance = ndimage.distance_transform_edt(foreground_processed)
viewer.add_image(distance)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Let's also compute a smoothed version of the distance transform, as it turns out this is beneficial to avoid over-segmentation. 
In this case, we will do this on the data in the viewer, in-place, rather than creating a yet a new layer, but if you wanted to explore this in more detail, you could create a new layer with the smoothed transform. Feel free to adjust the `sigma` parameter to see how it affects the segmentation.

```{code-cell} ipython3
smoothed_distance = filters.gaussian(distance, sigma=5)
viewer.layers['distance'].data = smoothed_distance
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

Now we can try and identify the centers of each of the nuclei by finding peaks of the smoothed distance transform.

```{code-cell} ipython3
peak_local_max = feature.peak_local_max(
    smoothed_distance,
    min_distance=10,
    exclude_border=False
)
```

```{code-cell} ipython3
viewer.add_points(peak_local_max, name='peaks', size=5, face_color='red', blending='translucent_no_depth')
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

```{important}
If you don't see any points, you may need to use the slider to look at other z-slices of the image or switch to 3D view. You can also click the `out of slice` option in the Points layer controls, which will show points in adjacent slices based on their diameters.
```

We can now remove any of the points that don't correspond to nuclei centers or add any new ones using 
the GUI Points layer tools. Or we could select a point using the GUI tools, but remove it 
programmatically—or some combination of both.

```{code-cell} ipython3
# uncomment these lines to remove a point programmatically
#viewer.layers['peaks'].selected_data = {5}
#viewer.layers['peaks'].remove_selected()
```

Using the markers we can now seed the watershed algorithm which will find the nuclei boundaries.
However, we do need to convert the coordinates into a labeled array first, because [that's what the `skimage` watershed algorithm expects](https://scikit-image.org/docs/stable/api/skimage.segmentation.html#skimage.segmentation.watershed).

```{note}
We'll use the Points layer data, because we may have added or removed a point!
```

```{code-cell} ipython3
markers = util.label_points(viewer.layers['peaks'].data, output_shape=nuclei.shape)
```

Now that we have our markers, we can proceed with the watershed.

```{code-cell} ipython3
nuclei_segmentation = segmentation.watershed(
    -smoothed_distance, 
    markers, 
    mask=foreground_processed
)

viewer.add_labels(nuclei_segmentation)
```

```{code-cell} ipython3
nbscreenshot(viewer)
```

We can now save our segmentation programmatically using our builtin save method.

```{code-cell} ipython3
# uncomment this line to save the segmentation to a tif file
#viewer.layers['nuclei_segmentation'].save('nuclei-automated-segmentation.tif', plugin='builtins')
```

## Visualizing measurements

We can use the `skimage.measure.regionprops_table` to make measurements using the segmentation. For the available properties to compute, please see the [`skimage.measure.regionprops` documentation](https://github.com/scikit-image/scikit-image/blob/v0.25.2/skimage/measure/_regionprops.py#L1109-L1410). Here we will compute the area, mean intensity, and solidity of each nucleus.

```{code-cell} ipython3
info_table = measure.regionprops_table(
        nuclei_segmentation,
        intensity_image=nuclei,
        properties=['label', 'area', 'mean_intensity', 'solidity'],
    )
```

```{note}
When using `regionprops` in 3D, `area` ends up being a volume in voxels. You can also pass the keyword argument `spacing` with the pixel spacing (the Layer `scale` property) in each dimension to get values in scaled units.
```

Now lets visualize the results in napari by mapping, for example, normalized volume (`area` property) to the color of the segmentations. First we will import a colormap, in this case `viridis`, and use it to get an array of colors from the normalized volumes.

```{important}
When mapping values to colors using a napari colormap, the input values should be normalized to the range [0, 1].
```

```{code-cell} ipython3
from napari.utils.colormaps import ensure_colormap

viridis = ensure_colormap('viridis')

volume_colors = viridis.map(info_table['area']/info_table['area'].max())
```

Then we can set the `colormap` property of the segmentation layer to this array of colors.

```{code-cell} ipython3
viewer.layers['nuclei_segmentation'].colormap = volume_colors
```

This will set the `color mode` of the Labels layer to `direct`, meaning that the color of each label will be determined by the corresponding value in the `colormap` array. You can use the layer controls to switch back to the `auto` mode to get the default behavior of assigning a random color to each label.

```{code-cell} ipython3
nbscreenshot(viewer)
```

Finally, lets also add all of the computed properties to the segmentation layer. Layer `features` store a table or data frame where each column represents a feature and each row represents a label (or Point or Shape). Importantly, `features` will be displayed in the status bar when you mouse over a label.

```{code-cell} ipython3
viewer.layers['nuclei_segmentation'].features = info_table
```

````{tip}
An alternative to visualizing the measurements by mapping them onto the labels is to use the helper function [`map_array` from `skimage.util`](https://github.com/scikit-image/scikit-image/blob/main/skimage/util/_map_array.py#L4-L73). This function will create a new array where each label is replaced by the corresponding value from the provided mapping. This can then be added as a new image layer in the viewer!

```python
from skimage.util import map_array

viewer.add_image(map_array(input_arr=nuclei_segmentation, input_vals=info_table['label'], output_vals=info_table['area']))
```
````

## Conclusions

We've now seen how to interactively perform analyses by adding data to the napari viewer and exploring different filters or workflow parameters as we moved through an analysis workflow. We've also seen how we can visualize derived data, such as segmentation results or measurements, in the viewer.
This is a powerful way to explore data and develop analysis workflows, as it allows you to quickly iterate on your analysis and visualize the results in real-time.
