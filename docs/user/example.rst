Quick start
===========

This guide will show you the basic usage of SnakeLines pipelines on small toy example.
We assume that you have Linux system installed (tested on Ubuntu 16.04.1).

Install required dependencies
-----------------------------

Follow installation guides for `Miniconda <https://conda.io/docs/user-guide/install/index.html>`_ and `Snakemake <https://snakemake.readthedocs.io/en/stable/getting_started/installation.html>`_.
There is no need to install any other tools manually.
All required bioinformatic tools are installed automatically by SnakeLines inside dedicated virtual environment to avoid dependency conflicts with already installed tools.

Download SnakeLines scripts
---------------------------

Use git to download sources

.. code-block:: bash

   git clone https://github.com/jbudis/snakelines.git
   cd snakelines

Alternately you may `download sources directly <running.html#installation>`_.

Execute pipeline
----------------

Toy example read files and references are stored at the `/example` directory of downloaded sources.
Assuming, that sources are downloaded in /usr/local/snakelines, `variant_calling` pipeline may be executed using

.. code-block:: bash

   cd /usr/local/snakelines/example/mhv

   snakemake \
      --config snakelines_dir=/usr/local/snakelines \
      --snakefile scripts/variant_calling/germline/Snakefile