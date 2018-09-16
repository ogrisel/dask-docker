FROM jupyter/base-notebook

USER root
RUN apt-get update \
  && apt-get install -yq graphviz git vim build-essential \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*
USER $NB_USER

RUN conda update conda 
RUN conda install --yes \
    python-blosc \
    cytoolz \
    dask==0.19.1 \
    nomkl \
    numpy==1.15.1 \
    pandas==0.23.4 \
    ipywidgets \
    jupyterlab==0.34.7 \
    && conda clean -tipsy
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager dask-labextension

RUN pip install -U --no-cache-dir cython pip
RUN pip install \
    matplotlib \
    joblib \
    scikit-learn==0.20.rc1 \
    kubernetes \
    git+https://github.com/dask/dask-ml.git \
    git+https://github.com/dask/dask-kubernetes.git \
    graphviz \
    nbgitpuller \
    --no-cache-dir

RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager dask-labextension

USER root
COPY worker-template.yaml /home/$NB_USER/worker-template.yaml
COPY custom-dask-config.yaml /home/$NB_USER/.config/dask/custom-dask-config.yaml
COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh
RUN mkdir /opt/app
COPY examples/ /home/$NB_USER/examples
RUN chown -R $NB_USER /home/$NB_USER/examples
USER $NB_USER


ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
CMD ["start.sh", "jupyter", "lab"]
