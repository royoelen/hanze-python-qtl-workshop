QTL mapping
=====


.. _libraries:

Libraries
---------

QTLs are statistical associations. To do these, we'll need some libraries. We also need to load data and will visualize in the end

.. code-block:: python

   import pandas as pd
   import seaborn as sbs
   import matplotlib.pyplot as plt
   from scipy.stats import spearmanr
   from statsmodels.stats.multitest import multipletests



.. _files_colab:

Files colab
-----------

We can use Google Colab if we don't want to set up a Python environment ourselves. We will however need to upload the files there as well

.. code-block:: python

   # we need to upload files if we are using colab
   from google.colab import files
   uploaded = files.upload()



.. _file_loading:

Data loading
------------

We have the files downloaded and we have the QTL mapping tool downloaded. Let us store the locations of these files into variables, so that we can reference them more easily.
Let us also store the result file in a variable

Let us set up the paths and load some data

.. code-block:: python

   # the locations of our files
   genotypes_loc = 'CeD_genotypes_adjusted27082018_chr22.txt'
   expression_loc = 'geuvadis_normalised_gene_expression_adjusted27082018_chr22.txt'
   variant_positions_loc = 'snp_locations_CeD_adjusted27082018_chr22.tsv.gz'
   gene_positions_loc = 'gene_locations.txt'
   # and let's load these
   genotypes = pd.read_csv(genotypes_loc, sep = '\t', header = 0, index_col = 0)
   expression = pd.read_csv(expression_loc, sep = '\t', header = 0, index_col = 0)
   variant_positions = pd.read_csv(variant_positions_loc, sep = '\t', header = 0)
   gene_positions = pd.read_csv(gene_positions_loc, sep = '\t', header = 0)


When we associate the expression and genotype information, we need measurements in both modalities. Let us make sure that we have that

.. code-block:: python

   # get which samples are in both
   samples_both = genotypes.columns.intersection(expression.columns)
   # and subset in that order for both tables
   genotypes_aligned = genotypes[samples_both]
   expression_aligned = expression[samples_both]



.. _first_run:

First run
------------

Let us do a first run now, we will just associate the first variant to the first gene

.. code-block:: python

   # extract first gene first variant, the first row of each dataframe
   first_variant = genotypes_aligned.iloc[0]
   first_gene = expression_aligned.iloc[0]
   # calculate the correlation and accompanying p value
   first_correlation, first_p_value = spearmanr(first_variant, first_gene)
   # print the result
   print(''.join(['correlation of first gene to first variant:', str(first_correlation)]))
   print(''.join(['p of first gene to first variant:', str(first_p_value)]))


That p-value is not going to impress anyone. Let's just try all combinations to see what sticks

.. code-block:: python

   # we'll save the results in a list first
   results = []

   # check each variant
   for variant in genotypes_aligned.index:
      # check each gene
      for feature in expression_aligned.index:
         # extract genotype and expression
         row_genotype = genotypes_aligned.loc[variant]
         row_expression = expression_aligned.loc[feature]
         # calculate stats
         correlation, p_value = spearmanr(row_genotype, row_expression)
         # put in a list
         results.append([variant, feature, correlation, p_value])

   # put it all together
   results_nowindow = pd.DataFrame(results, columns=['variant', 'feature', 'correlation', 'p_value'])


.. code-block:: console

   (pqtl_env) $ python pyqtl_mapper.py --snp_file_location %snp_loc% --probe_file_location %probe_loc% --snp_positions_file_location %snp_anno% --probe_positions_file_location %probe_anno% --use_model linear --output_location %result_loc% --cis_distance 10000 --cis True


This could take a little while to run. If you are having some problems getting everything to run, or if your laptop is having a particularly hard time doing the computations, the results are also in the unconfined_mapping.tsv file `here <https://drive.google.com/drive/u/1/folders/1eU1RI9GjH9IQBGPWFMGW_IBcvKado4rH>`_

There are some parameters that we glossed over. One is the cis distance. The cis distance defines what we think is 'close' to a gene (cis) and what we think is 'far' from a gene 'trans'. Another parameter is the cis parameter. This one tells the software to look at the cis or trans effects specifically. Here we chose cis. This combination of parameters means that we are looking at genetic effects within 10000 basepairs of each gene.

If you take a look at the results, you see that we have only one significant result. You can also see that a lot of variants are tested for the same gene. This is of course causing us having to deal with a very large multiple testing burden. Let us hypothesize that we have an idea of which variants mighf affect gene expression due to previous studies, and want to test only those. Selecting which snp-gene combinations to test a priori is called using a confinement. Let us use define and use a confinement file. We will also write to a new result file.

Bash:

.. code-block:: console

   (pqtl_env) $ conf_loc='/Users/royoelen/hanze-master/2021/eqtls_eqtlgen_confinement.tsv'
   (pqtl_env) $ result2_loc='/Users/royoelen/hanze-master/2021/pyqtl_results_eqtlgen.tsv'


Windows Anaconda prompt:

.. code-block:: console

   (pqtl_env) $ conda env config vars set conf_loc = 'C:\Users\royoelen\hanze-master\2021\eqtls_eqtlgen_confinement.tsv'
   (pqtl_env) $ conda env config vars set result2_loc = 'C:\Users\royoelen\hanze-master\2021\pyqtl_results_eqtlgen.tsv'
   (pqtl_env) $ conda activate pyqtl_env


And add that confinement to our parameters

Bash:

.. code-block:: console

   (pqtl_env) $ python pyqtl_mapper.py --snp_file_location ${snp_loc} --probe_file_location ${probe_loc} --snp_positions_file_location ${snp_anno} --probe_positions_file_location ${probe_anno} --use_model linear --output_location ${result2_loc} --cis_distance 0 --cis True --confinements_snp_probe_pairs_location ${conf_loc}


Windows Anaconda prompt:

.. code-block:: console

   (pqtl_env) $ python pyqtl_mapper.py --snp_file_location %snp_loc% --probe_file_location %probe_loc% --snp_positions_file_location %snp_anno% --probe_positions_file_location %probe_anno% --use_model linear --output_location %result2_loc% --cis_distance 0 --cis True --confinements_snp_probe_pairs_location %conf_loc%


That should be quite a bit faster. Again, the results are also already available if you run into any computational issues in the confined_mapping.tsv file `here <https://drive.google.com/drive/u/1/folders/1eU1RI9GjH9IQBGPWFMGW_IBcvKado4rH>`_

When you look at these results, you should see that we have less entries that were tested, but they are mostly significant, owing to our decreased multiple testing burden.


Using a confinement can thus be very helpfull, but you will never find any new effects. Let us visualize eQTLs at :doc:`qtl_visualization`