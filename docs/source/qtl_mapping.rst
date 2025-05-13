QTL mapping
=====


.. _jupyter:

Jupyter
------------

Let's now start Jupyter  so we can make and view our plots, or use Google Colab

.. code-block:: console

   (pqtl_env) $ jupyter-lab --no-browser --port=8887 # or some other port


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


Let us see if we found anything

.. code-block:: python

   results_nowindow[results_nowindow['p_value'] < 0.05]


We have some significant associations, though what we did is not exactly fair. We did a lot of tests, so we should ideally correct for that.

.. code-block:: python

   # get a bonferroni one
   _, p_bonferroni, _, _ = multipletests(results_nowindow['p_value'], method='bonferroni')
   results_nowindow['p_bonferroni'] = p_bonferroni

   # and BH
   _, p_bh, _, _ = multipletests(results_nowindow['p_value'], method='fdr_bh')
   results_nowindow['p_bh'] = p_bh


Let us see how many are left with Bonferroni or BH

.. code-block:: python

   results_nowindow[results_nowindow['p_bonferroni'] < 0.05]
   results_nowindow[results_nowindow['p_bh'] < 0.05]


Previous studies have mapped eQTLs as well, so let us see what they found

.. code-block:: python

   # the location of this data
   eqtlgen_confinement_loc = 'eqtlgen_fdr005_chr22.tsv.gz'
   # read this data
   eqtlgen_confinement = pd.read_csv(eqtlgen_confinement_loc, sep = '\t', header = 0)
   # give it column names we like
   # subset to just variant and feature
   eqtlgen_confinement = eqtlgen_confinement[['SNP', 'Gene']]
   # rename the columns to be the same as our output
   eqtlgen_confinement = eqtlgen_confinement.rename({'SNP' : 'variant', 'Gene' : 'feature'}, axis = 1)
   eqtlgen_confinement


If we just wanted to replicate these findings, we would only test these, and only correct the number of tests for these:

.. code-block:: python

   # we already tested everything, so we can subset our results to what is in our confinement
   results_eqtlgen = results_nowindow.merge(eqtlgen_confinement, on=['variant', 'feature'])
   # if we would have only tried to replicate, we'd only have to correct for those
   _, p_bonferroni_eqtlgen, _, _ = multipletests(results_eqtlgen['p_value'], method='bonferroni')
   results_eqtlgen['p_bonferroni'] = p_bonferroni_eqtlgen
   _, p_bh_eqtlgen, _, _ = multipletests(results_eqtlgen['p_value'], method='fdr_bh')
   results_eqtlgen['p_bh'] = p_bh_eqtlgen
   # let's see how many we would replicate then
   results_eqtlgen
   # which seems it is all of them (for this chromosome)


Q: Can you think of a reason why subsetting the data post-hoc is not the greatest idea?
Q: What is the major limitation of trying to replicate?

To also reduce our multiple testing burden, we can also only test variants that are close to genes. We need that information though. Let us add this to the results for now.

They look like this

.. code-block:: python

   variant_positions


.. code-block:: python

   gene_positions


And adding it

.. code-block:: python

   # let's rename the columns to make them both unique and matching where needed
   variant_positions = variant_positions.rename({'snpid' : 'variant', 'chr' : 'var_chr', 'pos' : 'var_pos'}, axis = 1)
   gene_positions = gene_positions.rename({'geneid' : 'feature', 'chr' : 'feature_chr', 'left' : 'feature_start', 'right' : 'feature_end'}, axis = 1)

   # and add them to the output
   results_nowindow = results_nowindow.merge(variant_positions, how = 'left', on = 'variant')
   results_nowindow = results_nowindow.merge(gene_positions, how = 'left', on = 'feature')

   # and let's see what they look like
   results_nowindow


We can add the distances between variant and gene

.. code-block:: python

   # we can also test less, by looking at variants close to genes. let's see how far the variants are from the flanks of the genes
   results_nowindow['variant_to_start'] = results_nowindow['feature_start'] - results_nowindow['var_pos']
   results_nowindow['variant_to_end'] = results_nowindow['feature_end'] - results_nowindow['var_pos']
   results_nowindow


and subset the data to variants close to genes at 50k


.. code-block:: python

   # if we confine ourselves to variants at most 50k away from the gene, we should have lest tests
   results_50kwindow = results_nowindow[((results_nowindow['variant_to_start'] > -50000) & (results_nowindow['variant_to_start'] < 50000)) | ((results_nowindow['variant_to_end'] > -50000) & (results_nowindow['variant_to_end'] < 50000))].copy()
   # if we would have only tried in this window, we'd only have to correct for those
   _, p_bonferroni_50k, _, _ = multipletests(results_50kwindow['p_value'], method='bonferroni')
   results_50kwindow['p_bonferroni'] = p_bonferroni_50k
   _, p_bh_50k, _, _ = multipletests(results_50kwindow['p_value'], method='fdr_bh')
   results_50kwindow['p_bh'] = p_bh_50k
   # so let's see what we have
   results_50kwindow

Q: what is the limitation of doint this?

Let us next visualize eQTLs at :doc:`qtl_visualization`