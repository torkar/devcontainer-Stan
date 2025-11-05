FROM r-base:4.5.2

##### root stuff #####

# Prereqs and build toolchain (needed for cmdstan)
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends \
        sudo apt-utils \
        python3 python3-dev pipx \
        git htop curl ca-certificates \
        r-cran-brms r-cran-remotes r-cran-shape \
        libcurl4-openssl-dev libfontconfig1-dev libnode-dev libxml2-dev \
        libcairo2-dev libtiff-dev libtbb-dev \
        build-essential gfortran cmake pkg-config \
    && rm -rf /var/lib/apt/lists/*

# Configure a new user 'codespace'
RUN echo "codespace ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/nopasswd && \
    chmod 0440 /etc/sudoers.d/nopasswd && \
    useradd -m -s /bin/bash codespace

# Root: R compile flags (speedy compiles for system-wide installs)
RUN mkdir -p /root/.R && \
    printf "CXX17FLAGS=-O3 -march=native -mtune=native -fPIC\nCXX17=g++\n" >> /root/.R/Makevars && \
    printf "options(mc.cores = 4)\nSys.setenv(MAKEFLAGS='-j4')\n" >> /root/.Rprofile

# Install some extra R packages (system-wide)
RUN install2.r --error \
      ggdag \
      languageserver \
    && rm -rf /tmp/downloaded_packages /root/.R*

# Install cmdstanr, rethinking, httpgd system-wide
RUN R -e "remotes::install_github('stan-dev/cmdstanr')" && \
    R -e "remotes::install_github('rmcelreath/rethinking')" && \
    R -e "remotes::install_github('nx10/httpgd')"

##### user stuff #####

USER codespace
WORKDIR /home/codespace

# Make HOME explicit for this user
ENV HOME=/home/codespace
# pipx installs into ~/.local/bin; make it available for RUN and at runtime
ENV PATH=/home/codespace/.local/bin:${PATH}

# User-level R flags
RUN mkdir -p $HOME/.R && \
    printf "CXX17FLAGS=-O3 -march=native -mtune=native -fPIC\nCXX17=g++\n" >> $HOME/.R/Makevars && \
    printf "options(mc.cores = 4)\nSys.setenv(MAKEFLAGS='-j4')\n" >> $HOME/.Rprofile

# Install cmdstan for the user (needs toolchain installed above)
RUN R -e "cmdstanr::install_cmdstan(cores = 4)"

# Use radian (nice REPL); pipx already installed as root, usable as user
RUN pipx install radian
# Optional convenience alias for interactive shells
RUN echo "alias r='radian'" >> $HOME/.bashrc

# --- Julia via juliaup ---
RUN curl -fsSL https://install.julialang.org | sh -s -- -y

# Put Julia shims on PATH (now that HOME is defined)
ENV PATH=/home/codespace/.juliaup/bin:${PATH}

# Install + precompile Julia pkgs (guaranteed compiled into the image)
RUN julia -e 'import Pkg; Pkg.add("Turing"); Pkg.precompile()'

CMD ["/bin/bash"]