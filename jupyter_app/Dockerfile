FROM jupyter/minimal-notebook:latest

USER root

RUN apt-get update && apt-get install -y --no-install-recommends \
    wget gnupg ca-certificates procps \
    libxss1 libasound2 libnspr4 libnss3 xdg-utils \
    fonts-liberation && \
    apt-get install -y --no-install-recommends chromium-browser && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER ${NB_UID}

COPY --chown=${NB_UID}:${NB_GID} requirements.txt /tmp/
RUN pip install --no-cache-dir -r /tmp/requirements.txt && \
    fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

COPY --chown=${NB_UID}:${NB_GID} . /home/jovyan/work/