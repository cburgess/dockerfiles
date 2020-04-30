FROM debian:buster-slim AS builder

ARG STEP_CA_VERSION
ARG STEP_CLI_VERSION

WORKDIR /tmp/smallstep
RUN apt-get update \
&&  apt-get install -y wget \
&&  wget https://github.com/smallstep/cli/releases/download/v${STEP_CLI_VERSION}/step_linux_${STEP_CLI_VERSION}_amd64.tar.gz \
&&  wget https://github.com/smallstep/certificates/releases/download/v${STEP_CA_VERSION}/step-certificates_linux_${STEP_CA_VERSION}_amd64.tar.gz \
&&  tar -zxvf step_linux_0.14.2_amd64.tar.gz \
&&  tar -zxvf step-certificates_linux_0.14.3_amd64.tar.gz

# Build the final image
FROM debian:buster-slim

ARG STEP_CA_VERSION
ARG STEP_CLI_VERSION
ARG STEP_DATA_DIR=/var/local/step

RUN adduser --disabled-password --gecos '' step \
&&  mkdir ${STEP_DATA_DIR} \
&&  chown -R step:step ${STEP_DATA_DIR}

COPY --from=builder /tmp/smallstep/step_${STEP_CLI_VERSION}/bin/step /usr/local/bin/step
COPY --from=builder /tmp/smallstep/step-certificates_${STEP_CA_VERSION}/bin/step-ca /usr/local/bin/step-ca

ENV CONFIGPATH="${STEP_DATA_DIR}/config/ca.json"
ENV PWDPATH="${STEP_DATA_DIR}/secrets/password"
ENV STEPDEBUG=1
ENV STEPPATH=${STEP_DATA_DIR}
ENV STEP=${STEP_DATA_DIR}

USER step
WORKDIR ${STEP_DATA_DIR}
VOLUME ${STEP_DATA_DIR}
STOPSIGNAL SIGTERM
CMD exec /bin/sh -c "/usr/local/bin/step-ca --password-file $PWDPATH $CONFIGPATH"
