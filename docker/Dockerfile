FROM debian:12-slim AS fetcher

ARG PICARD_VERSION=3.4.0
ARG PICARD_SHA256=e76128c283889fc583c9dea33a3b7448974c067d102c9e35be152642d4d5f901

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      curl; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /opt/picard

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fL \
      "https://github.com/broadinstitute/picard/releases/download/${PICARD_VERSION}/picard.jar" \
      -o picard.jar; \
    echo "${PICARD_SHA256}  picard.jar" | sha256sum -c -

FROM debian:12-slim

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends \
      ca-certificates \
      libgomp1 \
      openjdk-17-jre-headless \
      r-base; \
    rm -rf /var/lib/apt/lists/*

COPY --from=fetcher /opt/picard/picard.jar /opt/picard/picard.jar
COPY docker/picard /usr/local/bin/picard

ENV PICARD_JAR=/opt/picard/picard.jar
ENV PICARD_JAVA_XMX=2g

RUN set -eux; \
    chmod 0755 /usr/local/bin/picard; \
    command -v java >/dev/null; \
    command -v Rscript >/dev/null; \
    command -v picard >/dev/null; \
    java -version; \
    Rscript --version; \
    picard FastqToSam --version 2>&1 | grep -Fx "Version:3.4.0" >/dev/null; \
    picard -h 2>&1 | grep -F "Available Programs:" >/dev/null; \
    picard -h 2>&1 | grep -F "FastqToSam" >/dev/null; \
    picard FastqToSam --help 2>&1 | grep -F "USAGE: FastqToSam" >/dev/null; \
    picard CreateSequenceDictionary --help 2>&1 | grep -F "USAGE: CreateSequenceDictionary" >/dev/null; \
    mkdir -p /tmp/picard-check; \
    cd /tmp/picard-check; \
    printf '>chr1\nACGTACGTACGTACGT\n' > ref.fa; \
    PICARD_JAVA_XMX=512m picard CreateSequenceDictionary R=ref.fa O=ref.dict QUIET=true >/tmp/picard-dict.log 2>&1; \
    test -s ref.dict; \
    grep -F 'SN:chr1' ref.dict >/dev/null; \
    grep -F 'LN:16' ref.dict >/dev/null; \
    printf '@r1\nACGTACGT\n+\nIIIIIIII\n' > reads.fq; \
    PICARD_JAVA_XMX=512m picard FastqToSam F1=reads.fq O=reads.bam SM=sample RG=rg1 QUALITY_FORMAT=Standard QUIET=true >/tmp/picard-fastqtosam.log 2>&1; \
    test -s reads.bam; \
    PICARD_JAVA_XMX=512m picard SamToFastq I=reads.bam FASTQ=roundtrip.fq QUIET=true >/tmp/picard-samtofastq.log 2>&1; \
    test -s roundtrip.fq; \
    grep -F '@r1' roundtrip.fq >/dev/null; \
    grep -F 'ACGTACGT' roundtrip.fq >/dev/null; \
    PICARD_JAVA_XMX=512m picard MeanQualityByCycle I=reads.bam O=mean_quality.txt CHART_OUTPUT=mean_quality.pdf QUIET=true >/tmp/picard-mean-quality.log 2>&1; \
    test -s mean_quality.txt; \
    test -s mean_quality.pdf; \
    grep -F 'MEAN_QUALITY' mean_quality.txt >/dev/null; \
    cd /; \
    rm -rf /tmp/picard-check /tmp/picard-dict.log /tmp/picard-fastqtosam.log /tmp/picard-samtofastq.log /tmp/picard-mean-quality.log

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=picard
