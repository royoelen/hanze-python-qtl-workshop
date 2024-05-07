visualize eQTLs
=====

.. _extra_libraries:

Plotting libraries
------------

We'll need some extra libraries for making plots. In these examples we'll use seaborn. We can install these using pip

.. code-block:: console

   (pqtl_env) $ pip install seaborn


.. _jupyter_again:

Jupyter again
------------

Let's now start Jupyter again so we can make and view our plots

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

    # read the genotype data
    genotypes = pandas.read_table('/Users/royoelen/hanze-master/2021/CeD_genotypes_adjusted27082018.txt', sep='\t', index_col = 0)
    # read the expression data
    expression = pandas.read_table('/Users/royoelen/hanze-master/2021/geuvadis_normalised_gene_expression_adjusted27082018.txt', sep='\t', index_col = 0)

Based on the indices, we can extract the variant and the gene we are interested in from these two dataframes. These can then again be merged based on the donors (the column names)

.. code-block:: python

    # extract the gene
    expression_gene = expression.loc['ENSG00000179344']
    # extract the genotype
    genotype_variant = genotypes.loc['rs9273493']
    # combine into one dataframe
    variant_to_gene = pandas.merge(genotype_variant, expression_gene, right_index = True, left_index = True)


Finally, we could plot those using seaborn

.. code-block:: python

    # set the size of the plot
    f, ax = plt.subplots(figsize=(7, 6))

    # now create the plot
    seaborn.boxplot(x="rs9273493", y="ENSG00000179344", data=variant_to_gene, whis=[0, 100], width=.6, palette="vlag")

    # Add in points to show each observation
    seaborn.stripplot(x="rs9273493", y="ENSG00000179344", data=variant_to_gene, size=4, color=".3", linewidth=0)

    # Tweak the visual presentation
    ax.xaxis.grid(True)
    ax.set(ylabel="")
    seaborn.despine(trim=True, left=True)


That should be a snp-gene combination that looks like it is an actual effect. Now on your own, try to plot some more possible QTLs from the two output files you generated earlier.

Now let us move onto the last part, where you try to solve some problems on your own here: :doc:`assignments`