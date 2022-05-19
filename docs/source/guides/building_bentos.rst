.. _building-bentos-page:

Bento Format
============

BentoML is a standard file format that describes how to load and run
a ``bentoml.Service`` defined by the user. It includes code that
instantiates the ``bentoml.Service`` instance, as well as related
configurations, data/model files, and dependencies. 

Here's an example of a ``bentofile.yaml`` configuration file:

.. code:: yaml

   service: "service:svc"
   description: "file: ./README.md"
   labels:
      owner: bentoml-team
      stage: demo
   include:
      - "*.py"
   python:
      packages:
         - scikit-learn
         - pandas

Service
-------
The `service` parameter is a required field which uses the specified Python module and variable name to locate the 
``bentoml.Service`` instance. It's format is as follows:

`service: <your service .py filename>:<variable name of service in .py file>`

If either parameter is incorrect, the Bento will not be built properly. BentoML uses this convention to find the service, 
inspect it and then determine which models should be packed into the bento.

Description
-----------

The `description` argument's contents will be used to create the `README.md` file in the Bento archive. If not explicitly specified, 
the build will search for the presence of a `README.md` file in the current working directory and set its contents as the
description.

Labels
------
The `labels` argument creates a key value map which sets labels on the bento so that you can add your own custom descriptors to the Bento.

Additional Models
-----------------

The build automatically identifies the models and their versions based on the
:ref:`service definition <service-definition-page>`. When the service definition loads runners through
the framework specific `load_runner()` function, the build will identify the model through the tag
provided in the arguments. Use the `additional_models`` keyword argument to include models tags that
are used in customer `runners`.

Building Bentos
===============

A Bento can be built with the ``bentoml build`` command and the ``bentofile.yaml`` configuration file. 
Once built, they are added to the local Bento store and can be managed with both Python APIs and the CLI.

.. code-block:: bash

    > bentoml list # list all bentos in the store
    > bentoml get iris_classifer:latest # get the description of the bento

The build options by default work for the most common cases but can be further customized by calling
the `set_build_options()` function on the service. Let's explore the available options. See documentation
for in-depth details on build options.


Configuring files to include
----------------------------

In the sample configuration file above, the ``*.py`` is including every Python file in
the working directory.

You can also include other wildcard and directory matching.

.. code:: yaml

   ...

   include:
      - "data/"
      - "**/*.py"
      - "config/*.json"

If the include field is not specified, BentoML, by default, will include
every file in the working directory. Try to limit the amount of files that
are included in your bento. For example, if unspecified, or if * is
specified, all Git versioning in the directory could be included in the
Bento by accident.

Configuring files to exclude
----------------------------

If the user needs to include a lot of files, another approach is to
only specify which files should be ignored.

The `exclude` keyword argument specifies the pathspecs (similar to the
.gitignore files) of the Python modules or data files to be excluded in the
build. The pathspecs are relative the current working directory. Users can
also opt to place a `.bentoignore` file in the directory where `bentoml build`
is run to achieve the same file exclusion during build. If not explicitly
specified, nothing is excluded from the build. Exclude is applied after
include.

This is what a ``.bentoignore`` file would look like:

.. code:: bash

   __pycache__/
   *.py[cod]
   *$py.class
   .ipynb_checkpoints/
   training_data/

Build your Bento
----------------

To build a Bento, simply run the following command from your project
directory that contains your ``bentofile.yaml``:

.. code:: bash

   bentoml build

By default, ``build`` will include all files in the current working
directory, while excluding the files specified in the ``.bentoignore`` file of
the same directory. It will also automatically infer all PyPI packages
that are required by the service code, and pin down the version used
in the current environment.

The ``--version`` keyword argument is used to specify the version of the Bento being built. If not explicitly
specified, the version is automatically generated based on the timestamp of the build combined with random bytes.

Although the ``bentofile.yaml`` is used for the build configuration by default, you may also specify a custom bentofile
using the ``--bentofile`` parameter.

Python Packages
===============

Whether you're using pip or conda, you can specify which Python packages
to include in your Bento by configuring them in ``bentofile.yaml``.

Python Options
--------------

There are two ways to specify packages in the Bentofile. First,
we can list packages like below. When left without a version,
pip will just use the latest release.

.. code:: yaml

   python:
     packages:
        - numpy
        - "matplotlib==3.5.1"

The user needs to put all required Python packages for the Bento service in a
``requirements.txt``. For a project, you can run ``pip freeze > requirements.txt``
to generate a requirements file to load with BentoML.

.. code:: yaml

   python:
     requirements_txt: "requirements.txt"

Additionally, there are more fields that can help manage larger projects.

.. code:: yaml

   python:
     requirements_txt: "requirements.txt"
     lock_packages: False
     index_url: "https://example.org/"
     no_index: False
     trusted_host: "localhost"
     find_links:
        - "https://test.org/"
     extra_index_url:
        - "https://test.org/"
     pip_args: "--quiet"
     wheels:
        - "./libs/my_package.whl"

+-------------------+------------------------------------------------------------------------------------+
| Field             | Description                                                                        |
+===================+====================================================================================+
| requirements_txt  | The path to a custom requirements.txt file                                         |
+-------------------+------------------------------------------------------------------------------------+
| packages          | Packages to include in this Bento                                                  |
+-------------------+------------------------------------------------------------------------------------+
| lock_packages     | Whether to lock the packages or not                                                |
+-------------------+------------------------------------------------------------------------------------+
| index_url         | Inputs for the `--index-url` pip argument                                          |
+-------------------+------------------------------------------------------------------------------------+
| no_index          | Whether to include the `--no-index` pip argument                                   |
+-------------------+------------------------------------------------------------------------------------+
| trusted_host      | List of trusted hosts used as inputs using the `--trusted-host` pip argument       |
+-------------------+------------------------------------------------------------------------------------+
| find_links        | List of links to find as inputs using the `--find-links` pip argument              |
+-------------------+------------------------------------------------------------------------------------+
| extra_index_url   | List of extra index urls as inputs using the `≈` pip argument                      |
+-------------------+------------------------------------------------------------------------------------+
| pip_args          | Any additional pip arguments that you would like to add when installing a package  |
+-------------------+------------------------------------------------------------------------------------+
| wheels            | List of paths to wheels to include in the bento                                    |
+-------------------+------------------------------------------------------------------------------------+


Package Locking
---------------

By default, when the BentoML service generates package requirements
from the Bentofile, the package versions will be locked for easier
reproducibility. BentoML uses pip-tools to lock the packages.

If the ``requirements.txt`` includes locked packages or a configuration
you need, set the ``lock_packages`` field to False.

Pip Wheels
----------

If you're maintaining a private pip wheel, it can be included
with the ``wheels`` field.

If the wheel is hosted on a local network without TLS, you can indicate
that the domain is safe to pip with the ``trusted_host`` field.

Conda Options
-------------

Similarly to PyPi, you can use Conda to handle dependencies.

.. code:: yaml

   conda:
     dependencies:
        - "scikit-learn==1.2.0"
        - numpy
        - nltk
     channels:
        - "conda-forge"

Here, we need the conda-forge repository to install numpy with conda.
The ``channels`` field lets us specify that to the BentoML service.

In a preexisting environment, running ``conda export`` will generate
an ``environment.yml`` file to be included in the ``environment_yml``
field.

.. code:: yaml

   conda:
     environment_yml: "environment.yml"

Conda Fields
^^^^^^^^^^^^
+------------------+----------------------------------------------------------------------------------------------------------------------------------+
| Field            | Description                                                                                                                      |
+==================+==================================================================================================================================+
| environment_yml  | Path to a conda environment file to copy into the Bento. If specified, this file will overwrite any additional option specified  |
+------------------+----------------------------------------------------------------------------------------------------------------------------------+
| channels         | Custom conda channels to use. If not specified will use "defaults"                                                               |
+------------------+----------------------------------------------------------------------------------------------------------------------------------+
| dependencies     | Custom conda dependencies to include in the environment                                                                          |
+------------------+----------------------------------------------------------------------------------------------------------------------------------+
| pip              | The specific "pip" conda dependencies to include                                                                                 |
+------------------+----------------------------------------------------------------------------------------------------------------------------------+



Docker Options
--------------

BentoML makes it easy to deploy a Bento to a Docker container.

Here's a basic Docker options configuration.

.. code:: yaml

   docker:
     distro: debian
     gpu: True
     python_version: "3.8.9"
     setup_script: "setup.sh"

For the ``distro`` options, you can choose from 5:

* debian
* amazonlinux2
* alpine
* ubi8
* ubi7

This config can be explored from `BentoML's Docker page <https://hub.docker.com/r/bentoml/bento-server>`_.

