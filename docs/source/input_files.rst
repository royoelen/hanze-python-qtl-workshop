Input files
=====

.. _download:
------------

The data files can be downloaded `here <https://drive.google.com/drive/u/1/folders/1eU1RI9GjH9IQBGPWFMGW_IBcvKado4rH>`_
Below you can find a description of what each file represents


.. _expression:

Expression file
------------

The expression file is geuvadis_normalised_gene_expression_adjusted27082018.txt. It is a large file that contains the expression of all genes for all participans that were included in the study. It is a gene by sample matrix. As you can see, each gene-sample combination is a number. This number represents the (relative) expression of that gene for that donor. 
We have subsets of this data for chromosome 6 and 22, which are smaller and easier to work with when testing.

.. _expression:

Expression annotations
------------

The genes we have measured the expression for, are located in specific positions in the genome. The file gene_locations.txt tells us where the genes are positioned on the genome.


.. _genotype:

Genotype file
------------

The genotype file is CeD_genotypes_adjusted27082018.txt. It is again a large file that has the genotypes of variant for all the participants. Here the genotypes are represented as numbers. Think of the 0 as homozygous wild-type alleles, 1 as heterozygous, and 2 as homozygous mutant alleles. 
We have subsets of this data for chromosome 6 and 22, which are smaller and easier to work with when testing.


.. _geno_annotation:

Genotype annotations
------------

These genetic variants of course have a position on the genome. The file snp_locations_CeD_adjusted27082018.txt shows for each variant in the genotype file, where it is located on the genome. This is required to decide on which effects are 'cis' and which are 'trans'
We have subsets of this data for chromosome 6 and 22, which are smaller and easier to work with when testing


Now that you have taken a look at the files, let us continue onto the actual eQTL mapping at :doc:`qtl_mapping`