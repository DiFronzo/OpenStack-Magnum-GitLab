FROM ubuntu:18.04
MAINTAINER Andreas Lien

ENV USER root
ENV GITLAB_RUNNER_VERSION=latest DOCKER_MACHINE_VERSION=0.16.2
ARG URL
ARG TOKEN

RUN apt-get update && apt-get install -y \
                  wget \
                  grep \
                  build-essential \
                  devscripts \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

RUN wget -O /usr/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/${GITLAB_RUNNER_VERSION}/binaries/gitlab-runner-linux-amd64 && \
        chmod +x /usr/bin/gitlab-runner && \
        ln -s /usr/bin/gitlab-runner /usr/bin/gitlab-ci-multi-runner && \
        wget -q https://github.com/docker/machine/releases/download/v${DOCKER_MACHINE_VERSION}/docker-machine-Linux-x86_64 -O /usr/bin/docker-machine && \
        chmod +x /usr/bin/docker-machine && \
        mkdir -p /etc/gitlab-runner/certs && \
        chmod -R 700 /etc/gitlab-runner

RUN gitlab-runner register \
  --non-interactive \
  --url ${URL} \
  --registration-token ${TOKEN} \
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --tag-list "docker,aws" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"

WORKDIR /etc/gitlab-runner
CMD A=$(sed -rn 's/^.*token = "(.*)"/\1/p' config.toml); sed -i "s/token\ =/token\ =\ \"$A\"/" /mnt/gitlab-runner-k8s.yaml
