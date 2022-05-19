.. _configuration-page:

Configuration
=============

BentoML starts with an out-of-the-box configuration that works for common use cases. Advanced users can use the 
configuration to customize a variety of features, including the BentoML CLI and Python APIs. Configuration is 
best used for scenarios where the customizations can be specified once and applied to the entire project.

.. note::
    BentoML's configuration underwent a major redesign in release 0.13.0. The previous configuration set through the 
    `bentoml config` CLI command is no longer compatible with the configuration releases in 0.13.0 and above. 
    Please see the legacy configuration property mapping table below to upgrade your configuration to the new format.

The BentoML configuration is defined by a YAML file placed in a directory specified by the `BENTOML_CONFIG` 
environment variable. The example below starts the Bento server with the configuration defined in `~/bentoml_configuration.yaml`.

.. code-block:: shell

    $ BENTOML_CONFIG=~/bentoml_configuration.yml bentoml serve IrisClassifier:latest

Rather than creating a full configuration schema, users only need to specify the properties they wish to customize in a 
partial configuration. In the example below, the microbatching workers count is overridden to 4. Remaining properties will 
take their defaults values.

.. code-block:: yaml
    :caption: BentoML Configuration

    bento_server:
        port: 6000
        timeout: 120
        backlog: 1024

Throughout the BentoML documentation, features that are customizable through configuration are demonstrated 
like the example above. For a full configuration schema including all customizable properties, refer to 
the BentoML configuration template defined in 
`default_configuration.yml <https://github.com/bentoml/BentoML/blob/main/bentoml/_internal/configuration/default_configuration.yaml>`_.

Docker Deployment
-----------------

The configuration file can be mounted to the Docker container using the `-v` option and specified to the BentoML 
runtime using the `-e` environment variable option.

.. code-block:: shell

    $ docker run -v /local/path/configuration.yml:/home/bentoml/configuration.yml -e BENTOML_CONFIG=/home/bentoml/configuration.yml


.. spelling::

    customizations
    microbatching
    customizable
    multiproc
    dir
    tls
    apiserver
    uri
    gcs
