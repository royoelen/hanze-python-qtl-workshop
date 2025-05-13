visualize eQTLs
=====

.. _extra_libraries:

Plotting libraries
------------

We'll need some extra libraries for making plots. In these examples we'll use seaborn. We can install these using pip if we don't this yet.

.. code-block:: console

   (pqtl_env) $ pip install seaborn


.. _jupyter_again:

Jupyter again
------------

Let's now start Jupyter again so we can make and view our plots, or use Google Colab.

.. code-block:: console

   (pqtl_env) $ jupyter-lab --no-browser --port=8887 # or some other port


.. _notebook_start:

Notebook start
------------

Let us help you get on your way with regards to plotting some QTLs. First we need to import some libraries:

.. code-block:: python

    import numpy as np
    import seaborn as seaborn
    import matplotlib.pyplot as plt
    import pandas as pandas


We also need to read the expression data. For now we can use Pandas. This might however not always be the best solution, as Pandas is not the most efficient method.

.. code-block:: python

    # the locations of our files
    genotypes_loc = 'CeD_genotypes_adjusted27082018_chr22.txt'
    expression_loc = 'geuvadis_normalised_gene_expression_adjusted27082018_chr22.txt'
    # and let's load these
    genotypes = pd.read_csv(genotypes_loc, sep = '\t', header = 0, index_col = 0)
    expression = pd.read_csv(expression_loc, sep = '\t', header = 0, index_col = 0)


Again, make sure we have data in both modalities.

.. code-block:: python

    # get which samples are in both
    samples_both = genotypes.columns.intersection(expression.columns)
    # and subset in that order for both tables
    genotypes_aligned = genotypes[samples_both]
    expression_aligned = expression[samples_both]


Based on the indices, we can extract the variant and the gene we are interested in from these two dataframes. These can then again be merged based on the donors (the column names).

.. code-block:: python

    # extract the gene
    expression_gene = expression_aligned.loc['ENSG00000161180']
    # extract the genotype
    genotype_variant = genotypes_aligned.loc['rs7444']
    # combine into one dataframe
    variant_to_gene = pd.merge(genotype_variant, expression_gene, right_index = True, left_index = True)



Finally, we could plot those using seaborn.

.. code-block:: python

    # set the size of the plot
    f, ax = plt.subplots(figsize=(7, 6))

    # now create the plot
    sbs.boxplot(x="rs7444", y="ENSG00000161180", data=variant_to_gene,
        whis=[0, 100], width=.6, palette="vlag")

    # Add in points to show each observation
    sbs.stripplot(x="rs7444", y="ENSG00000161180", data=variant_to_gene,
        size=4, color=".3", linewidth=0)

    # Tweak the visual presentation
    ax.xaxis.grid(True)
    ax.set(ylabel="expression of ENSG00000161180")
    ax.set(xlabel="rs7444 genotype")
    sbs.despine(trim=True, left=True)



That should be a snp-gene combination that looks like it is an actual effect. Now on your own, try to plot some more possible QTLs from output you generated earlier.

Now let us move onto the last part, where you try to solve some problems on your own here: :doc:`assignments`