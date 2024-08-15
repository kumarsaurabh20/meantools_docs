Usage
=====


Databases availability
***********************

* The MetaNetX database release ``chem_prop.tsv``, which can be downloaded from their website: https://www.metanetx.org/mnxdoc/mnxref.html

* The RetroRules database version "rr_01" in SQLite (named ``mvc.db``), which can be downloaded from their website: https://retrorules.org/dl

* The ``Retrorules-Pfam annotations``, which have been curated and can be found in the github repository.

* ``LOTUS database`` is available in the form of an sqlite database within the GitHub repository.

Database Preparation
*********************

**These steps only need to be performed once: upon downloading the RetroRules database**.

Two inputs are needed for this:

* The MetaNetX database release "chem_prop.tsv", which can be downloaded from their website: https://www.metanetx.org/mnxdoc/mnxref.html
* The RetroRules database version "rr_01" in SQLite (named mvc.db), which can be downloaded from their website: https://retrorules.org/dl

``formatDatabase.py``

This script does the following to prepare RetroRules content in a format required by MEANtools:

* The script parses the RetroRules and MetaNetX database to generate a CSV with reaction_ids and their associated mass_transitions. 

   .. note::
   Using the monoisotopic mass is recommended, with the optional flag: "--monoisotopic_mass"

* It rounds and analyses the mass_transitions generated in previous step to make it compatible with the mass transitions that will later be identified by rdkit during the virtual molecule generation process. This script also generates some plots showing the transition degeneracy (answering questions such as how many reactions can be attributed to a specific mass_transition number).

* It validates the SMILES in RetroRules reactions with that of the metabolites in MetaNetX. This is because some SMILES have small differences across the two databases. This script basically filters out these rules, leaving only those in which the molecular structures involved are identical as those described by MetaNetX. 

   .. note::
   Using the optional flag "--use_greedy_rxn" is recommended. This flag will make the validation tests ignore differences in missing or extra H atoms (valence).

* This script maps the relationships among rules in RR. Because the molecular structures in RR rules are all described in different diameters (substructures), many of these substructures are identical across several rules, or are substructures of more complex structures in other rules. This script identifies and output these relationship to optimize the speed at which rules are tested with the metabolome data. For example, this script will identify that a rule involving the substructure C-N-C does not need to be tested for a specific molecule if a rule involving the substructure C-N was already tested and was not found in the molecule.

"Base rules": are the reaction rules that describe substructures that cannot be further decomposed in smaller substructures that are also in RetroRules. Base rules represent step number one in testing the metabolome data.

"Small rules": are the reaction rules described at their smallest diameter, as found in RR.

MEANtools workflow
===================

.. image:: images/workflow.png
   :width: 600

Arrows:
~~~~~~~~
* Arrows in this workflow show where the input of which script comes from.

* Grey arrows show the simplest workflow: only using metabolome information.

* Red and blue arrows show the additional steps when using transcriptome data as well. Introducing transcriptomic data acts as a filter: reactions without associated correlated transcripts are filtered out. Because of this, when using the blue arrows (recommended), the red arrows are not necessary.

Squares:
~~~~~~~~
* Blue-colored squares are the DATABASE PREPARATION phase. These scripts parse the RetroRules database to extract the data pertinent to MEANtools. These steps only need to be performed once: upon downloading the RetroRules database.

* Green-colored squares are the OMICS DATA PREPARATION phase.

* Red-colored square is a list annotating the data from the RetroRules database and its PFAM predictions. This can be downloaded from #TODO.

* Grey-colored squares are the PREDICTION phase.


Input Description:
==================

Metabolomics data
~~~~~~~~~~~~~~~~~~

A feature table from the metabolomics data is a processed data file where rows represent individual features and columns are different variables. For example, a typical feature table has columns like m/z, retention time (RT), and abundance/area-under-the-curve of each feature across samples. 

.. image:: images/featuretable1.png
   :width: 400


MEANtools processes the m/z and abundance inforation as separate files. So keeping the feature column intact, it is required that you create two csv files one with feature ids and m/z and another with feature ids and abundance values. Name these files in such a way that you remember which file has which information. 

.. image:: images/featuretable2.png
   :width: 400

File with abundance values will be used in the correlation step, where as file with m/z values will be used in the step where LOTUS database is queried. 

``queryMassNPDB.py`` is used to query the mass/charge ratio CSV described above in a custom CSV list of molecules (and their monoisomeric mass), or LOTUS database stored as sqlite formatted database. When using a custom list of molecules, this is the format required:

