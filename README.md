# test_ws

Readme
======

## 1. tuto_sphinx_doxygen

Small project to see how to build the documentation of an hybrid project
C++/python

## 2. Getting started

In order to build the documentation of the package one can use CMake or the
Makefile directly.

### 2.1 Using CMake

The classic way of using CMake is the following after changing directory
to this package:

    cd _build
    cmake ..
    make
    firefox docs/sphinx/html/index.html
    
 ## 3. Advanced explanation on the tools

In order to build the documentation we need to setup the following tools:
- [Doxygen](http://www.doxygen.nl/) the C++ api documentation parser,
- [Breathe](https://breathe.readthedocs.io/en/latest/) a sphinx extension that
    parse the doxygen xml output into restructured text files,
- [recommonmark](https://recommonmark.readthedocs.io/en/latest/) a sphinx
    extension parsing markdown files.
- [sphinx-apidoc](http://www.sphinx-doc.org/en/master/man/sphinx-apidoc.html)
    the Python api documentation parser,
- [Sphinx](http://www.sphinx-doc.org/en/master/) the documentation renderer,

### 3.1 Doxygen

In order to execute to generate the C++ API documentation we use the
Doxygen tool.
We wrote a `Doxyfile`, used to parameter Doxyygen, to notably:

- Output the files in the `_build/docs/doxygen` folder with the
  `OUTPUT_DIRECTORY` parameter. 
- Generate a list of xml files containing the API documentation setting
the `GENERATE_XML` to `YES`.

The Makefile looks at the `Doxyfile` in `doc_config_files/doxygen/` and 
CMake configure the `Doxyfile.in` from `cmake/doxygen/`.

### 3.2 Breathe

This tool is a module of sphinx that parse the Doxygen xml output.
Breathe provide two import tools:

- An API that allow you to reference to the object from the Doxygen xml
  output.
- An executable `breathe-apidoc` that generates automatically the C++ API
  into ReStructed files.

In order to use it we need to add a couple of line in the `config.py`
used by Sphinx:

~~~python
extensions = [
    # ... other stuff
    'breathe', # to define the C++ api
    # ... other stuff
]
~~~

We also need to add the following variable that determine the behavior of
Breathe:

~~~python
# breath project names and paths. Here project is the name of the repos and the path is the path to the Doxygen output.
breathe_projects = { project: "../doxygen/xml" }
# Default project used for all Doxygen output (we use only one here).
breathe_default_project = project
# By default we ask all informations to be displayed.
breathe_default_members = ('members', 'private-members', 'undoc-members')
~~~

Once the `config.py` is setup we execute `breath-apidoc` on the Doxygen
xml output:

    breathe-apidoc -o $(BREATHE_OUT) $(BREATHE_IN) $(BREATHE_OPTION)

with:

- `BREATHE_OUT` the output path (`_build/docs/sphinx/breathe/`),
- `BREATHE_IN` the path to the Doxygen xml output (`_build/docs/doxygen/xml/`),
- and `BREATHE_OPTION` some output formatting option, here empty.

This breathe-apidoc will generate the list of all classes, namespace and
files in a different ReStructuredText (`.rst`) files.
We will use them to generate the final layout of the documentation.

### 3.3 sphinx-build

The final layout is managed here and build using `shpinx-build`. The tricky
thing with `sphinx-build` is that everything included needs to be in the
working directory. Therefore in the build directory we set the output of
`breathe-apidoc` and `shpinx-apidoc` to `_build/docs/sphinx`.
And inside the same folder we create a symlink that points to the source
`doc/` folder.

Therefore in order:

- The `index.rst` includes the C++ API main `.rst` files from Breath.
- Then it includes the `modules.rst` file from `sphinx-apidoc`
- And then is adds all files inside `doc/`, which, again, points toward the
  source `doc/` directory.

The command to execute is the following:

    sphinx-build -M html _build/docs/sphinx _build/docs/sphinx

This will generate the documentation website in `_build/docs/sphinx/html/`
Thefore `firefox _build/docs/sphinx/html/index.html` opens the 
documentation
