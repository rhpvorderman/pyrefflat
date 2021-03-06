.. image:: https://travis-ci.org/sndrtj/pyrefflat.svg?branch=master
    :target: https://travis-ci.org/sndrtj/pyrefflat

.. image:: https://coveralls.io/repos/sndrtj/pyrefflat/badge.svg?branch=master&service=github 
    :target: https://coveralls.io/github/sndrtj/pyrefflat?branch=master
 
=========
pyrefflat
=========

Pyrefflat is a parser for refFlat files, used by UCSC to annotate genes and their exons, in pure python.
It includes a reader and writer object for easy manipulation of refFlat files.

Installation
------------

To install pyrefflat, clone the repository at  https://github.com/sndrtj/pyrefflat .
Run ``python setup.py install`` to install the module. It is recommended you use a virtual environment.

Usage
-----

Reading refFlat files
~~~~~~~~~~~~~~~~~~~~~
Pyrefflat provides a ``Reader`` object for reading refFLat files.

.. code-block:: python

    from pyrefflat import Reader
    reader = Reader(filename="file.reFlat")

A reader is an iterator that returns records. Each record has associated exons.
E.g., to print the start site of every exon in every record, one would do:

.. code-block:: python

    for record in reader:
        for exon in record.exons:
            print exon.start


Writing refFlat files
~~~~~~~~~~~~~~~~~~~~~
Writing refFlat files is done with the ``Writer`` object, which consumes ``Record`` instances.
E.g., to copy a refFlat file using a ``Reader`` and ``Writer`` one could do:

.. code-block:: python

    from pyrefflat import Reader, Writer
    reader = Reader(filename="original.refFlat")
    writer = Writer(filename="writer.refFlat")

    for record in reader:
        writer.write(record)
    writer.close()
    reader.close()



New records can be generated using the ``fromline`` or ``fromdict`` classmethods of the ``Record`` object.
The ``fromline`` classmethod takes a single refFlat line, whereas ``fromdict`` takes a dictionary.
Key names can be found be found in ``generics.COLUMNS``

Processor
~~~~~~~~~
Apart from reading lines iteratively, a ``RefFlatProcessor`` object is provided.
This processes all records in the given refFlat file, and will store their genes, transcripts and exons as attributes,
and it will calculate derived gene coordinates.
The ``process`` function will by default remove any duplicated transcripts, storing only the first it encounters.
One can disable this, but this might throw off derived gene coordinates
(e.g. some refseq transcripts are annotated on multiple chromosomes!)

The processor allows one to for instance sort records. E.g., to get a list of unique genes in the refFlat file, and write all records sorted on gene name:

.. code-block:: python

    from pyrefflat import Writer
    from pyrefflat.parser import RefFlatProcessor

    writer = Writer("/path/to/new.refFlat")
    proc = RefFlatProcessor("path/to/file.refFlat")
    proc.process

    for gene in sorted(set(proc.genes.keys())):
        for tr in proc.genes[gene].transcripts:
        writer.write(tr)

    writer.close()

Tools
-----

Apart from a parser, it includes several standalone tools. These additionally depend on the ``pysam`` and ``vcf`` modules.

gVCFCoverage
~~~~~~~~~~~~
This tool calculates coverage on the regions specified in the refFlat exon fields in a gVCF file.
It can output in three different formats, namely simple tab-delimited (csv or tsv) format, BED format or as a JSON.
It supports using the GQ field, allowing to filter only those regions with a minimum GQX value.

.. code-block::

    gVCFCoverage -h
    usage: gVCFCoverage [-h] -I INPUT -O OUTPUT -R REFFLAT -m {BP_RESOLUTION,GVCF}
                        -om {bedgraph,csv,json} [--tab-delimited] [-g] [-p]
                        [--bp-res] [-c] [--limit | --reverse-limit]
                        [--min-value [MIN_VALUE]] [--margin [MARGIN]]
                        [--intron-margin [INTRON_MARGIN]]
                        [--utr-margin [UTR_MARGIN]]

    optional arguments:
      -h, --help            show this help message and exit
      -I INPUT, --input INPUT
                            Your input gzipped gVCF
      -O OUTPUT, --output OUTPUT
                            Path to output file
      -R REFFLAT, --refflat REFFLAT
                            Path to refflat file
      -m {BP_RESOLUTION,GVCF}, --mode {BP_RESOLUTION,GVCF}
                            gVCF mode
      -om {bedgraph,csv,json}, --output-mode {bedgraph,csv,json}
                            Output file type
      --tab-delimited       Output CSV as TSV
      -g, --gqx             Output gqx values
      -p, --perc            Output perc of --min-value
      --bp-res              Output base-pair resolution
      -c, --cds-only        Consider coding regions only
      --limit               Limit output to just those records at least attaining
                            --min-value
      --reverse-limit       Limit output to just those records which do NOT attain
                            --min-value
      --min-value [MIN_VALUE]
                            minimal value, in gqx or dp, defaults to 0
      --margin [MARGIN]     General margin around refflat records. Will be
                            overriden by --utr-margin and --intron-margin If these
                            are set
      --intron-margin [INTRON_MARGIN]
                            Margin around introns of refflat records. Defaults to
                            0 (no margin)
      --utr-margin [UTR_MARGIN]
                            Margin around utrs of refflat records. Defaults to 0
                            (no margin)


createMargin
~~~~~~~~~~~~
This tool adds a margin around each exon and writes the result to a new refFlat file.

refFlat2Bed
~~~~~~~~~~~
This tool converts a refFlat file to a BED file, with the regions based on the exons.


License
-------
pyrefflat is MIT licensed.
