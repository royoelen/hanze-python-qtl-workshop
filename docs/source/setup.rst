Setup
=====

.. _installation:

Installation
------------

We are going to need some Python libraries to be able to do our eQTL mapping. The best way to do this, is by setting up a `Conda <https://docs.anaconda.com/free/miniconda/>`_ environment, and installing the libraries there:

.. code-block:: console

   (base) $ conda create -n pyqtl_env python=3.9
   (base) $ conda activate pyqtl_env
   (base) $ conda install -n pyqtl_env pip
   (pqtl_env) $


You can also use a venv or just your base Python install. Regardless, you will end up installing libraries via pip:

.. code-block:: console

   (pqtl_env) $ pip install argparse numpy scikit-learn==1.1.3 scipy pandas statsmodels


.. _jupyter:

Jupyter Notebook
------------

For visualizing our results, it is convenient to have an IDE to work with. The examples here will use Jupyter Notebook. You are free to use a different IDE, but you will be on your own with regards to making this work.

Let us install jupyter-lab

.. code-block:: console

   (pqtl_env) $ pip install jupyterlab


Now we will test if the notebook will start. First in the terminal, start the server on a specific port. The default port of 8887 is perfectly fine.

.. code-block:: console

   (pqtl_env) $ jupyter-lab --no-browser --port=8887


In the output, a URL will be printed. For example mine was http://localhost:8887/lab?token=376b4874f0c8e2398cfb0badcc6a941f5d95403502b3fd17
We can copy this URL into our browser to start working in Jupyter notebooks. Under Notebook, 'click Python3' to create a new notebook to work in.

That should do it. Now let's take a look at the files we will need to do eQTL mapping at :doc:`input_files`
