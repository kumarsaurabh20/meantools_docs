Tutorial
=============
   
The test data for this tutorial is available in the repository under the folder data. The data is a subsampled version of the multi-omics data obtained from Jeon et al. 2020. Along with the accessory files, three omic-related files are available:

#. Metabolomics abundance data data/test_data/test.bac.abundance.csv
#. Metabolomics m/z data data/test_data/test.bac.metabolome.csv
#. Transcriptomic expression matrix data/test_data/test.bac.rnaseq.rpkm.csv

The RetroRules database has already been formatted and the resulting files are available here:

**Step 1** Query the LOTUS database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::
	
	The adducts file is available in the github repository data/ESI-MS-adducts.csv

	``queryMassNPDB_mod.py -add docs/ESI-MS-adducts.csv -ms Jeon_tomato/test_data/test.bac.metabolome.csv -db lotus -dbp demo.sqlite -dtn Jeon_dummy_falcarindiol -t Solanum -dn test.sqlite -tn test_metabolites -c 20 -p 20 -v True``


**Step 2** Correlation analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	.. note::

		Make sure to keep the same column names in the transcriptomics and metabolomics CSV files


	``corrMultiomics_mod.py -ft Jeon_tomato/test_data/test.bac.abundance.csv -qm Jeon_tomato/test_data/test.bac.rnaseq.rpkm.csv -mr -cl -mad -r -c 0.1 -w 0.01 -mdr 5 10 25 50 -t 4 -dn test.sqlite -tn bacterial``


**Step 3** Merge Functional Clusters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Once you finish the correlation step, analyze the functional clusters from each network with different decay rates. Using this script you can annotate functional clusters and investigate if some known patwhays genes and metabolites are clustering together. Some accessory scripts are available in the github repository to generate network graphs from the functional clusters.

	``merge_clusters.py -ft Jeon_tomato/test_data/test.bac.abundance.csv -qm Jeon_tomato/test_data/test.bac.rnaseq.rpkm.csv -a -f Jeon_tomato/annotation/tomato.new.pfams_description.csv -mc -mm overlap -dr 25 -dn test.sqlite``


**Step 4** Map mass transitions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	``pathMassTransitions_mod.py -c Jeon_tomato/test_data/test_merged_cluster_filtered.csv -t test_db/format_database/MassTransitions.csv -dn test.sqlite -tn transitions_test_falca -ct bacterial -mt test_metabolites -p pfam_RR_annotation_file.csv -a Jeon_tomato/Bacterial/bacterial.tomato.pfams.sol.csv -s loose -cc 0.1 -cpc 1 -v``


**Step 5** Predict reaction steps
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	``heraldPathways_mod.py -c Jeon_tomato/test_data/test_merged_cluster_filtered.csv -r test_db/format_database/ValidateRulesWithOrigins.csv -m test_db/format_database/base_rules.csv -p pfam_RR_annotation_file.csv -a Jeon_tomato/Bacterial/bacterial.tomato.pfams.sol.csv -s loose -i 3 -dn test.sqlite -tn test_herald -ct bacterial -mt test_metabolites -tt transitions_test_falca -v -dv -o test_herald -d pfams_dict.csv``

**Step 6** Generate reaction graphics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	``paveWays.py -sp test_herald/structure_predictions.csv -of test_paveWays -r test_herald/reactions.csv -praf pfam_RR_annotation_file.csv -gaf Jeon_tomato/Bacterial/bacterial.tomato.pfams.sol.csv -rr loose -pam True -pup True -v``