``molecule_id,molecule_monoisomeric_mass,SMILES
natural_product_1,70,CCCO
natural_procut_2,180.5,CCCCCCCCNC``


.. note::
Matching mass/charge ratio data with metabolite structures requires a library of ions indicating how they affect the mass/charge ratio of a structure. This is provided within the github repository as a csv file. 

A CSV with the metabolome abundance of each metabolic feature (rows) in each sample (columns). A header must be included, ``with each sample being identically named in the transcriptome``.

``name,sample1,sample2
metabolite1,100,200
metabolite2,400,500``

A CSV with the transcriptome abundance (expression mnatrix from the transcriptome data) of each locus_tag (rows) in each sample (columns). A header must be included, with each sample being identically named in the metabolome abundance file.

``name,sample1,sample2
gene1,55,66
gene2,77,88``

The above two files are used by ``corrMultiomics.py`` to generate a list of correlated metabolite-transcripts pairs. The correlation output is directly saved in the SQLIte database in the following format with a table name suffixed with ``*_correlation``:
metabolite,gene,correlation,P

``metabolite1,gene1,0.7,0.001
metabolite2,gene1,0.6,0.0001``

A CSV with PFAM annotations of the genes in the transcriptome. This can be integrated with the rest of the data at different steps (see workflow picture and help commands of each script). The format for these annotations is as follows (note that multiple pfams for a given gene can be be separated by semicolons ;):

``gene1,p450
gene2,Transferase
gene3,SQHop_cyclase_C;SQHop_cyclase_N``

OMICS DATA PREPARATION
=======================


``queryMassLOTUS.py``

   This script will query a metabolome described in a CSV table of IDs and associated mass/charge ratios to produce a list of predicted structures to each ID. The metabolome can be queried in a custom CSV table of structures (see input descriptions section above), or a NPDB database in SQLite.

   **Input**

   LOTUS SQLite database.
   CSV of feature_id,m/z

   **Output**

   The script creates a table in the project's SQLite database.
   CSV of structure predictions for each mass_signature (Optional)


``corrMultiOmics.py``

   This script will correlate the metabolome abundances with the transcriptome abundances, and return a list of annotated and correlated pairs, according to customizable score, P-value and MAD thresholds. The script also converts the correlation scores into mutual-rank (MR) [Wisecaver et al., 2017] and by using an exponential function converts mutual ranks into edge scores. The script combinely use 4 decay rates (For details check the paper) to generate 4 networks of variable size. Users can also optionally provide their own set of decay rates or use only one decay rate. 

   **Input**
   Transcriptome (RPKM)
   Metabolome (mass_signature abundance per sample)
   
   **Output**
   The script creates multiple tables in the project's SQLite database.
   

``merge_clusters.py``




Prediction
===========

``pathMassTransitions.py``

   This script integrates the metabolome and transcriptome data with the RR and MetaNetX data. In short, this script filters the mass transitions associated with RR reactions according to the mass signatures found in the metabolome. In this manner, if the metabolome has no metabolites with a mass of a 1000, then reactions involving masses of a 1000 are filtered out.

   **Optional arguments**

      --ghost: Adds "ghost" mass signatures; these are metabolites that cannot be measured in the metabolome. Each ghost mass signature is linked to at least two other metabolites that can be measured.
      --corr_cutoff and --corr_p_cutoff: to filter the correlation input through custom thresholds.
      --pfam_RR_annotation_dataset: to filter the associations between pfams and RR reactions (which are often predictions).

python pathMassTransitions_mod.py -c Jeon_tomato/Falcarindiol_Reaction_Rules/falcarindiol_cluster.csv -t test_db/format_database/MassTransitions.csv -dn /Users/singh018/Documents/Meantools_v1/Jeon_tomato/Jeon_bac.sqlite -tn transitions_falcarindiol -ct Jeon_bac_correlations -mt Jeon_metabolites_lotus -p pfam_RR_annotation_file.csv -a /Users/singh018/Documents/Meantools_v1/Jeon_tomato/Bacterial/bacterial.tomato.pfams.sol.csv -s loose -cc 0.1 -cpc 1 -v

   **Input**

      * Cluster file from the correlation step
      * Mass transition file (From the database preparation step)
      * Name of the project's sqlite database
      * Name of the correlation table
      * Name of the metabolite annotation table (from the queryMassLOTUS.py)
      * PFAM annotation file (csv)
      * PFAM-retroRules file (csv)

   **Output**

      The script creates multiple tables in the project's SQLite database.





.. _installation:

Installation
------------

To use MEANtools, first install it using pip:

.. code-block:: console


Workflow
---------

.. image:: images/workflow.png
   :width: 600

