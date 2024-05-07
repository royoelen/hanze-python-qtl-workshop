QTL mapping
=====

.. _file_locs:

File locations
------------

We have the files downloaded and we have the QTL mapping tool downloaded. Let us store the locations of these files into variables, so that we can reference them more easily.
Let us also store the result file in a variable

.. code-block:: console

   (pqtl_env) $ snp_loc='/Users/royoelen/hanze-master/2021/CeD_genotypes_adjusted27082018.txt'
   (pqtl_env) $ probe_loc='/Users/royoelen/hanze-master/2021/geuvadis_normalised_gene_expression_adjusted27082018.txt'
   (pqtl_env) $ snp_anno='/Users/royoelen/hanze-master/2021/snp_locations_CeD_adjusted27082018.txt'
   (pqtl_env) $ probe_anno='/Users/royoelen/hanze-master/2021/gene_locations.txt'
   (pqtl_env) $ result_loc='/Users/royoelen/hanze-master/2021/pyqtl_results_unconfined.tsv'


.. _first_run:

First run
------------

Let us do a first run now

.. code-block:: console

   (pqtl_env) $ python pyqtl_mapper.py --snp_file_location ${snp_loc} --probe_file_location ${probe_loc} --snp_positions_file_location ${snp_anno} --probe_positions_file_location ${probe_anno} --use_model linear --output_location ${result_loc} --cis_distance 10000 --cis True


This could take a little while to run. If you are having some problems getting everything to run, or if your laptop is having a particularly hard time doing the computations, the results are also in the unconfined_mapping.tsv.gz file `here <https://drive.google.com/drive/u/1/folders/1eU1RI9GjH9IQBGPWFMGW_IBcvKado4rH>`_

There are some parameters that we glossed over. One is the cis distance. The cis distance defines what we think is 'close' to a gene (cis) and what we think is 'far' from a gene 'trans'. Another parameter is the cis parameter. This one tells the software to look at the cis or trans effects specifically. Here we chose cis. This combination of parameters means that we are looking at genetic effects within 10000 basepairs of each gene.

If you take a look at the results, you see that we have only one significant result. You can also see that a lot of variants are tested for the same gene. This is of course causing us having to deal with a very large multiple testing burden. Let us hypothesize that we have an idea of which variants mighf affect gene expression due to previous studies, and want to test only those. Selecting which snp-gene combinations to test a priori is called using a confinement. Let us use define and use a confinement file. We will also write to a new result file.


.. code-block:: console

   (pqtl_env) $ conf_loc='/Users/royoelen/hanze-master/2021/eqtls_eqtlgen_confinement.tsv'
   (pqtl_env) $ result2_loc='/Users/royoelen/hanze-master/2021/pyqtl_results_eqtlgen.tsv'


And add that confinement to our parameters

.. code-block:: console

   (pqtl_env) $ python pyqtl_mapper.py --snp_file_location ${snp_loc} --probe_file_location ${probe_loc} --snp_positions_file_location ${snp_anno} --probe_positions_file_location ${probe_anno} --use_model linear --output_location ${result2_loc} --cis_distance 0 --cis True --confinements_snp_probe_pairs_location ${conf_loc}


That should be quite a bit faster. Again, the results are also already available if you run into any computational issues in the confined_mapping.tsv.gz file `here <https://drive.google.com/drive/u/1/folders/1eU1RI9GjH9IQBGPWFMGW_IBcvKado4rH>`_

When you look at these results, you should see that we have less entries that were tested, but they are mostly significant, owing to our decreased multiple testing burden.


Using a confinement can thus be very helpfull, but you will never find any new effects. Let us visualize eQTLs at :doc:`qtl_visualization`