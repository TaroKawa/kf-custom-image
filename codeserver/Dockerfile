ARG BASE_IMG=nvcr.io/nvidia/pytorch:22.09-py3
FROM $BASE_IMG

ARG TARGETARCH

# Common environment variables
ENV NB_USER jovyan
ENV NB_UID 1000
ENV NB_PREFIX /
ENV HOME /home/$NB_USER
ENV SHELL /bin/bash

# We copy the contents of $HOME_TMP to $HOME on startup
# This is to work around the fact that a PVC will be mounted to $HOME
# but we still want to have some default files in $HOME
# See `s6/cont-init.d/01-copy-tmp-home`
ENV HOME_TMP /tmp_home/$NB_USER

# s6 only gives 5 seconds by default, which is too small for slow PVC storage backends
# When running `/etc/cont-inid.d/01-copy-tmp-home` (note, this is in milliseconds)
ENV S6_CMD_WAIT_FOR_SERVICES_MAXTIME 300000

# Software versions
ARG KUBECTL_VERSION=v1.27.14
ARG S6_VERSION=v3.1.6.2
ARG CODESERVER_VERSION=v4.89.1
ARG CODESERVER_PYTHON_VERSION=2024.4.1
ARG MINIFORGE_VERSION=24.3.0-0
ARG PIP_VERSION=24.0
ARG PYTHON_VERSION=3.11.9

# Set shell to bash
SHELL ["/bin/bash", "-c"]

# Install useful Linux packages
RUN export DEBIAN_FRONTEND=noninteractive \
 && apt-get -yq update \
 && apt-get -yq install --no-install-recommends \
    apt-transport-https \
    bash \
    bzip2 \
    ca-certificates \
    curl \
    git \
    gnupg \
    gnupg2 \
    locales \
    lsb-release \
    nano \
    software-properties-common \
    tzdata \
    unzip \
    vim \
    wget \
    xz-utils \
    zip \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Install s6 overlay
RUN case "${TARGETARCH}" in \
      amd64) S6_ARCH="x86_64" ;; \
      arm64) S6_ARCH="aarch64" ;; \
      ppc64le) S6_ARCH="ppc64le" ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
    esac \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-noarch.tar.xz" -o /tmp/s6-overlay-noarch.tar.xz \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-noarch.tar.xz.sha256" -o /tmp/s6-overlay-noarch.tar.xz.sha256 \
 && echo "$(cat /tmp/s6-overlay-noarch.tar.xz.sha256 | awk '{ print $1; }')  /tmp/s6-overlay-noarch.tar.xz" | sha256sum -c - \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" -o /tmp/s6-overlay-${S6_ARCH}.tar.xz \
 && curl -fsSL "https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-${S6_ARCH}.tar.xz.sha256" -o /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256 \
 && echo "$(cat /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256 | awk '{ print $1; }')  /tmp/s6-overlay-${S6_ARCH}.tar.xz" | sha256sum -c - \
 && tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
 && tar -C / -Jxpf /tmp/s6-overlay-${S6_ARCH}.tar.xz \
 && rm /tmp/s6-overlay-noarch.tar.xz  \
       /tmp/s6-overlay-noarch.tar.xz.sha256 \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz \
       /tmp/s6-overlay-${S6_ARCH}.tar.xz.sha256

# Install kubectl
RUN curl -fsSL "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" -o /usr/local/bin/kubectl \
 && curl -fsSL "https://dl.k8s.io/${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl.sha256" -o /tmp/kubectl.sha256 \
 && echo "$(cat /tmp/kubectl.sha256 | awk '{ print $1; }')  /usr/local/bin/kubectl" | sha256sum -c - \
 && rm /tmp/kubectl.sha256 \
 && chmod +x /usr/local/bin/kubectl

