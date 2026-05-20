taf-picard 3.4.0-r1

TAFFISH wrapper for Picard 3.4.0, the Broad Institute Java toolkit for
manipulating HTS formats such as FASTQ, SAM, BAM, CRAM, and VCF.

Usage:
  taf-picard --help
  taf-picard --version
  taf-picard -- -h
  taf-picard picard -h
  taf-picard picard FastqToSam F1=reads.fq O=reads.bam SM=sample

Wrapper options:
  --help        Show this TAFFISH help text.
  --version     Show the TAFFISH package version.
  --compile     Print the generated wrapper script.
  --            Stop parsing TAFFISH options and pass the rest to the
                default upstream picard command.

Default upstream command:
  The default command is picard, a small launcher around the official
  upstream picard.jar. Picard's top-level program list uses -h:

    taf-picard -- -h
    taf-picard picard -h

  Picard's top-level --version is not a valid upstream command. Check a
  concrete tool version instead:

    taf-picard picard FastqToSam --version

Command mode:
  Command mode is enabled. Because Picard tool names such as FastqToSam and
  MarkDuplicates are arguments to the picard launcher, prefer:

    taf-picard picard FastqToSam ...
    taf-picard picard MarkDuplicates ...

  Do not use taf-picard FastqToSam ...; TAFFISH would treat FastqToSam as an
  executable inside the container.

Common workflows:
  Create an unmapped BAM from FASTQ:
    taf-picard picard FastqToSam F1=reads.fq O=reads.bam SM=sample RG=rg1

  Convert BAM back to FASTQ:
    taf-picard picard SamToFastq I=reads.bam FASTQ=reads.fq

  Create a reference sequence dictionary:
    taf-picard picard CreateSequenceDictionary R=ref.fa O=ref.dict

  Sort a SAM/BAM/CRAM file:
    taf-picard picard SortSam I=in.bam O=sorted.bam SORT_ORDER=coordinate

  Mark duplicates:
    taf-picard picard MarkDuplicates I=sorted.bam O=marked.bam M=metrics.txt

  Collect quality metrics with a chart:
    taf-picard picard MeanQualityByCycle I=reads.bam O=quality.txt CHART_OUTPUT=quality.pdf

Packaged commands:
  picard       Official Picard 3.4.0 jar launcher.
  java         OpenJDK 17 runtime required by Picard 3.x.
  Rscript      R runtime used by Picard chart/metrics tools.

Runtime notes:
  Set PICARD_JAVA_XMX to control Java heap size. The default is 2g.

    PICARD_JAVA_XMX=8g taf-picard picard MarkDuplicates I=in.bam O=out.bam M=metrics.txt

  Use Picard's own TMP_DIR=path argument for large sorts and duplicate marking.
  Use JAVA_TOOL_OPTIONS for advanced JVM flags when needed.

Platform:
  The image is built for native linux/amd64 and linux/arm64. Picard is a Java
  jar; some bundled Intel GKL native acceleration libraries may fall back to
  the Java implementation on unsupported host architectures.

Boundaries:
  This app packages the standard upstream picard.jar, Java 17, and R/Rscript.
  It does not package GATK, samtools, bwa, reference genomes, interval lists,
  chain files, Illumina run folders, cloudJar/picardcloud.jar, or Google Cloud
  provider configuration. Tools that require those inputs or services still
  need user-supplied local files or a separate upstream environment.
