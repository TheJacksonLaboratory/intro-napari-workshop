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

```{note}
Currently (napari 0.4.19) the built-in Plugin installer uses `pip` to install plugins into
the Python environment.
```

For more information on plugins, please see the [official Plugin documentation](https://napari.org/stable/plugins/index.html).

## Install Your First Plugin  
* Navigate to [napari-hub.org](https://www.napari-hub.org)—this is the platform to discover and share plugins.  
* Search for **sample:**.
* Scroll down until you see **napari-bio-sample-data**.  
* Select **napari-bio-sample-data**.  
* Read the documentation on the plugin page.  
* In napari, open  
**Plugins**>**Install/Uninstall Plugins...**   
and wait for the plugin list to populate.  
* Search for **sample** in the top search/filter bar, and click the install button next to **napari-bio-sample-data**.  

    ```{note}
    After clicking the install button and you can monitor the **installing…** process by clicking `Show status`. After installing, you will typically need to close napari and reopen it, then  continue.
    ``` 

* Restart napari and check if the plugin installed by opening **File**>**Open Sample**.  **napari-bio-sample-data** should be a menu option.  
  
* **File**>**Open Sample**>**napari-bio-sample-data**>**3D nuclei**  

This sample data includes the same `nuclei` data as before, but this time a 3D labels and surface layer
are present. You can explore these in the 3D view.
