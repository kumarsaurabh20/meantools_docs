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
   MEANtools is tested on Python=3.8.13 and Pandas=1.4.3. From Pandas version >=2.0.0, the support for *append* function has been deprecated for dataframes. We will fix the dependency issues in the upcoming version.

.. note::
   Download additional files from https://doi.org/10.5281/zenodo.14864196 and move them into the directory where you cloned meantools. Additional files include:
      #. SQLite formatted LOTUS database
      #. RetroRules cross-referenced data. Mentioned in the paper as *loose*, *medium*, and *strict* dataset.
      #. EC-PFAM mapping file.
      #. RNA-seq normalised gene expression matrix from Jeon et al. (2020).
