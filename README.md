# taf-picard

TAFFISH wrapper for [Picard](https://github.com/broadinstitute/picard), the
Broad Institute Java command-line toolkit for manipulating high-throughput
sequencing data and formats such as FASTQ, SAM, BAM, CRAM, and VCF.

## Package Identity

| Field | Value |
| --- | --- |
| name | `picard` |
| command | `taf-picard` |
| version | `3.5.0-r1` |
| kind | `tool` |
| container | `ghcr.io/taffish/picard:3.5.0-r1` |
| upstream | Picard `3.5.0` |
| runtime version | `Version:3.5.0` from a concrete Picard tool such as `FastqToSam --version`; jar manifest also reports `Implementation-Version: 3.5.0` and `htsjdk-Version: 5.0.0` |

## What This App Provides

This app packages the official Picard `3.5.0` release jar, pinned by SHA256.
The image includes:

- `picard`, a small launcher for the official upstream `picard.jar`;
- OpenJDK 17, required by Picard 3.x;
- R and `Rscript`, needed by Picard chart and metrics tools such as
  `MeanQualityByCycle`, `QualityScoreDistribution`, and
  `CollectInsertSizeMetrics`;
- the self-contained Java dependencies bundled inside the upstream release
  jar, including HTSJDK `5.0.0`.

Picard 3.5.0 upgrades HTSJDK from 4.2.0 to 5.0.0. This adds full CRAM 3.1
write support and faster BAM/CRAM I/O, fixes flow-based and physical-location
duplicate marking, and includes fixes for `FilterVcf`, fingerprinting,
`RevertSam`, and several metrics tools. Upstream also removed direct `.sra`
input support and retained the Java 17 requirement.

## Reproducibility

- Upstream tag commit: `a53d7ebd975606b79ca0eaddb7bf93f555ec4db4`.
- Official release JAR SHA256:
  `b7d97861c3a54ba5a421f5a317f38382f955803862d30ef4aca2bcdc54943631`.
- The image records the upstream version, commit, JAR digest, and HTSJDK
  version in `/opt/picard/TAFFISH-SOURCE.txt`.

## Usage

Show TAFFISH wrapper help:

```sh
taf-picard --help
```

Show the upstream Picard program list:

```sh
taf-picard -- -h
taf-picard picard -h
```

Picard's top-level `--version` is not a valid upstream command. Check a
concrete Picard tool instead:

```sh
taf-picard picard FastqToSam --version
```

Create an unmapped BAM from FASTQ:

```sh
taf-picard picard FastqToSam \
  F1=reads.fq \
  O=reads.bam \
  SM=sample \
  RG=rg1
```

Convert BAM back to FASTQ:

```sh
taf-picard picard SamToFastq \
  I=reads.bam \
  FASTQ=roundtrip.fq
```

Create a reference dictionary:

```sh
taf-picard picard CreateSequenceDictionary \
  R=ref.fa \
  O=ref.dict
```

Sort, mark duplicates, and collect a metrics file:

```sh
taf-picard picard SortSam \
  I=input.bam \
  O=sorted.bam \
  SORT_ORDER=coordinate

taf-picard picard MarkDuplicates \
  I=sorted.bam \
  O=marked.bam \
  M=duplicate_metrics.txt
```

Run a metrics tool that uses R for chart output:

```sh
taf-picard picard MeanQualityByCycle \
  I=reads.bam \
  O=mean_quality.txt \
  CHART_OUTPUT=mean_quality.pdf
```

## Command Mode

This is a normal TAFFISH tool app with command mode enabled. The TAFFISH entry
is intentionally thin:

```taf
<taf-app:container:ghcr.io/taffish/picard:3.5.0-r1>
picard ::*ARGV*::
```

Because command mode treats a first non-option argument as an executable inside
the container, Picard program names should be passed through the `picard`
launcher:

```sh
taf-picard picard FastqToSam ...
taf-picard picard MarkDuplicates ...
```

The short form `taf-picard -- FastqToSam ...` also passes `FastqToSam` to the
default launcher, but the explicit `taf-picard picard ...` form is clearer and
matches the rest of the TAFFISH tool-app style.

## Runtime Notes

Set `PICARD_JAVA_XMX` to control the Java heap size. The default is `2g`:

```sh
PICARD_JAVA_XMX=8g taf-picard picard MarkDuplicates \
  I=input.bam \
  O=marked.bam \
  M=duplicate_metrics.txt \
  TMP_DIR=/work/tmp
```

For advanced JVM switches, use `JAVA_TOOL_OPTIONS`. For temporary files, prefer
Picard's own `TMP_DIR=path` argument, especially for large sorting, duplicate
marking, or merge jobs.

Picard supports both legacy `KEY=value` syntax and newer long-option syntax in
many tools. The examples here use the traditional Picard `KEY=value` style
because it remains common in existing pipelines and avoids ambiguity with the
TAFFISH wrapper option layer.

Picard 3.5.0 writes CRAM 3.1 by default. Readers limited to CRAM 3.0 cannot
open those outputs. Check downstream compatibility before exchanging CRAM
files; when CRAM 3.0 is required, convert explicitly with a compatible tool
that exposes format-version selection, such as `samtools view -O
cram,version=3.0` through `taf-samtools`.

## Platform Notes

The image is intended for native `linux/amd64` and `linux/arm64` builds.
Picard is distributed as a Java jar. HTSJDK 5.0.0 includes native
`jlibdeflate` support for both Linux architectures, while some bundled Intel
GKL acceleration libraries remain architecture-specific and may fall back to
Java implementations. The packaged functional paths remain available.

## Boundaries

This app packages the standard upstream `picard.jar`, Java 17, and R/Rscript.
It does not package GATK, samtools, bwa, reference genomes, interval lists,
liftover chain files, Illumina run folders, array vendor files, Google Cloud
credentials, the upstream `cloudJar` / `picardcloud.jar` variant, or external
workflow logic. Picard tools that require those resources still need
user-supplied local files or a separate purpose-built environment.

Picard 3.5.0 no longer accepts `.sra` input. Convert SRA accessions or files to
FASTQ first with `taf-sra-tools`, then pass those FASTQ files to Picard. The
standard JAR includes upstream GAR-reading code, but GAR/cloud-provider and
credential workflows are not exercised by the offline smoke suite.

The smoke tests verify container wiring, version binding, program-list help,
representative tool help, FASTA dictionary generation, FASTQ to BAM conversion,
BAM to FASTQ conversion, an R-backed quality-chart path, and CRAM 3.1 writing
and reading. They do not validate production-scale performance, every Picard
tool, every input format, GAR/Google Cloud path-provider behavior, or
scientific correctness on real sequencing datasets.

## Smoke Coverage

The app smoke metadata checks:

- `picard`, `java`, `Rscript`, `od`, and `sh` exist;
- concrete Picard tools report `Version:3.5.0` and the JAR/provenance checks
  match the pinned 3.5.0 release;
- the Picard program list includes representative tools;
- help is reachable for read, reference, metrics, interval, and VCF tools;
- `CreateSequenceDictionary` creates a valid sequence dictionary from a tiny
  FASTA;
- `FastqToSam` and `SamToFastq` round-trip a tiny FASTQ through BAM;
- `MeanQualityByCycle` produces metrics and a PDF chart, exercising the
  bundled R runtime;
- `SamFormatConverter` writes a tiny CRAM 3.1 file and `ValidateSamFile` reads
  it back using a local reference.

## Upstream

- Source: <https://github.com/broadinstitute/picard>
- Website: <https://broadinstitute.github.io/picard/>
- Release: <https://github.com/broadinstitute/picard/releases/tag/3.5.0>
- Download: <https://github.com/broadinstitute/picard/releases/download/3.5.0/picard.jar>
- License: MIT
- Citation: "Picard Toolkit." 2019. Broad Institute, GitHub Repository.
  <https://broadinstitute.github.io/picard/>; Broad Institute. Upstream also
  lists `biotools:picard_tools` and `RRID:SCR_006525`. No DOI or PMID is
  listed by the upstream citation block.
