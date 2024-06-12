# ベースイメージを指定
FROM nvcr.io/nvidia/pytorch:22.09-py3

# ARGを指定
ARG DOCKER_USER=jovyan
ARG DOCKER_PASSWORD=jovyan

# 必要なパッケージをインストール
USER root
RUN apt-get update && apt-get install -y \
    sudo \
    git \
    curl \
    && rm -rf /var/lib/apt/lists/*

# jovyanユーザーを作成
RUN useradd -m -s /bin/bash -N -u 1000 ${DOCKER_USER} \
    && usermod -aG sudo ${DOCKER_USER} \
    && echo "${DOCKER_USER}:${DOCKER_PASSWORD}" | chpasswd

# 作業ディレクトリを設定
WORKDIR /home/jovyan

# 必要なPythonパッケージをインストール
RUN pip install jupyterlab

# ポート8888を開放
EXPOSE 8888

# JupyterLabの起動設定
CMD ["sh", "-c", "jupyter lab --no-browser --port=8888 --ip=0.0.0.0 --NotebookApp.token='' --NotebookApp.base_url=${NB_PREFIX} --NotebookApp.allow_origin='*'"]

# 権限をjovyanユーザーに変更
USER ${DOCKER_USER}

