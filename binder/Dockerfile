FROM jupyter/base-notebook:b37d13027dc7

# Install Linux prereqs
USER root 
RUN apt-get update && apt-get install -yq --no-install-recommends \
    build-essential \
    git \
    libsm6 \
    libxext-dev \
    libxrender1 \
    netcat \
    tzdata \
    unzip \
    nano \
    && rm -rf /var/lib/apt/lists/*

# Jupyter stuff from jupyter/scipy-notebook. Maybe useful. 
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager@^1.0.1 --no-build && \
    jupyter lab build --dev-build=False && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Julia 
ENV JULIA_VERSION=1.2.0
ENV JULIA_DEPOT_PATH=/opt/julia
ENV JULIA_PKGDIR=/opt/julia
RUN mkdir /opt/julia-${JULIA_VERSION} && \
    cd /tmp && \
    wget -q https://julialang-s3.julialang.org/bin/linux/x64/`echo ${JULIA_VERSION} | cut -d. -f 1,2`/julia-${JULIA_VERSION}-linux-x86_64.tar.gz && \
    echo "926ced5dec5d726ed0d2919e849ff084a320882fb67ab048385849f9483afc47 *julia-${JULIA_VERSION}-linux-x86_64.tar.gz" | sha256sum -c - && \
    tar xzf julia-${JULIA_VERSION}-linux-x86_64.tar.gz -C /opt/julia-${JULIA_VERSION} --strip-components=1 && \
    rm /tmp/julia-${JULIA_VERSION}-linux-x86_64.tar.gz

# Define the `julia` shell command
RUN ln -fs /opt/julia-*/bin/julia /usr/local/bin/julia

# Copy in repo 
COPY . ${HOME}
USER root
RUN chown -R ${NB_UID} ${HOME}
RUN chown -R ${NB_UID} ${JULIA_PKGDIR}-${JULIA_VERSION}

# Julia setup 
USER ${NB_USER}
WORKDIR ${HOME}
RUN julia -e "using Pkg; pkg\"add IJulia InstantiateFromURL\""
RUN julia -e "using Pkg; pkg\"activate .\"; pkg\"instantiate\""

# PackageCompiler 
RUN julia -e "using Pkg; pkg\"add GR Plots StatsPlots DataFrames StatsPlots CSV PlotUtils GeometryTypes Tables PackageCompiler#sd-notomls CategoricalArrays IteratorInterfaceExtensions PooledArrays WeakRefStrings\""    
RUN julia -e "using Pkg; pkg\"add IJulia Images DualNumbers Unitful Compat LaTeXStrings UnicodePlots DataValues IterativeSolvers VisualRegressionTests\"" 
USER root 
RUN julia -e "using PackageCompiler; syso, sysold = PackageCompiler.compile_incremental(:Plots, :DataFrames, :CSV, :StatsPlots, install = true); cp(syso, sysold, force = true)" 
USER ${NB_USER}
RUN julia -e "using Plots, Distributions, NLsolve, StaticArrays, ForwardDiff, QuantEcon, Parameters, Expectations, StatsPlots, Roots, Interpolations, Optim"