The `gpu` field instructs BentoML to select a Docker base
image that contains NVIDIA drivers and the cuDNN library.

For further Docker development, you can also use a ``setup_script``
for the container. This script will run during the ``docker build``
process, as Docker containerizes the image.

For example, with NLP projects you can preinstall NLTK data with:

.. code:: shell
   # ``setup.sh``
   python -m nltk.downloader all

Docker Fields
^^^^^^^^^^^^
+-----------------+--------------------------------------------------------------------------------------------------------------------+
| Field           | Description                                                                                                        |
+=================+====================================================================================================================+
| distro          | Configure the particular OS distribution on the Docker image ["debian", "amazonlinux2", "alpine", "ubi8", "ubi7"]  |
+-----------------+--------------------------------------------------------------------------------------------------------------------+
| python_version  | Specify which Python to include on the Docker image ["3.7", "3.8", "3.9"]                                          |
+-----------------+--------------------------------------------------------------------------------------------------------------------+
| gpu             | Determine if your container will have a GPU. This is not compatible with certain distros                           |
+-----------------+--------------------------------------------------------------------------------------------------------------------+
| devel           | If you want to use the latest main branch from the BentoML repo in your Bento                                      |
+-----------------+--------------------------------------------------------------------------------------------------------------------+
| setup_script    | Is a Python or shell script that executes during docker build time                                                 |
+-----------------+--------------------------------------------------------------------------------------------------------------------+
| base_image      | Is a user-provided custom Docker base image. This will override all other custom attributes of the image           |
+-----------------+--------------------------------------------------------------------------------------------------------------------+


Conclusion
----------

The ``bentofile.yaml`` is essential when generating a Bento,
and can be as simple or in-depth as you need. All configuration
can be included in the single file, or split into other smaller
requirements files.