# Create user and set required ownership
RUN useradd -M -s /bin/bash -N -u ${NB_UID} ${NB_USER} \
 && mkdir -p ${HOME} \
 && mkdir -p ${HOME_TMP} \
 && chown -R ${NB_USER}:users ${HOME} \
 && chown -R ${NB_USER}:users ${HOME_TMP} \
 && chown -R ${NB_USER}:users /usr/local/bin

# Set locale configs
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
 && locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# s6 - copy scripts
COPY --chown=${NB_USER}:users --chmod=755 s6/ /etc

USER $NB_UID

ENTRYPOINT ["/init"]

# Install code-server
USER root
RUN curl -fsSL "https://github.com/coder/code-server/releases/download/${CODESERVER_VERSION}/code-server_${CODESERVER_VERSION/v/}_${TARGETARCH}.deb" -o /tmp/code-server.deb \
 && dpkg -i /tmp/code-server.deb \
 && rm -f /tmp/code-server.deb

## Setup environment for conda
#ENV CONDA_DIR /opt/conda
#ENV PATH "${CONDA_DIR}/bin:${PATH}"
#RUN mkdir -p ${CONDA_DIR} \
# && echo ". /opt/conda/etc/profile.d/conda.sh" >> ${HOME}/.bashrc \
# && echo ". /opt/conda/etc/profile.d/conda.sh" >> /etc/profile \
# && echo "conda activate base" >> ${HOME}/.bashrc \
# && echo "conda activate base" >> /etc/profile \
# && chown -R ${NB_USER}:users ${CONDA_DIR} \
# && chown -R ${NB_USER}:users ${HOME}

USER $NB_UID

## Install conda, pip, python
#RUN case "${TARGETARCH}" in \
#      amd64) MINIFORGE_ARCH="x86_64" ;; \
#      arm64) MINIFORGE_ARCH="aarch64" ;; \
#      ppc64le) MINIFORGE_ARCH="ppc64le" ;; \
#      *) echo "Unsupported architecture: ${TARGETARCH}"; exit 1 ;; \
#    esac \
# && curl -fsSL "https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-${MINIFORGE_VERSION}-Linux-${MINIFORGE_ARCH}.sh" -o /tmp/Miniforge3.sh \
# && curl -fsSL "https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/Miniforge3-${MINIFORGE_VERSION}-Linux-${MINIFORGE_ARCH}.sh.sha256" -o /tmp/Miniforge3.sh.sha256 \
# && echo "$(cat /tmp/Miniforge3.sh.sha256 | awk '{ print $1; }')  /tmp/Miniforge3.sh" | sha256sum -c - \
# && rm /tmp/Miniforge3.sh.sha256 \
# && /bin/bash /tmp/Miniforge3.sh -b -f -p ${CONDA_DIR} \
# && rm /tmp/Miniforge3.sh \
# && conda config --system --set auto_update_conda false \
# && conda config --system --set show_channel_urls true \
# && echo "python ==${PYTHON_VERSION}" >> ${CONDA_DIR}/conda-meta/pinned \
# && conda install -y -q \
#    python=${PYTHON_VERSION} \
#    pip=${PIP_VERSION} \
# && conda update -y -q --all \
# && conda clean -a -f -y

# Install requirements.txt
COPY --chown=${NB_USER}:users requirements.txt /tmp
RUN python3 -m pip install -r /tmp/requirements.txt --quiet --no-cache-dir \
 && rm -f /tmp/requirements.txt

# Install codeserver extensions
RUN code-server --install-extension "ms-python.python@${CODESERVER_PYTHON_VERSION}" --force \
 && code-server --list-extensions --show-versions

# s6 - 01-copy-tmp-home
# NOTE: the contents of $HOME_TMP are copied to $HOME at runtime
#       this is a workaround because a PVC will be mounted at $HOME
#       and the contents of $HOME will be hidden
RUN cp -r -T "${HOME}" "${HOME_TMP}"

USER $NB_UID

# Expose port 8888
EXPOSE 8888
