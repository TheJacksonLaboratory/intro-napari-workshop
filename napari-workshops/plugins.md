# napari Plugins

Plugins are Python packages that allow developers to extend napari functionality.
Currently, plugins can:
* Add file format support with readers and writers
* Add custom widgets and user interface elements for Python functions, etc.
* Provide sample data
* Change the look of napari with a theme

[The napari hub](https://www.napari-hub.org) is a repository for searching/discovering plugins.
Plugins can be installed as Python packages using conda or pip as normal or from within the
napari GUI using the Plugin menu.

For more information on plugins, please see the [official Plugin documentation](https://napari.org/stable/plugins/index.html).

## Install Your First Plugins  
* In napari, open  
**Plugins**>**Install/Uninstall Plugins...**   
* Search for **sample** in the top search/filter bar, and then, in the bottom pane, click the install button next to **napari-bio-sample-data**.
* Once the install completes, search for **skimage** and install the **napari-skimage** plugin as well.  

    ```{note}
    - After clicking the install button and you can monitor the **installing…** process by clicking `Show status`. This can be helpful in trouble-shooting installation issues.
    - You can queue multiple plugins to install at once, by searching for a new plugin and clicking the install button.
    - After installing, it is recommended that you close napari and reopen it to continue.
    ``` 

* Restart napari. 
* Open **File**>**Open Sample**. **napari-bio-sample-data** should be a menu option.  
    * Select: **File**>**Open Sample**>**napari-bio-sample-data**>**3D nuclei**  

    This sample data includes the same `nuclei` data as before, but this time a 3D labels and surface layer
    are present. You can explore these in the 3D view. Delete the surface layer once you're ready to move
    on.
* Open **Layers**>**Measure**>**Regionprops (labels) (napari-skimage)**
    * Ensure that `nuclei label` is selected for the `Label Layer` and `nuclei` is selected
    for the `Intensity Image Layer`.
    * Select some properties to measure, you can use shift or control/command to multiselect. Refer to the [scikit-image `measure.regionprops` documentation](https://scikit-image.org/docs/stable/api/skimage.measure.html#skimage.measure.regionprops) for more information
    * Click `Analyze`, which should yield a table of measurements—depending how many you checked, this could take a moment to run. You can save the results to a CSV file by clicking the `Save Results` button.

    ```{tip}
    Grab the titlebar of the Table widget, it's on the left edge, and drag it off the napari
    viewer to undock. Then you can resize the table for easier browsing.
    You can re-dock the widget by dragging it on the existing side dock or the bottom of the viewer,
    e.g. where the viewer buttons are, to dock it beneath the canvas.
    ```

    * Select the `nuclei labels` layer in the layer list and in the layer controls widget at the top, check the box `show selected` to only view the currently
    selected label. Now in the table, when you click a cell you will be shown the corresponding label. You could now use the brush and eraser tools to edit the segmentation and then re-run the analysis.


## A few other handy plugins

1. [napari-animation](https://napari.org/napari-animation/): a plugin that lets you make
   (and export) animations in napari by simply setting key frames.
1. [napari-threedee](https://napari-threedee.github.io): a plugin to make interacting and annotating in 3D easier.
1. [napari-ome-zarr](https://www.napari-hub.org/plugins/napari-ome-zarr): a reader plugin for for `zarr` backed
   OME-NGFF images (local or remote).
1. [napari-tiff](https://www.napari-hub.org/plugins/napari-tiff): a reader plugin that can open `tiff` files and variants like `svs` or `ndpi` and load them as multi-scale data.
   with multiple pages and/or channels.
1. [napari-matplotlib](https://napari-matplotlib.github.io): a plugin providing widgets with `matplotlib` based
   plots, such as histogram, scatter plot, line plot, etc. 
   
## napari-workshop-browser

For the second session, we will use Jupyter notebooks. If you are using the napari bundled application (or you just want the easiest option) you can install the `napari-workshop-browser` plugin, which will allow you to download and run the notebooks that we will use in the second session. They will be pre-completed and you can follow along without needing to write code. Once installed, go to the menu **Plugins**>**Open a napari workshop** to open the widget. Then, to download the notebooks, you can double-click the pre-filled URL and paste the following URL into the `napari-workshop-browser` plugin:
`https://github.com/TheJacksonLaboratory/intro-napari-workshop/archive/refs/heads/main.zip`
Then click the `Launch Workshop` button. This will download the notebooks, hide the running napari application, and open a Jupyter session in your browser with the notebooks ready to go.
