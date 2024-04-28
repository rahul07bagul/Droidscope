# Start with Ubuntu 14.04 base image
FROM ubuntu:14.04

# Install dependencies for building Python
RUN apt-get update && apt-get install -y \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    wget \
    curl \
    llvm \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    git

# Create a directory for the repo tool
RUN mkdir -p /root/bin

RUN cd /home

# Download the repo tool and make it executable
RUN curl https://storage.googleapis.com/git-repo-downloads/repo > /home/repo

RUN mv /home/repo /root/bin/

RUN chmod a+x /root/bin/repo

# Add /root/bin to PATH permanently in .bashrc
RUN echo 'export PATH="/root/bin:$PATH"' >> /root/.bashrc

# Ensure commands below this line use the new PATH, in case .bashrc is not sourced automatically
ENV PATH="/root/bin:${PATH}"

# Download, extract and compile Python
RUN wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz \
    && tar -xvf Python-3.6.3.tgz \
    && cd Python-3.6.3 \
    && ./configure --enable-optimizations \
    && make install

# Set python3.6 as the default python3
RUN update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.6 1 \
    && update-alternatives --set python3 /usr/local/bin/python3.6

# Install Java
RUN apt-get install -y openjdk-7-jdk

# Set the working directory (optional)
WORKDIR /home

# By default, run a bash shell when the container starts
CMD ["/bin/bash"]
