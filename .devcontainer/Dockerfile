FROM python:3.12-slim-bookworm

ARG PUID=${PUID:-1000}
ARG PGID=${PGID:-1000}

ARG USER=anvil
ARG TZ=America/New_York
ENV USER=${USER}
ENV TZ=${TZ}

ENV LANGUAGE=en_US.UTF-8
ENV LANG=en_US.UTF-8

ENV DEBIAN_FRONTEND=noninteractive
ENV DEBCONF_NONINTERACTIVE_SEEN=true

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    cmake \
    curl \
    git \
    locales \

    zip unzip bzip2 \
    libjpeg-dev libpng-dev libpq-dev \
    imagemagick gnuplot gnuplot-x11 libssl-dev libzmq3-dev graphviz \

    make \
    nano \
    ninja-build \
    openssl \
    openssh-client \
    rsync \
    sshpass \
    symlinks \
    tar \
    tldr \
    sudo \
    tree \
    tzdata \
    zsh \
    zsh-autosuggestions && \
    apt-get autoremove -fy && \
    apt-get clean && \
    apt-get autoclean -y && \
    rm -rf /var/lib/apt/lists/*

RUN sed -i 's/UID_MAX .*/UID_MAX    100000/' /etc/login.defs && \
    groupadd --gid ${PGID} ${USER} && \
    useradd --uid ${PUID} --gid ${PGID} -s /bin/zsh -m ${USER} && \
    echo ${USER} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USER} && \
    chmod 0440 /etc/sudoers.d/${USER} 

# Use a virtual environment for Python (to avoid PEP668 "externally managed environment" error).
ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Install Python Packages (base)
RUN pip3 config --user set global.disable-pip-version-check true && \
    pip3 config --user set global.progress_bar off && \
    pip3 config --user set global.no-cache-dir true && \
    pip3 install --upgrade pip && \
    pip3 install --upgrade setuptools && \
    pip3 install --upgrade wheel

# Install Python Packages
# Note: using multiple pip install runs is intentional, else pip has a problem resolving the dependencies
RUN pip3 install ipykernel docutils jupyter notebook jupyterhub && \
    pip3 install watermark pyyaml pylint h5py tensorflow && \
    pip3 install --prefer-binary \
      matplotlib seaborn plotly graphviz opencv-python-headless imutils \
      keras --no-deps && \
    pip3 install --prefer-binary \
      numpy pandas pandas-datareader bottleneck \
      scipy scikit-learn duckdb sqlalchemy pyautogui requests_cache \
      yfinance alpha_vantage nasdaq-data-link finnhub-python \
      financetoolkit financedatabase statsmodels

RUN pip3 install chromadb sentence_transformers 

RUN pip3 install \
      bs4 \
      docker \
      feedparser \
      flask \
      flask-caching \
      langchain \
      langchain-community \
      lxml 

USER ${USER}
RUN git clone https://github.com/${USER}/dotfiles.git ~/.dotfiles && \
    ~/.dotfiles/install

# Install ipykernel
RUN /opt/venv/bin/python -m ipykernel install --user --name="jupyter_devbox" --display-name="jupyter_devbox_python3_venv"