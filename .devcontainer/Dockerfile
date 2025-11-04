FROM r-base:4.5.2

##### root stuff #####
RUN apt modernize-sources -y && apt-get update -qq && apt-get install -y sudo \
    apt-utils python3 pipx python3-dev git htop \
    && echo "codespace ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/nopasswd \
    && chmod 0440 /etc/sudoers.d/nopasswd && useradd -m -s /bin/bash codespace

# Set compiler settings for R in root
RUN mkdir $HOME/.R/
RUN echo "CXX17FLAGS=-O3 -march=native -mtune=native -fPIC" >> $HOME/.R/Makevars
RUN echo "CXX17=g++" >> $HOME/.R/Makevars

# Set R settings in root
RUN echo "options(mc.cores =  4)" >> $HOME/.Rprofile
RUN echo "Sys.setenv(MAKEFLAGS = '-j4')" >> $HOME/.Rprofile

# This installs a lot of crap
RUN apt-get -y install r-cran-brms r-cran-remotes r-cran-shape libcurl4-openssl-dev \
    libfontconfig1-dev libnode-dev libxml2-dev 

# keep the below in case of emergency...
# libtbb-dev  libcairo2-dev libtiff-dev

# Install some xtra R packages
RUN install2.r --error \
    ggdag \
    languageserver \
    && rm -rf /tmp/downloaded_packages && rm -rf /root/.R*

RUN R -e "remotes::install_github('stan-dev/cmdstanr')"
RUN R -e "remotes::install_github('rmcelreath/rethinking')"
RUN R -e "remotes::install_github('nx10/httpgd')"

##### user stuff #####
USER codespace

# Set compiler settings for R
RUN mkdir $HOME/.R/
RUN echo "CXX17FLAGS=-O3 -march=native -mtune=native -fPIC" >> $HOME/.R/Makevars
RUN echo "CXX17=g++" >> $HOME/.R/Makevars

# Set R settings
RUN echo "options(mc.cores =  4)" >> $HOME/.Rprofile
RUN echo "Sys.setenv(MAKEFLAGS = '-j4')" >> $HOME/.Rprofile

# misc
RUN R -e "cmdstanr::install_cmdstan(cores = 4)"

# Use radian instead of R terminal
RUN pipx install radian
RUN pipx ensurepath
RUN echo "alias r='radian'" >> $HOME/.bashrc


CMD ["/bin/bash"]
WORKDIR /home/codespace