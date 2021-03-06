#################################################
#  Short docker file to distribute some notebooks
#################################################

ARG FROMIMG_ARG=tristansalles/coastproc-bundle:latest
FROM ${FROMIMG_ARG}

##################################################
# Non standard as the files come from the packages

USER root

RUN apt-get update -qq && \
    DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    gettext && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

### RADWave - Notebooks
RUN python3 -m pip install --no-cache-dir \
    RADWave==0.0.2 \
    statsmodels \
    py-wave-runup \
    earthengine-api==0.1.173 \
    geopandas==0.4.1 \
    pytz==2019.1 \
    scikit-image==0.15.0 \
    scikit-learn==0.20.3 \
    shapely==1.6.4 \
    scipy==1.2.1 \
    spyder==3.3.4 \
    notebook==5.7.8 \
    astropy

RUN python3 -m pip install --no-cache-dir \
    joblib \
    oauth2client

WORKDIR /home/jovyan
RUN git clone --depth=1 https://github.com/TristanSalles/CoastSat.git
RUN cd /home/jovyan/CoastSat; python3 setup.py install

WORKDIR /home/jovyan
RUN git clone --depth=1 https://github.com/TristanSalles/pybeach.git
RUN cd pybeach; python3 setup.py install

WORKDIR /home/jovyan
RUN git clone --depth=1 https://github.com/matplotlib/legacycontour.git
RUN cd legacycontour; python3 setup.py install

ENV MODULE_DIR="src"
ADD --chown=jovyan:jovyan  $MODULE_DIR $MODULE_DIR
RUN   cd $MODULE_DIR && python3  -m pip install --no-cache-dir --no-deps --upgrade .

ADD  --chown=jovyan:jovyan Notebooks/0-StartHere.ipynb Notebooks/0-StartHere.ipynb

RUN python3 -m pip install --no-cache-dir --upgrade nose \
   jupyterhub notebook ipykernel
RUN python3 -m ipykernel install

RUN apt-get update -qq && \
    DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
        python-dev \
        python-pip

RUN python2 -m pip install --no-cache-dir --upgrade pip setuptools

RUN python2 -m pip install jupyter markupsafe zmq singledispatch backports_abc certifi jsonschema path.py matplotlib pandas plotly
RUN apt-get install -y libnetcdf-dev python-mpltoolkits.basemap
RUN python2 -m pip install scipy
RUN python2 -m pip install numpy
RUN python2 -m pip install cmocean
RUN python2 -m pip install colorlover
RUN python2 -m pip install scikit-fuzzy

RUN python2 -m pip install -e git+https://github.com/hplgit/odespy.git#egg=odespy

RUN python2 -m pip install --no-cache-dir --upgrade ipykernel
RUN python2 -m ipykernel install

WORKDIR /home/jovyan
RUN git clone https://github.com/pyReef-model/pyReefCore.git
RUN cd /home/jovyan/pyReefCore; python2 setup.py install  #python2 -m pip install -e /home/jovyan/pyReefCore
RUN cp -r /home/jovyan/pyReefCore/pyReefCore/* /usr/local/lib/python2.7/dist-packages/pyReefCore/

WORKDIR /home/jovyan
RUN git clone https://github.com/TristanSalles/CoastProc.git
RUN rm -rf /home/jovyan/Notebooks
RUN mv CoastProc/Notebooks /home/jovyan
RUN cd /home/jovyan/Notebooks/notebooks/LongTerm/wavesed;  f2py -c -m ocean ocean.f90

RUN mkdir -p /usr/local/files && chown -R jovyan:jovyan /usr/local/files
ADD --chown=jovyan:jovyan Docker/scripts  /usr/local/files
ENV PATH=/usr/local/files:${PATH}


# change ownership of everything
ENV NB_USER jovyan
RUN chown -R jovyan:jovyan /home/jovyan
USER jovyan

# Non standard as the files come from the packages
##################################################


## These are supplied by the build script
## build-dockerfile.sh

ARG IMAGENAME_ARG
ARG PROJ_NAME_ARG=CoastProc
ARG NB_PORT_ARG=8888
ARG NB_PASSWD_ARG=""
ARG NB_DIR_ARG="Notebooks"
ARG START_NB_ARG="0-StartHere.ipynb"

# The args need to go into the environment so they
# can be picked up by commands/templates (defined previously)
# when the container runs

ENV IMAGENAME=$IMAGENAME_ARG
ENV PROJ_NAME=$PROJ_NAME_ARG
ENV NB_PORT=$NB_PORT_ARG
ENV NB_PASSWD=$NB_PASSWD_ARG
ENV NB_DIR=$NB_DIR_ARG
ENV START_NB=$START_NB_ARG

## NOW INSTALL NOTEBOOKS

# (This is not standard - nothing to do here )

## The notebooks (and other files we are serving up)


# Trust all notebooks
RUN find -name \*.ipynb  -print0 | xargs -0 jupyter trust

# expose notebook port server port
EXPOSE $NB_PORT

VOLUME /home/jovyan/$NB_DIR/share

# note we use xvfb which to mimic the X display for lavavu
ENTRYPOINT ["/usr/local/bin/xvfbrun.sh"]

# launch notebook
ADD --chown=jovyan:jovyan Docker/scripts/run-jupyter.sh scripts/run-jupyter.sh
CMD scripts/run-jupyter.sh
