# Optional: Downloading and launching the completed Jupyter notebooks

The complete, executed notebooks are hosted in the materials on this website. We encourage you
to follow along with the workshop in a fresh, blank notebook. However, if you
would like to be able to run the completed notebooks locally, you can download them
and launch the `jupyter lab` application using the instructions below.

```{tip}
You can also run the completed notebooks using the `napari-workshop-browser` plugin. To do this, follow [the instructions at the bottom of the Plugins section](plugins.md#napari-workshop-browser). Note that the notebooks will only be available via the plugin temporarily, so if you want to keep them, you should download them as instructed below.
```

## Downloading the notebooks

There are two ways to download the notebooks; follow the instructions below for
either downloading a .zip file (recommended for beginners) or cloning via git.

### Downloading a .zip file
To download the notebooks as a .zip file, do the following:

1. Click the GitHub logo at the top of this page or navigate your web browser to [the workshop's github repository](https://github.com/TheJacksonLaboratory/intro-napari-workshop): https://github.com/TheJacksonLaboratory/intro-napari-workshop
2. Click the green (or blue) "Code" button to open the download menu and then
   "Download ZIP" ![download code](./resources/download_code.png)
3. Choose the location you would like to download the .zip into.
4. Open your file browser and double click on the .zip file to uncompress it.
5. You have downloaded the notebooks! Proceed to the "Launching jupyter
   notebook" section.

### Cloning via git
To clone the repository containing the tutorial materials to your computer, open
your Terminal and navigate to the folder where you will download the course
materials into. We recommend cloning the materials into your Documents folder,
but you can choose another suitable location. 

 ```bash
 cd ~/Documents
 ```

Then, clone the repository. This will download all of the files necessary for
this tutorial.

 ```bash
 git clone https://github.com/TheJacksonLaboratory/intro-napari-workshop.git
 ```

## Install additional `jupytext` dependency

Because the notebooks were converted to MyST Markdown files (with a .md extension),
to better  visualize on GitHub and provide a nice rendered look on the web,
you need to install additional dependencies to run them locally.

````{important}
Remember to activate the right environment!

```bash
conda activate intro-napari-workshop
```
````

```bash
conda install jupytext jupyterlab-myst
```


## Launch the `jupyter lab` application

Navigate to the `notebooks` subdirectory of the
`intro-napari-workshop` directory you just downloaded.

```
cd <path to intro-napari-workshop>/notebooks
```

Remember to activate the `intro-napari-workshop` conda environment if you haven't already.

```bash
conda activate intro-napari-workshop
```

To start the Jupyter application, enter:

```bash
jupyter lab
```

The Jupyter interface will open in a browser window and you will see the notebooks
in the file browser on the left.

````{important}
To open these workshop notebooks in the Jupyter interface, right click the notebook name in the file navigation panel from the Jupyter interface, and click "Open with -> Notebook".

![Right click on "intro_bioimage_visualization.md" file, and select "Open with -> Notebook"](./resources/open_with_notebook.png)

Or, as an alternative you can first convert them to normal `.ipynb` using:

```bash
jupytext –to ipynb <notebook_file>.md
```

````
