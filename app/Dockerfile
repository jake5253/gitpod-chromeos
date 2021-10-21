FROM ubuntu:bionic

### base ###
ARG DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install --yes --no-install-recommends \
    git \
    gitk \
    git-gui \
    curl \
    xz-utils \
    python3-pkg-resources \
    python3-virtualenv \
    python3-oauth2client\
    gpg-agent \
    build-essential \
    locales \
    software-properties-common \
    sudo \
    curl \
    openssh-client \
    lvm2 \
    thin-provisioning-tools \
    && locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8

### Git ###
RUN add-apt-repository -y ppa:git-core/ppa \
    && apt install --yes git git-lfs

### Gitpod user ###
# '-l': see https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user
RUN useradd -l -u 33333 -G sudo -md /home/gitpod -s /bin/bash -p gitpod gitpod \
    # passwordless sudo for users in the 'sudo' group
    && sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers
ENV HOME=/home/gitpod
WORKDIR $HOME
# custom Bash prompt
#RUN { echo && echo "PS1='\[\033[01;32m\]\u\[\033[00m\] \[\033[01;34m\]\w\[\033[00m\]\$(__git_ps1 \" (%s)\") $ '" ; } >> .bashrc

# make sudo more permissive
RUN cat << EOF | tee -a /etc/sudoers \
Defaults !tty_tickets \
Defaults timestamp_timeout=300 \
EOF

### Gitpod user (2) ###
USER gitpod
# use sudo so that user does not get sudo usage info on (the first) login
RUN sudo echo "Running 'sudo' for Gitpod: success" && \
    # create .bashrc.d folder and source it in the bashrc
    mkdir -p ${HOME}/.bashrc.d && \
    (echo; echo "for i in \$(ls -A \$HOME/.bashrc.d/); do source \$HOME/.bashrc.d/\$i; done"; echo) >> /home/gitpod/.bashrc

# configure git-lfs
RUN sudo git lfs install --system

# install pyenv
#RUN unset PYENV_ROOT && \
#    curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
#ENV PATH="${HOME}/.pyenv/bin:${PATH}"
#RUN pyenv install 3.9.7 && \
#    pyenv global 3.9.7

# install depot_tools
RUN git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git /home/gitpod/depot_tools
ENV PATH=${HOME}/depot_tools:${PATH}
RUN git config --global user.name "username" && \
    git config --global user.email "email@example.com" && \
    echo "\numask 022" | tee -a ${HOME}/.bashrc

ENV CHROMIUM_SRC_DIR=${HOME}/chromiumos
RUN echo 'Creating directories' && \
    mkdir -p ${CHROMIUM_SRC_DIR} && \ 
    cd ${CHROMIUM_SRC_DIR} && \
    echo "Initializing repo" && \
        repo init -u https://chromium.googlesource.com/chromiumos/manifest.git --submodules --depth=1 -b stable -g minilayout && \
    echo "Syncronizing" && \
        repo sync -j$(nproc)
        
ENTRYPOINT [ "/bin/bash" ]


FROM gitpod/workspace-full-vnc:latest

USER root

# Install Chromium build dependencies
RUN apt update \
  && apt install --yes --no-install-recommends \
    git \
    gitk \
    git-gui \
    curl \
    xz-utils \
    python3-pkg-resources \
    python3-virtualenv \
    python3-oauth2client\
    gpg-agent \
    build-essential \
    locales \
    software-properties-common \
    sudo \
    curl \
    openssh-client \
    lvm2 \
    thin-provisioning-tools \
  && locale-gen en_US.UTF-8 \
  && rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.UTF-8

USER gitpod

# Install Chromium's depot_tools.
RUN git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git /home/gitpod/depot_tools
ENV PATH $PATH:/home/gitpod/depot_tools
RUN echo "\n# Add Chromium's depot_tools to the PATH." >> /home/gitpod/.bashrc \
 && echo "export PATH=\"\$PATH:/home/gitpod/depot_tools\"" >> /home/gitpod/.bashrc

# Enable bash completion for git cl.
RUN echo "\n# The next line enables bash completion for git cl." >> /home/gitpod/.bashrc \
 && echo "if [ -f \"/home/gitpod/depot_tools/git_cl_completion.sh\" ]; then" >> /home/gitpod/.bashrc \
 && echo "  . \"/home/gitpod/depot_tools/git_cl_completion.sh\"" >> /home/gitpod/.bashrc \
 && echo "fi" >> /home/gitpod/.bashrc

# Set up git
RUN git config --global user.name "/dev/null" \
  && git config --global user.email "dev@null.com" 

# Give back control.
USER root
