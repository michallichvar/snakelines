Configuration of pipeline
=========================

Supported pipelines are stored at the pipeline/ directory of SnakeLines scripts.
Pipelines typically contains a master file (Snakefile), several sub-workflows (Snakefile.<sub-workflow>) and a configuration file (config.yaml).
The master file and sub-workflows define, what files would be generated.
Rules, that define, how they would be generated are stored in the rules/ directory.
These rules may be easily configured adjusting the config.yaml configuration of the pipeline.

Configurations typically consists of several steps.
At first, define set of samples and reference genomes to analyse.
Next, define where to store summary reports of the analysis.
Finally, adjust parameters of rules in the pipeline.
See `example <#example-configuration>`_ at the end of this chapter.


Define set of samples
---------------------

Fastq files stored in the reads/original directory can be analysed together.
To analyse only a subset of samples configuration must start with `samples:` attribute.
The attribute may contain several group categories, each with its own reference and target panel set.
For example, you may specify to all sample with name started with `one` to be analysed against hg19 genome, one panel.
For `sureselect6` samples you may use different genome and panel.

.. code-block:: yaml

   samples:                  # List of sample categories to be analysed
      # Category with one panel
      - name: one.*          # Regex expression of sample names to be analysed (reads/original/one.*_R1.fastq.gz)
        reference: hg19      # Reference genome for reads in the category (reference/hg19/hg19.fa)
        panel: one           # Bed file with target region coordinates (reference/hg19/annotation/one/regions.bed)

      # Another category with sureselect6 panel
      - name: sureselect6.*  # Regex expression of sample names to be analysed (reads/original/sureselect6.*_R1.fastq.gz)
        reference: hg38      # Reference genome for reads in the category (reference/hg19/hg19.fa)
        panel: sureselect6   # Bed file with target region coordinates (reference/hg19/annotation/sureselect6/regions.bed)


Report directory
----------------

Using Snakemake generally leads to a wide hierarchy of intermediate directories and files.
Typically only few final files and reports are used for interpretation.
SnakeLines therefore copies relevant files to the separate, `report` directory in the last step of a pipeline.
User may define, where to put output reports.

.. code-block:: yaml

   report_dir: report/public/01-exome  # Generated reports and essential output files would be stored there


Adjust rules parameters
-----------------------

Each step of the analysis have its own configuration, that would be used to parametrize designated rule.
The first argument is a method, that may be easily swap to another supported one by changing configuration.
The rest is the set of parameters that would be passed to designated rule.


.. code-block:: yaml

   reads:                           # Prepare reads and quality reports for downstream analysis
      preprocess:                   # Pre-process of reads, eliminate sequencing artifacts, contamination ...
         trimmed:                   # Remove low quality parts of reads
            method: trimmomatic     # Supported values: trimmomatic
            temporary: False        # If True, generated files would be removed after successful analysis
            crop: 500               # Maximal number of bases in read to keep. Longer reads would be truncated.
            quality: 20             # Minimal average quality of read bases to keep (inside sliding window of length 5)
            headcrop: 20            # Number of bases to remove from the start of read
            minlen: 35              # Minimal length of trimmed read. Shorter reads would be removed.

This example snippet would be roughly translated to


.. code-block:: python

   method_config = {'temporary': False,
                    'crop': 500,
                    'quality': 20,
                    'headcrop': 20,
                    'minlen': 35}

   # Configuration would be passed to the included, trimmomatic rule
   include rules/reads/preprocess/trimmed/trimmomatic.snake

Skipping preprocess steps
-------------------------

Parts of preprocess and postprocess steps may be easily skipped by removal of corresponding configuration parts.
This snipped would at first trim reads, then decontaminate them, and finally deduplicate them.

.. code-block:: yaml

   reads:                              # Prepare reads and quality reports for downstream analysis
       preprocess:                     # Pre-process of reads, eliminate sequencing artifacts, contamination ...

           trimmed:                    # Remove low quality parts of reads
               method: trimmomatic     # Supported values: trimmomatic
               temporary: False        # If True, generated files would be removed after successful analysis
               crop: 500               # Maximal number of bases in read to keep. Longer reads would be truncated.
               quality: 20             # Minimal average quality of read bases to keep (inside sliding window of length 5)
               headcrop: 20            # Number of bases to remove from the start of read
               minlen: 35              # Minimal length of trimmed read. Shorter reads would be removed.

           decontaminated:             # Eliminate fragments from known artificial source, e.g. contamination by human
               method: bowtie2         # Supported values: bowtie2
               temporary: False        # If True, generated files would be removed after successful analysis
               references:             # List of reference genomes
                   - mhv
               keep: True              # Keep reads mapped to references (True) of remove them as contamination (False)

           deduplicated:               # Remove fragments with the same sequence (PCR duplicated)
               method: fastuniq        # Supported values: fastuniq
               temporary: False        # If True, generated files would be removed after successful analysis


Omitting configuration for the decontamination step would restrict read preprocessing to the trimming and the deduplication step, only.

.. code-block:: yaml

   reads:                              # Prepare reads and quality reports for downstream analysis
       preprocess:                     # Pre-process of reads, eliminate sequencing artifacts, contamination ...

           trimmed:                    # Remove low quality parts of reads
               method: trimmomatic     # Supported values: trimmomatic
               temporary: False        # If True, generated files would be removed after successful analysis
               crop: 500               # Maximal number of bases in read to keep. Longer reads would be truncated.
               quality: 20             # Minimal average quality of read bases to keep (inside sliding window of length 5)
               headcrop: 20            # Number of bases to remove from the start of read
               minlen: 35              # Minimal length of trimmed read. Shorter reads would be removed.

           deduplicated:               # Remove fragments with the same sequence (PCR duplicated)
               method: fastuniq        # Supported values: fastuniq
               temporary: False        # If True, generated files would be removed after successful analysis



Example configuration
---------------------

Example for the read mapping pipeline.

.. literalinclude:: ../_static/configuration/mapping.yaml
   :language: yaml