Installation
=============

To use MEANtools, prepare the meantools repository:

.. code-block:: bash
   :linenos:

   conda create -n meantools
   conda activate meantools
   export SKLEARN_ALLOW_DEPRECATED_SKLEARN_PACKAGE_INSTALL=True
   export LC_ALL=en_US.UTF-8
   export LANG=en_US.UTF-8
   conda env update -f environment.yml
   git clone https://github.com/kumarsaurabh20/meantools.git
   cd meantools

.. note::
   Download additional files from https://doi.org/10.34894/2MVBGK and move them into the directory where you cloned meantools.
