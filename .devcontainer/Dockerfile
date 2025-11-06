FROM r-base:4.5.2

LABEL maintainer="richard.torkar@gmail.com"
LABEL description="Dev container with R, Stan, rethinking, brms, and Julia"

# Build arguments you can customize
ARG MAKEFLAGS="-j4"
ARG MC_CORES="4"
ARG USERNAME="codespace"

##### Root: System packages and configuration #####

# Install system dependencies and build toolchain
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends \
        # User management
        sudo \
        apt-utils \
        # Python tooling
        python3 python3-dev pipx \
        # Utilities
        git htop curl ca-certificates \
        # R packages
        r-cran-brms r-cran-remotes r-cran-shape \
        # Development libraries
        libcurl4-openssl-dev libfontconfig1-dev libnode-dev libxml2-dev \
        libcairo2-dev libtiff-dev libtbb-dev \
        # Build tools
        build-essential gfortran cmake pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Create user with sudo privileges
RUN useradd -m -s /bin/bash ${USERNAME} && \
    echo "${USERNAME} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/nopasswd && \
    chmod 0440 /etc/sudoers.d/nopasswd

# Configure R compiler flags (root)
RUN mkdir -p /root/.R && \
    printf "CXX17FLAGS=-O3 -march=native -mtune=native -fPIC\nCXX17=g++\n" > /root/.R/Makevars && \
    printf "options(mc.cores = ${MC_CORES})\nSys.setenv(MAKEFLAGS='${MAKEFLAGS}')\n" > /root/.Rprofile

# Install R packages system-wide
RUN install2.r --error --skipinstalled \
        ggdag \
        languageserver \
    && rm -rf /tmp/downloaded_packages

# Install R packages from GitHub (consider pinning commits/tags for reproducibility)
RUN R -e "remotes::install_github('stan-dev/cmdstanr')" && \
    R -e "remotes::install_github('rmcelreath/rethinking')" && \
    R -e "remotes::install_github('nx10/httpgd')" && \
    rm -rf /tmp/downloaded_packages /tmp/Rtmp*

##### User: Application setup #####

USER ${USERNAME}
WORKDIR /home/${USERNAME}

# Set user environment variables
ENV HOME=/home/${USERNAME} \
    PATH=/home/${USERNAME}/.local/bin:/home/${USERNAME}/.juliaup/bin:${PATH}

# Configure R for user and install cmdstan
RUN mkdir -p ${HOME}/.R && \
    printf "CXX17FLAGS=-O3 -march=native -mtune=native -fPIC\nCXX17=g++\n" > ${HOME}/.R/Makevars && \
    printf "options(mc.cores = ${MC_CORES})\nSys.setenv(MAKEFLAGS='${MAKEFLAGS}')\n" > ${HOME}/.Rprofile && \
    R -e "cmdstanr::install_cmdstan(cores = ${MC_CORES})"

# Install and configure radian REPL
RUN pipx install radian && \
    echo "alias r='radian'" >> ${HOME}/.bashrc

# Install Julia via juliaup
RUN curl -fsSL https://install.julialang.org | sh -s -- -y

# Install and precompile Julia packages
RUN julia -e 'import Pkg; Pkg.add("Turing"); Pkg.precompile()'

CMD ["/bin/bash"]