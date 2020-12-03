FROM difronzo/openstack-cli:ubuntu
MAINTAINER Andreas Lien
USER root
SHELL ["/bin/bash", "-c"]

ENV KUBECONFIG=/root/clusters/kubernetes-cluster/config
ENV FILENAME=gitlab-runner-k8s.yaml
ARG CLUSTER
ARG OS_REGION_NAME
ARG OS_PROJECT_DOMAIN_ID
ARG OS_INTERFACE
ARG OS_AUTH_URL
ARG OS_USERNAME
ARG OS_PROJECT_ID
ARG OS_USER_DOMAIN_NAME
ARG OS_PROJECT_NAME
ARG OS_PASSWORD
ARG OS_IDENTITY_API_VERSION

RUN curl -O https://storage.googleapis.com/kubernetes-release/release/v1.19.4/bin/linux/amd64/kubectl \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/kubectl

RUN mkdir -p ~/clusters/kubernetes-cluster
RUN eval $(openstack coe cluster config ${CLUSTER} --dir ~/clusters/kubernetes-cluster)

CMD kubectl apply -f /mnt/${FILENAME}
