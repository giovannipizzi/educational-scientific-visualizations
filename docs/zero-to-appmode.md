# From zero to App Mode
A quick list of items to be addressed in a course on how to create interactive visualizations with jupyter and the app mode

## The python package manager `pip` and the python virtual environments
- Python packages: `pip` software and the [the PyPI](https://pypi.org) website
- Installing pip on your computer: use your package manager or use the instructions from [the official documentation](https://pip.pypa.io/en/stable/installing/) to install it
- Use `pip install --ugrade pip` to upgrade pip
- If possible, favour `python3` (ideally >=3.6) over `python2`, even if for the purpose of this course, essentially the only difference will be that in `python2` one writes `print "Hello"` without brackets, while in `python3` one uses the brackets: `print("Hello")` (there are more differences of course)
- Installing `virtualenv` (and `virtualenvwrapper`)
- Use `pip install virtualenv virtualenvwrapper`
- Explanation of what `virtualenv` does, what is the benefit of `virtualenvwrapper`
- Setup of `virtualenvwrapper` (instructions [here](https://virtualenvwrapper.readthedocs.io), essentially by adding the following two lines to the `~/.bashrc` file: 
  ```bash
  VIRTUALENVWRAPPER_PYTHON=<XXX>
  source <YYY>
  ```
  where `<XXX>` must be substituted by the output of `which python` (or `which python3`, or the python version you want to use) and `<YYY>` by the output of `which virtualenvwrapper.sh`
- Creation of a new virtualenvironment named `appmode` using `mkvirtualenv appmode` (use the `-p` option to specify the python executable if needed)
- Enter the virtualenvironment with `workon appmode` (it would not be needed the first time, but you need in a new shell), exit using `deactivate`

# The python notebook: jupyter
- Let's install jupyter in the virtual environment with `pip install jupyter`
- Let's start jupyter by the command `jupyter notebook`
  - It should start the web browser; if it does not, copy the URL printed on output in a browser (it will look something like `http://localhost:8888/?token=xyz`)
- Quick explanation of the jupyter User Interface
  - File broswing interface, uploading file, creating new files and folder
  - Opening a new terminal in the browswer
  - Notes on the security of jupyter
  - What is a kernel, how to create a new jupyter notebook with a given kernel
  - What the `Logout` button does (in general do not press it)
- Quick explanation of the jupyter notebook User Interface
  - Create a new (python) notebook
  - Click on the very top where it is written `Untitled`, replace it with the filename you want (e.g. `Day1`, that will create in the current folder a `Day1.ipynb` file)
  - First Hello World: type `print("Hello world")` in the first cell, and press `Shift+ENTER` to see the output
  - Concept of cells, input and output
  - Difference between Markdown and Code cells, basic Markdown syntax (lists, bold, italics, titles, links, LaTeX), how to re-edit a Markdown cell
  - Order of execution of cells, importance of rerunning the whole notebook often, commands in the `kernel` menu (`Restart`, `Restart and clear output`, `Restart and run all`)
  - Moving cells, merging them, deleting them

## The app mode
- Close the browser and press `CTRL+C` twice in the terminal to stop also the kernel
- Install the following two packages (make sure to be in the `appmode` virtual environment, use `workon appmode` if you are not): `pip install appmode ipywidgets`
  - `ipywidgets` is a python library that provides basic widgets (buttons, labels, text boxes, sliders, drop down menus, ...) and allows to interact with them from python
  - the `appmode` is a plugin for jupyter to hide all the input cells, run the whole notebook and show only the output. In combination with `ipywidgets`, it will allow us to create interactive visualizaitons
- To install the appmode, we also need to register and activate it within jupyter, since it is going to change the behavior. You can do this by running the following two commands in bash:
  ```bash
  jupyter nbextension     enable --py --sys-prefix appmode
  jupyter serverextension enable --py --sys-prefix appmode
  ```
- Testing the appmode: open again a jupyter notebook by running `jupyter notebook`, open the notebook you created earlier (`Day1.ipynb`) or create a new one, write a cell that creates some output (for instance the Hello World example above), and then click on the new button `Enter App Mode` in the toolbar. You should be brought to the app mode and only see the output of your print command. Click on `Edit app` to go back to the standard Jupyter view.

## Widgets
- Basic widgets: Let's try some widgets (you can find the complete list [here](https://ipywidgets.readthedocs.io/en/stable/examples/Widget%20List.html))
  - First import the needed libraries at the top:
    ```python
    import ipywidgets as ipw
    from IPython.display import display
    ```
  - Create a few widgets and visualize them:
    ```python
    w_text = ipw.Text(description="Value", value="Change here")
    w_button = ipw.Button(description="Click me")
    w_label = ipw.Label(value="Initial text")
    w_slider = ipw.FloatSlider(
        description="Frequency", 
        value=3, min=1, max=10)
    
    display(w_text, w_button, w_label, w_slider)
    ```
  - Try to switch to the app mode to see how they look like (remember to save first)
  - Discussion of the need of the `display` command
  - Discussion the widgets layout: try to replace the last line with
    ```python
    display(ipw.VBox([
      ipw.HBox([
          w_text, w_button
      ]),
      w_slider,
      w_label
      ]))
    ```
  - Adding an event to a button, the concept of callbacks
  - In a new cell, add and run (once) this code
    ```python
    def clicked(the_button):
        global w_text, w_label
        msg = "Text box value: {}".format(
            w_text.value
        )
        w_label.value = msg

    w_button.on_click(clicked)
    ```
  - Try it out in the app mode to see that the text of the label is updated at every click with the current content of the text box
  - A second way to produce output is to capture it in a widget. First add a new widget in a new cell:
  ```
  w_output = ipw.Output()
  display(w_output)
  ```
  Then replace the function above with the following:
  ```python
  def clicked(the_button):
    global w_text, w_label, w_output
    msg = "Text box value: {}".format(
        w_text.value
    )
    w_label.value = msg

    w_output.clear_output(wait=True)
    with w_output:
        print(msg)
  ```
  - Explanation of the `with` statement, of `clear_output` (and of the use of `wait=True`), of the differences of the two methods
  - Observing changes of any widget: replace the function `clicked` with the original one, then add the following in a new cell:
  ```python
  def onchange(event):
      global w_output
      
      if event['type'] != 'change' or event['name'] != 'value':
          return
      
      w_output.clear_output(wait=True)
      with w_output:
          print("Slider value changed from {} to {}".format(
              event['old'], event['new']
          ))

  w_slider.observe(onchange)
  ```
  - Explanation of `observe`, of the type of `event` and its content (try to print the `event` object instead), why we are filtering only for `change type` events
  - Try this in app mode
  - Discussion that this continuously updates and calls the `onchange` function while dragging, while this might be inefficient
  - Replace in the first cell the code to generate the slider with the following:
  ```python
  w_slider = ipw.FloatSlider(
        description="Frequency", 
        value=3, min=1, max=10,
        continuous_update=False)
  ```
  - Run again in app mode and check that now the function is called only when you stop dragging
  - Mention of the `interact` shortcut to create a bunch of widgets for a given function in a shorter way, documentation [here](https://ipywidgets.readthedocs.io/en/stable/examples/Using%20Interact.html) (we will not use it in this course but it's good to know, often it is more than enough)


## Next arguments
- integration with binder: making the notebooks available in the browser to everybody, without installation
  - create GitHub account
  - create a new repository
  - commit the jupyter notebook
  - additional files to add: `requirements.txt` and `postBuild`
  - get the link from `mybinder.org` (explain syntax, in particular user `urlpath` and use `apps/<notebook_name>.ipynb`)
  - Try it out, explain build phase and caching of docker images
- simple plots with matplotlib
  - numpy arrays and basic methods: `array`, `linspace`
  - math operations on arrays: `np.sin` (and comparison with `np.zeros` + loop)
  - basic plotting: `plot method`
  - changing line color, marker color, marker style
  - adding a legend
  - inline versus notebook mode
    - digression on `!` to execute bash commands, line magics, cell magics, and `?` to get the docstring (and `??` to get the source code)
  - plot components: figure, subplot, ... (need of creating a new figure)
  - replacing lines instead of recreating the whole plot: case of `inline` and case of `notebook` (see examples [here](https://github.com/osscar-org/widget-code-input/tree/master/demos), `projectile-*` examples)
- Exercise: plotting sine and cosine functions: use a slider for `k`, a drop-down for the function: `sin` vs `cos`, and show the function `sin(kx)` or `cos(kx)` in a plot
  - two variants: with the `@interact` decorator, or manually with `widget.observe`

[//]: # (Intermediate mode: the `code-input` widget, even if probably there will not be enough time)
