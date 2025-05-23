# napari console

The napari viewer has full-featured, integrated IPython console that allows you to interact with the viewer and its layers programmatically. This is useful for performing interactive analysis, adjusting layer properties, or accessing features that do not yet have a GUI.

```{important}
The integrated Python console is only available if napari was started from a non-interactive session, meaning the command line/terminal (using `napari`), a Python script, or the napari bundle application.  
If you are already in an interactive session, like iPython or a notebook, just keep using that!
```

## Interacting with the viewer

* Open the integrated console with the first button on the row of the viewer control buttons.  
![console-button](resources/console-button.png)

```{tip}
Within the console, by default, the current viewer will be accessible as the variable `viewer`.  
```

* Get the layer list programmatically; enter:

    ```python
    viewer.layers
    ```

* You can access the individual layers by name or index. For example, to access the `nuclei` layer, you can type:

    ```python
    viewer.layers['nuclei']
    ```
or, you can use the index, remembering that the bottom-most layer is index 0:
    ```python
    viewer.layers[1]
    ```
or, because the `nuclei` layer is the top layer, meaning the last one, you can also type:
    ```python
    viewer.layers[-1]
    ```

* Layers have their data as a `data` property that contains an array. So for example, you can access the `shape` of the data array of the `nuclei` layer by typing:

    ```python
    viewer.layers['nuclei'].data.shape
    ```
As noted above, you could use the index of the layer instead of the name.

````{tip}
You can use `tab` for auto-completions and the Up
arrow to access the command history. Use a `?` to access documentation, e.g.
    
    ```
    viewer.camera?
    ```
Use `q` or `escape` to exit the help mode. 
````

* Adjust the scale of an image. If you have just the `nuclei` and `membrane` image layers, you can provide micron to pixel scale information for the `nuclei` layer as follows:

    ```Python
    viewer.layers['nuclei'].scale = [0.35, 0.2, 0.2]
    ```

* If you want to change the scale of *all of the layers*, enter the following commands one line at a time, ensuring an indent on the second line, as shown below (copy-paste should work for multi-line commands like this!): 

    ```python
    for layer in viewer.layers:  
        layer.scale = [0.35, 0.2, 0.2]
    ```

```{important} 
If your layers "disappeared", you will need to click the `home` button to reset the viewer and you may need to adjust the slider to a new slice.
```
Note that we have only changed the way the data is displayed! We have no altered tha actual data itself. You can check this by typing:

   ```python
    viewer.layers['nuclei'].data.shape
    ```
and
    ```python
    viewer.layers['nuclei'].extent
    ```
The first command will confirm that the shape of the data array hasn't changed. The second command will show you the extent of the data both in data coordinates and in the physical coordinates, which napari calls "world coordinates".


* Add a scale bar using the GUI **View** > **Scale Bar** > **Scale Bar Visible** or by typing 
`viewer.scale_bar.visible = True`  in the integrated console. 
* Add physical units of microns to the scale bar by typing `viewer.scale_bar.unit = "um"` in the integrated console.
* Try zooming in and out of the image, while watching the scale bar!

```{tip}
The integrated console really is a full-featured iPython kernel. By default, `napari` is already imported and the current viewer can be accessed using the variable `viewer`. You can use it for interactive analysis--in fact, `numpy` is already imported for you as `np`!  
You can also import any other packages you need, like [`scikit-image`](https://scikit-image.org) or [`SciPy`](https://scipy.org).  
Finally, if you launch napari from a script, the console will inherit any variables you defined in the script. 
```

## A simple thresholding example

* Threshold the `nuclei` layer data using a manual threshold of `6000`, which will yield a binary mask, by typing:

    ```python
    nuclei_thresholded = viewer.layers['nuclei'].data > 6000
    ```
* Check the output. You can check the shape and data type of the binary mask by typing:

    ```python
    nuclei_thresholded.shape
    ```
followed by:

    ```python
    nuclei_thresholded.dtype
    ```
You can also check the number of `True` voxels in the binary mask by typing:

    ```python
    np.sum(nuclei_thresholded)
    ```
* Let's visualize the result. Add the binary mask as a new Labels layer to the viewer, by typing:

    ```python
    viewer.add_labels(nuclei_thresholded)
    ```

```{important}
The result may be unexpected! The Labels layer likely "spills over" the edges of the image! This is because we re-scaled the image Layers earlier, but did not provide a scale for the Labels layer! We can  update the scale of the Labels layer as we did before:

    ```python
    viewer.layers['nuclei_thresholded'].scale = [0.35, 0.2, 0.2]
    ```
Better yet, recognizing that we want to use the scale of the `nuclei` layer, we can set the scale of the Labels layer to be the same as the `nuclei` layer by typing:

    ```python
    viewer.layers['nuclei_thresholded'].scale = viewer.layers['nuclei'].scale
    ```
Then press the `home` button to reset the view.
For future reference, we could have set the scale of the Labels layer when we added it to the viewer, by using the `scale` key word argument in the `add_labels` method:

    ```python
    viewer.add_labels(nuclei_thresholded, scale=viewer.layers['nuclei'].scale)
    ```
```