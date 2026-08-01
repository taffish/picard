FROM debian:12-slim AS fetcher

ARG PICARD_VERSION=3.5.0
ARG PICARD_SHA256=b7d97861c3a54ba5a421f5a317f38382f955803862d30ef4aca2bcdc54943631
ARG PICARD_COMMIT=a53d7ebd975606b79ca0eaddb7bf93f555ec4db4
ARG HTSJDK_VERSION=5.0.0

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            curl \
            unzip; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

WORKDIR /opt/picard

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fL \
      "https://github.com/broadinstitute/picard/releases/download/${PICARD_VERSION}/picard.jar" \
      -o picard.jar; \
    echo "${PICARD_SHA256}  picard.jar" | sha256sum -c -; \
    unzip -p picard.jar META-INF/MANIFEST.MF | tr -d '\r' > picard-manifest.txt; \
    grep -Fx "Implementation-Version: ${PICARD_VERSION}" picard-manifest.txt >/dev/null; \
    grep -Fx "htsjdk-Version: ${HTSJDK_VERSION}" picard-manifest.txt >/dev/null; \
    printf '%s\n' \
      "upstream_version=${PICARD_VERSION}" \
      "upstream_commit=${PICARD_COMMIT}" \
      "jar_sha256=${PICARD_SHA256}" \
      "htsjdk_version=${HTSJDK_VERSION}" \
      > TAFFISH-SOURCE.txt

FROM debian:12-slim

ARG PICARD_VERSION=3.5.0
ARG PICARD_SHA256=b7d97861c3a54ba5a421f5a317f38382f955803862d30ef4aca2bcdc54943631
ARG PICARD_COMMIT=a53d7ebd975606b79ca0eaddb7bf93f555ec4db4
ARG HTSJDK_VERSION=5.0.0

ENV DEBIAN_FRONTEND=noninteractive

RUN set -eux; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            ca-certificates \
            libgomp1 \
            openjdk-17-jre-headless \
            r-base; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*

COPY --from=fetcher /opt/picard/picard.jar /opt/picard/picard.jar
COPY --from=fetcher /opt/picard/TAFFISH-SOURCE.txt /opt/picard/TAFFISH-SOURCE.txt
COPY docker/picard /usr/local/bin/picard

ENV PICARD_JAR=/opt/picard/picard.jar
ENV PICARD_JAVA_XMX=2g

RUN set -eux; \
    chmod 0755 /usr/local/bin/picard; \
    command -v java >/dev/null; \
    command -v Rscript >/dev/null; \
    command -v od >/dev/null; \
    command -v picard >/dev/null; \
    java -version; \
    Rscript --version; \
    sha256sum /opt/picard/picard.jar | grep -Fx "${PICARD_SHA256}  /opt/picard/picard.jar" >/dev/null; \
    grep -Fx "upstream_version=${PICARD_VERSION}" /opt/picard/TAFFISH-SOURCE.txt >/dev/null; \
    grep -Fx "upstream_commit=${PICARD_COMMIT}" /opt/picard/TAFFISH-SOURCE.txt >/dev/null; \
    grep -Fx "htsjdk_version=${HTSJDK_VERSION}" /opt/picard/TAFFISH-SOURCE.txt >/dev/null; \
    picard FastqToSam --version 2>&1 | grep -Fx "Version:${PICARD_VERSION}" >/dev/null; \
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
    cd /; \
    rm -rf /tmp/picard-check /tmp/picard-dict.log

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=picard
