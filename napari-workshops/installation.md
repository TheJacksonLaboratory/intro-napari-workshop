# napari installation

napari is a Python application and package, so it requires a working Python installation.
There are multiple ways that Python installations and packages can be managed. For a detailed
breakdown, we recommend [this excellent guide and tutorial by Talley Lambert (Harvard Medical School)](https://hackmd.io/@talley/SJB_lObBi).
For the purposes of this workshop, for consistency, we will use [`conda`](https://docs.conda.io/en/latest/) as the Python, package, and environment manager.

## Installing Python using `conda`

In this tutorial, we will install Python via miniforge, a distribution of
Python based in the [conda package manager](https://docs.conda.io/en/latest/).

````{important}
If you already have anaconda, miniconda, or miniforge installed, those will work
as well. You can check using:

```bash
conda info
```

However, we recommend you that the conda version is >23.10 and that you are using 
[`conda-forge` channels](https://conda-forge.org/docs/user/introduction/). You can ensure this using:

```bash
conda update conda
conda config --add channels conda-forge
```

Once you have that set, you can skip to [the next section](setting-up-your-napari-workshop-environment).
````

1. In your web browser, navigate to the
   [miniforge page](https://github.com/conda-forge/miniforge). 
2. Scroll down to the ["Miniforge3" header of the "Downloads" section](https://github.com/conda-forge/miniforge?tab=readme-ov-file#miniforge3). Click the
   link to download the appropriate version for your operating system:
    - Windows: `Miniforge3-Windows-x86_64`
    - Mac with Intel processor (OS X, x86-64): `Miniforge3-MacOSX-x86_64`
    - Mac with M1 ("Apple silicon" (OS X, arm64): `Miniforge3-MacOSX-x86_64`
    - Linux with an x86-64 processor: `Miniforge3-Linux-x86_64`
3. Once you have downloaded miniforge installer, run it to install Python. Note: you may need 
   admin permissions during the installation process.
    - **Windows**
        1. Find the file you downloaded (`Miniforge3-Windows-x86_64.exe`) and
           double click to execute it. Follow the instructions to complete the
           installation.  
           Note: In the Advanced Installation Options there is an installer setting for adding 
           Miniforge3 to the PATH variable, we recommend that this it is checked.
        2. Once the installation has completed, you can verify it was correctly
           installed by searching for the "miniforge prompt" in your Start menu.
    - **Mac OS**
        1. Open your Terminal (you can search for it in spotlight - `cmd` +
           `space`)
        2. Navigate to the folder you downloaded the installer to. For example,
           if the file was downloaded to your Downloads folder, you would enter:

            ```bash
            cd ~/Downloads
            ```

        3. Execute the installer with the command below. You can use your arrow
           keys to scroll up and down to read it/agree to it.

            ```bash
            bash Miniforge3-MacOSX-x86_64.sh -b
            ```
            Note: when asked if you wish to initialize, we recommend that you answer `yes`.
        4. To verify that your installation worked, close your Terminal window
           and open a new one. You should see `(base)` to the left of your
           prompt.
        5. You can also initialize miniforge manually, with the command below. This makes sure
           that your terminal is set up correctly for your python installation.

            ```bash
            conda init
            ```

    - **Linux**
        1. Open your terminal application
        2. Navigate to the folder you downloaded the installer to. For example,
           if the file was downloaded to your Downloads folder, you would enter:

            ```bash
            cd ~/Downloads
            ```

        3. Execute the installer with the command below. You can use your arrow
           keys to scroll up and down to read it/agree to it.

            ```bash
             bash Miniforge3-Linux-x86_64.sh -b
            ```

        4. To verify that your installation worked, close your Terminal window
           and open a new one. You should see `(base)` to the left of your
           prompt.
        5. Finally, initialize miniforge with the command below. This makes sure
           that your terminal is set up correctly for your python installation.

            ```bash
            conda init
            ```

## Setting up your napari workshop environment

1. Open your terminal.
   - **Windows**: Open the "miniforge prompt" from your start menu
   - **Mac OS**: Open Terminal (you can search for it in spotlight - `cmd` +
     `space`)
   - **Linux**: Open your terminal application
2. We will use a virtual environment to encapsulate the Python tools used for this workshop.
   This ensures that the requirements for this workshop do not interfere with
   any other Python projects. To create the environment (named
   `napari-workshop`) with Python 3.11 in it, enter the following command:

    ```bash
    conda create -n napari-workshop -c conda-forge python=3.11 napari pyqt jupyterlab
    ```

3. Once the environment setup has finished, activate the environment:

    ```bash
    conda activate napari-workshop
    ```

    If you successfully activated the environment, you should now see
   `(napari-workshop)` to the left of your command prompt.

   ```{important}
   Remember to activate the right environment for what you need! And always 
   double-check that the activated environment is the one you want before installing any 
   packages—you never want to install into the `base` environment.
   ```

4. Test that your notebook installation is working. We will be using notebooks
   for interactive analysis. Enter the command below and it should launch the
   `jupyter lab` application in a web browser. Once you've confirmed it
   launches, close the web browser and press `ctrl+c` in the terminal window to
   stop the notebook server.

    ```bash
    jupyter notebook
    ```

5. Test your napari installation. Enter the command below and an empty napari
   viewer should open. You can close the window after it opens using the File menu. 
   Please note that it takes a bit of extra time (as much as 60 s) to launch napari 
   the first time, particularly if your computer has security and virus scan applications
   running.
    
    ```bash
    napari
    ```

```{admonition} Errors launching?
Contact the workshop instructors or you can reach out to the napari community on the [napari zulip chat](https://napari.zulipchat.com/#narrow/stream/212875-general)
```
