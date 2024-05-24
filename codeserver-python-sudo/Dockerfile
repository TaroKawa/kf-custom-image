FROM kubeflownotebookswg/codeserver-python

ARG DOCKER_USER=jovyan
ARG DOCKER_PASSWORD=jovyan

USER root

RUN usermod -aG sudo ${DOCKER_USER} && echo "${DOCKER_USER}:${DOCKER_PASSWORD}" | chpasswd

RUN apt-get update && apt-get -y install sudo git

USER ${DOCKER_USER}
