# (Optional) Using napari in a script

napari can be used in Python scripts much like any other module. This can be very useful
for setting a viewer, loading data, etc. for  reproducibility or for sharing a setup.

A Python script can be executed by simply typing:

```bash
python <name of script>.py
```

```{tip}
You can find a wide range of napari scripts in [the examples Gallery](https://napari.org/stable/gallery.html).
```

## Starting the Qt event loop

However, there is one important aspect, which is [the Qt event loop](https://napari.org/stable/guides/event_loop.html#an-introduction-to-the-event-loop-in-napari). napari uses a GUI backend which
"listens" to various `events` (mouse clicks, drags, functions/methods being called, etc.) to 
enable interactions. These `events` are managed in a queue called the `event loop`.   

In an interactive Python environment, like a Jupyter notebook or iPython REPL, napari will use their 
integrated Qt event loop, allowing you to interact with the napari GUI. However, when using napari 
in a script, if you want the user to be able to interact with the napari GUI, you will need to 
explicitly "start the application", blocking further execution of the script and allowing the user 
to interact the napari GUI. The way to do this is to add the following line of code to your script:

```Python
napari.run()
```

Any code before this line will be executed as normal: you can set up a viewer, add some layers, etc.
Then when this line is executed, the viewer will be shown and allow for GUI interaction, but further
script execution will be blocked. 

```{important}
Any code below the `napari.run()` line will not be executed until the viewer is closed!
```

Try this example by copy-pasting into a `.py` file in your local computing environment, e.g.
`napari_run_example.py`:

```Python
from skimage import data

import napari

# create the viewer with an image
viewer = napari.view_image(data.astronaut(), rgb=True)

# start the event loop allowing you to interact with the viewer
napari.run()

# this line will not be executed until after the viewer is closed.
print("napari was closed!")
```

Execute the script using:

```bash
python napari_run_example.py
```

You should see a viewer spawn with the astronaut image, which you can interact with.

```{important}
Ensure you can keep an eye on the terminal from which you ran the script.
```

When you close the viewer, you should see the print statement in your terminal:

```
napari was closed!
```

## A few napari scripting tips

1. If your script has classes or functions, you may want to be able to the the script as a module, so
   you can re-use them in other scripts. But remember, when you import a module, Python will execute 
   the code in it. In this case, you probably *don't* want to start the event loop. So it's a good 
   idea to put a guard around `napari.run()` to ensure that this is only executed when run as a script.
   
   ```Python
   if __name__ == '__main__':
     napari.run()
   ```

   ```{tip}
   You'll also need this if you want to [contribute your script as an example for the napari examples
   Gallery](https://napari.org/stable/developers/contributing.html#adding-examples-to-the-gallery)!
   ```

1. Remember, when you launch napari from a script, you have access to the napari console! But, by default
   you won't have access to all of the local variables from your script in the console. If you do want
   to access them, e.g. names of layer objects, etc. you can push the local variables to the console by
   including the following line at the end of your script:

   ```Python
   viewer.update_console(locals())
   ```
   
   To see this in action, check out [this example](https://napari.org/stable/gallery/update_console.html#sphx-glr-gallery-update-console-py). Run it and try to access, for example `polygons`, in the
   console. Then, comment out `viewer.update_console(locals())` and run it again and you will get a 
   **NameError**.
