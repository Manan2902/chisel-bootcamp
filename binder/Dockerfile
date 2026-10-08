FROM eclipse-temurin:8-jre-jammy

# Binder supplies these build arguments.
ARG NB_USER=jovyan
ARG NB_UID=1000

ENV USER=${NB_USER}
ENV HOME=/home/${NB_USER}
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8

# Install Python/Jupyter and Graphviz.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        python3 \
        python3-pip \
        graphviz \
        ca-certificates \
        curl \
        passwd && \
    rm -rf /var/lib/apt/lists/*

# Use the Jupyter versions that are appropriate for this old
# Scala/Almond-based Bootcamp.
RUN python3 -m pip install --no-cache-dir --upgrade pip && \
    python3 -m pip install --no-cache-dir \
        notebook==6.5.7 \
        jupyterlab==3.6.8

# Create the Binder user.
RUN useradd \
        --create-home \
        --user-group \
        --uid ${NB_UID} \
        --shell /bin/bash \
        ${NB_USER}

# Put the repository inside the user's home directory.
WORKDIR ${HOME}/chisel-bootcamp
COPY . ${HOME}/chisel-bootcamp

# Preserve the Bootcamp's custom JavaScript.
RUN mkdir -p ${HOME}/.jupyter/custom && \
    cp ${HOME}/chisel-bootcamp/source/custom.js \
       ${HOME}/.jupyter/custom/custom.js

# Install Coursier and Almond (Scala Jupyter kernel).
RUN curl -fL -o /tmp/coursier \
        https://github.com/coursier/coursier/releases/download/v2.1.26/coursier && \
    chmod +x /tmp/coursier && \
    /tmp/coursier bootstrap \
        -r jitpack \
        sh.almond:scala-kernel_2.12.10:0.9.1 \
        --sources \
        --default=true \
        -o /tmp/almond && \
    /tmp/almond --install --global && \
    rm -f /tmp/coursier /tmp/almond && \
    rm -rf /root/.cache/coursier

# Make everything editable by the Binder user.
RUN chown -R ${NB_UID}:${NB_UID} ${HOME}

USER ${NB_USER}

WORKDIR ${HOME}/chisel-bootcamp

EXPOSE 8888

CMD ["jupyter", "notebook", "--no-browser", "--ip=0.0.0.0", "--port=8888", "--NotebookApp.default_url=/tree/"]
