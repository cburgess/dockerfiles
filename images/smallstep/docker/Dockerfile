# If you use an ARG in your FROM line you have to declare it *before* the
# FROM line. You also have to re-declare it again *after* the FROM line.
ARG BASE_IMAGE
FROM ${BASE_IMAGE} AS builder

ARG STEP_CA_VERSION
ARG STEP_CLI_VERSION

WORKDIR /tmp/smallstep
RUN apt-get update \
&&  apt-get install -y wget \
&&  wget https://github.com/smallstep/cli/releases/download/v${STEP_CLI_VERSION}/step_linux_${STEP_CLI_VERSION}_amd64.tar.gz \
&&  wget https://github.com/smallstep/certificates/releases/download/v${STEP_CA_VERSION}/step-certificates_linux_${STEP_CA_VERSION}_amd64.tar.gz \
&&  tar -zxvf step_linux_${STEP_CLI_VERSION}_amd64.tar.gz\
&&  tar -zxvf step-certificates_linux_${STEP_CA_VERSION}_amd64.tar.gz

# Build the final image
FROM ${BASE_IMAGE}

# Docker is weird, you have to re-declare ARG *after* FROM
ARG BASE_IMAGE
ARG IMAGE_NAME
ARG IMAGE_VERSION
ARG VCS_REV
ARG VCS_URL
ARG BUILD_DATE
ARG MAINTAINER
ARG BUILD_TAG

ARG STEP_CA_VERSION
ARG STEP_CLI_VERSION

ENV STEP_DATA_DIR=/var/local/step

RUN adduser --disabled-password --gecos '' step \
&&  mkdir ${STEP_DATA_DIR} \
&&  chown -R step:step ${STEP_DATA_DIR} \
&&  apt-get update \
&&  apt-get -y dist-upgrade \
&&  rm -rf /var/lib/apt/*

COPY --from=builder /tmp/smallstep/step_${STEP_CLI_VERSION}/bin/step /usr/local/bin/step
COPY --from=builder /tmp/smallstep/step-certificates_${STEP_CA_VERSION}/bin/step-ca /usr/local/bin/step-ca

ENV CONFIGPATH="${STEP_DATA_DIR}/config/ca.json"
ENV PWDPATH="${STEP_DATA_DIR}/secrets/password"
ENV STEPDEBUG=1
ENV STEPPATH=${STEP_DATA_DIR}
ENV STEP=${STEP_DATA_DIR}

LABEL \
  org.label-schema.schema-version="1.0"                 \
  org.label-schema.vendor="Smallstep Labs, Inc."        \
  org.label-schema.name="${IMAGE_NAME}"                 \
  org.label-schema.version="${IMAGE_VERSION}"           \
  org.label-schema.url="https://smallstep.com"          \
  org.label-schema.vcs-url="${VCS_URL}"                 \
  org.label-schema.vcs-ref="${VCS_REV}"                 \
  org.label-schema.build-date="${BUILD_DATE}"           \
  org.liquidreality.docker-base-image="${BASE_IMAGE}"   \
  org.liquidreality.docker-build-tag="${BUILD_TAG}"     \
  license="Apache-2.0"

USER step
WORKDIR ${STEP_DATA_DIR}
VOLUME ${STEP_DATA_DIR}
STOPSIGNAL SIGTERM
CMD exec /bin/sh -c "/usr/local/bin/step-ca --password-file $PWDPATH $CONFIGPATH"
