.. title: Machine learning with Tensorflow: an introduction
.. slug:
.. date: 2020-11-03 19:36:53 UTC+01:00
.. tags: 
.. category: 
.. link: 
.. description: Material for the CISM/CÉCI training session on 10/11/2020, see https://indico.cism.ucl.ac.be/event/84/
.. type: text


*This page collects the material for for the CISM/CÉCI training session on 10/11/2020,
see the* `indico page`_ *for more details.*

Table of contents
-----------------

- `A brief introduction to machine learning <mlintro.html>`_
- `A more technical introduction to Tensorflow <tfprimer.html>`_
- `Other building blocks and architectures <nnarchitectureexamples.html>`_
- An exercise with a slightly more interesting problem

To make it more interactive
---------------------------

While not absolutely essential, it is nice if you can try some things yourself
(time permitting, there are also some exercises), so if possible you should 
try to set up an environment where you can play a bit with Tensorflow_ (ideally
before the training, to save a bit of time).

On manneback
''''''''''''

The simplest way (since you are assumed to have an account,
and to be familiar with using SSH and the command line) if you have a stable
internet connection is to open a terminal and launch an interactive job with slurm on the
`Manneback <https://www.cism.ucl.ac.be/doc/_contents/Computing/index.html#id1>`_ cluster.

First connect to the interactive node:

.. code-block:: sh

   ssh gwceci.cism.ucl.ac.be # or gwcism.cism.ucl.ac.be
   ssh manneback.cism.ucl.ac.be


We will in any case launch an interactive job on the worker (such that we
don't all compete for resources on the same machine), but there are two options:
the most convenient is to launch a Jupyter_ kernel, such that you can edit the
notebook (and view some figures) from the browser on your personal computer.

This needs a tunnel to manneback, which can be set up with sshuttle_ following
the `CISM instructions <https://www.cism.ucl.ac.be/doc/_contents/Other/index.html#jupyter>`_,
and changing the ``srun`` command to

.. code-block:: sh

   srun --pty bash -c 'source /cvmfs/sft.cern.ch/lcg/views/LCG_98python3/x86_64-centos7-gcc9-opt/setup.sh; jupyter notebook --ip $(hostname -i) --no-browser'

which will run

.. code-block:: sh

   source /cvmfs/sft.cern.ch/lcg/views/LCG_98python3/x86_64-centos7-gcc9-opt/setup.sh
   jupyter notebook --ip $(hostname -i) --no-browser

in an interactive slurm job: source an environment setup script, and launch
a Jupyter_ kernel..
Slurm may take a few seconds to assign you a worker node, then the kernel
should print a few lines.
At the end, there should be "Or copy and paste one of these URLs:",
with at least one URL that starts with ``http://10....``.
Thanks to the tunnel you can open this in a web browser on your laptop/desktop
(which is a better idea than running a web browser with X11 forwarding).

To check that everything is working correctly, you can create a test notebook
somewhere in your home directory, and try the following to check that
Tensorflow_ is correctly set up:

.. code-block:: python

   import tensorflow as tf
   print(tf.__version_)

The main advantage of the notebook over an IPython_ session is that this gives
an inline figure:

.. code-block:: python
   
   import numpy as np
   from matplotlib import pyplot as plt
   x = np.linspace(0., 5., 101)
   plt.plot(x, np.sin(x))


Using binder_
'''''''''''''

.. image:: https://mybinder.org/badge_logo.svg
   :target: https://mybinder.org/v2/gh/pieterdavid/cism-mltf2020-docker/main

The badge above will launch a binder_ session with an environment prepared
for the exercises.
Resources will be limited, so some of the larger networks may not finish
training.

Using Colaboratory_
'''''''''''''''''''

If you have a Google account, you can use Colaboratory_ to interactively edit
and run notebooks.
The environment is meant for machine learning, so no additional setup is
required (resources are not guaranteed, but likely sufficient for anything we
will do this afternoon).

Locally
'''''''

If your internet connection is not very stable, or you are used to running
python on your personal computer or elsewhere, you can also install Tensorflow_
in a conda_ environment, or (preferably in a `virtual environment`_) with pip,
more details can be found in the `installation instructions`_.

With conda_, all you should need is this:

.. code-block:: sh

   conda config --add channels conda-forge # if not already the case
   conda create -n mltftraining2020 tensorflow=2.3.0 tensorboard=2.3.0 ipython matplotlib ipykernel
   conda activate mltftraining2020
   ipython kernel install --user --name "mltftraining2020"

And with virtualenv and pip:

.. code-block:: sh

   python -m venv mltftraining2020 # pick a name
   source mltftraining2020/bin/activate
   pip install tensorflow tensorboard ipython matplotlib ipykernel
   ipython kernel install --user --name "mltftraining2020"

If you do not already have the Jupyter_ notebook server installed, you should
add the ``notebook`` package to the conda or pip install command.
The last line installs a kernel that you can select to run the notebook with.

The exercises assume that you are have at least version 2.1.0 of Tensorflow_,
which requires Python 3.5 or above.

You could also reuse the docker image used by binder_ above, it is available
on dockerhub as
`pieterdavid/cism-mltf2020-docker <https://hub.docker.com/r/pieterdavid/cism-mltf2020-docker>`_,
and can be pulled with

.. code-block:: sh

   docker pull pieterdavid/cism-mltf2020-docker


.. _indico page: https://indico.cism.ucl.ac.be/event/84/

.. _Tensorflow: https://www.tensorflow.org

.. _Jupyter: http://jupyter.org

.. _IPython: http://ipython.org

.. _sshuttle: https://sshuttle.readthedocs.io/en/stable/

.. _binder: https://mybinder.org

.. _Colaboratory: https://colab.research.google.com/

.. _docker: https://www.docker.com

.. _conda: https://docs.conda.io/en/latest/

.. _virtual environment: https://docs.conda.io/en/latest/

.. _installation instructions: https://www.tensorflow.org/install

.. |---| unicode:: U+2014
   :trim:

