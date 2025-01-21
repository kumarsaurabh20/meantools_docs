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

.. warning::
   MEANtools is tested on Pandas=1.4.3. From version >=2.0.0, the support for *append* function has been deprecated for dataframes. We will fix this issue in the upcoming version. But for the current version, please install Pandas -v1.4.3 or <2.0.0

.. note::
   Download additional files from https://doi.org/10.5281/zenodo.14651195 and move them into the directory where you cloned meantools. Additional files include:
      #. SQLite formatted LOTUS database
      #. RetroRules cross-referenced data. Mentioned in the paper as *loose*, *medium*, and *strict* dataset.
      #. EC-PFAM mapping file.
