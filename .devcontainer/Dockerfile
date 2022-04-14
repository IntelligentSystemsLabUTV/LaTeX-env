# LaTeX environment.
#
# Alessandro Tenaglia <alessandro.tenaglia42@gmail.com>
# Roberto Masocco <robmasocco@gmail.com>
#
# April 5, 2022

FROM ubuntu:20.04

ARG USER_UID=1000

ENV DEBIAN_FRONTEND=noninteractive

# Install basic utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-utils \
    ca-certificates \
    cpanminus \
    curl \
    git \
    gnupg \
    iproute2 \
    less \
    openssh-client \
    openssl \
    python3 \
    python3-pip \
    procps \
    software-properties-common \
    sudo \
    wget && \
    add-apt-repository universe && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*

# Install language and locales
RUN apt-get update && apt-get install -y --no-install-recommends \
    locales && \
    locale-gen en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*
ENV LANG=en_US.UTF-8

# Set up UTC timezone
RUN ln -fs /usr/share/zoneinfo/UTC /etc/localtime && \
    apt-get update && \
    apt-get install -y --no-install-recommends tzdata && \
    dpkg-reconfigure --frontend noninteractive tzdata && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*

# Install Zsh
RUN apt-get update && apt-get install -y --no-install-recommends \
    zsh \
    zsh-doc \
    chroma && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*

# Install TeX Live and various TeX utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    biber \
    cm-super \
    ghostscript \
    inkscape \
    latexmk \
    pandoc \
    texlive \
    texlive-bibtex-extra \
    texlive-extra-utils \
    texlive-fonts-extra \
    texlive-font-utils \
    texlive-latex-extra \
    texlive-latex-recommended \
    texlive-plain-generic \
    texlive-science && \
    rm -rf /var/lib/apt/lists/*

# Install latexindent dependencies (Perl modules)
RUN cpanm -n -q Log::Log4perl && \
    cpanm -n -q XString && \
    cpanm -n -q Log::Dispatch::File && \
    cpanm -n -q YAML::Tiny && \
    cpanm -n -q File::HomeDir && \
    cpanm -n -q Unicode::GCString

# Cleanup
RUN apt-get autoremove -y && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*/apt/lists/*

# Create a non-root sudo user with Zsh as shell (psw: latexdev)
RUN useradd -r -m -s /bin/bash -u ${USER_UID} -g root -G sudo -p "$(openssl passwd -1 latexdev)" latex && \
    echo latex ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/latex && \
    chmod 0440 /etc/sudoers.d/latex
ENV HOME=/home/latex
RUN chsh -s /usr/bin/zsh latex

# Create workspace directory: host workspaces will be mounted here
RUN mkdir ${HOME}/workspace && \
    chown latex ${HOME}/workspace

# Create shell history file
RUN mkdir ${HOME}/zsh_history && \
    chown latex ${HOME}/zsh_history

# Create SSH directory for user
RUN mkdir ${HOME}/.ssh

# Switch to internal user
USER latex
WORKDIR ${HOME}

# Copy user configuration files
COPY --chown=latex ./config/.aliases.sh ./
COPY --chown=latex ./config/.bashrc ./
COPY --chown=latex ./config/.nanorc ./

# Configure Zsh for internal user
ENV ZSH=${HOME}/.oh-my-zsh
ENV ZSH_CUSTOM=${ZSH}/custom
ENV ZSH_PLUGINS=${ZSH_CUSTOM}/plugins
ENV ZSH_THEMES=${ZSH_CUSTOM}/themes
RUN wget -qO- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh | zsh || true
RUN git clone --single-branch --branch 'master' --depth 1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_PLUGINS}/zsh-syntax-highlighting \
    && git clone --single-branch --branch 'master' --depth 1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_PLUGINS}/zsh-autosuggestions \
    && git clone --single-branch --depth 1 https://github.com/romkatv/powerlevel10k.git ${ZSH_THEMES}/powerlevel10k
COPY --chown=latex ./config/.p10k.zsh ./
COPY --chown=latex ./config/.zshrc ./

# Start shell in container
CMD ["zsh"]
