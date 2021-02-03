FROM ubuntu:bionic-20191029

ARG ssh_port=22
ARG file_system_port=8080
ARG time_zone="Asia/Taipei"

#================================================
# Customize sources for apt-get
#================================================
RUN  echo "deb http://archive.ubuntu.com/ubuntu bionic main universe\n" > /etc/apt/sources.list \
  && echo "deb http://archive.ubuntu.com/ubuntu bionic-updates main universe\n" >> /etc/apt/sources.list \
  && echo "deb http://security.ubuntu.com/ubuntu bionic-security main universe\n" >> /etc/apt/sources.list

# No interactive frontend during docker build
ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NONINTERACTIVE_SEEN=true

#========================
# Miscellaneous packages
# Includes minimal runtime used for executing non GUI Java programs
#========================
# 安裝說明 
# build-essential = C/C++
# iputils-ping = ping
# net-tools = ifconfig等
# psmisc = fuser 显示使用指定文件或者文件系统的进程的PID。
#          killall 杀死某个名字的进程，它向运行指定命令的所有进程发出信号。
#          pstree 树型显示当前运行的进程。
# apt-get -o Dpkg::Options::="--force-confmiss" install --reinstall netbase = socket io需要
RUN apt-get -qqy update \
  && apt-get -qqy --no-install-recommends install \
    bzip2 \
    ca-certificates \
    tzdata \
    sudo \
    unzip \
    wget \
    jq \
    curl \
    supervisor \
    gnupg2 \
    openssh-server \
    nano \
    iputils-ping \
    net-tools \
    psmisc \
    build-essential \
    python-minimal \
    python-pip \
    python-setuptools \
    git \
  && curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash - \
  && apt-get install -y nodejs \
  && rm -rf /var/lib/apt/lists/* /var/cache/apt/*

RUN apt-get install npm
RUN npm install -g apidoc
RUN npm install -g live-server
RUN pip install virtualenv
RUN apt-get update && apt-get -o Dpkg::Options::="--force-confmiss" install --reinstall netbase

#===================
# Timezone settings
# Possible alternative: https://github.com/docker/docker/issues/3359#issuecomment-32150214
#===================
ENV TZ ${time_zone}
RUN echo "${TZ}" > /etc/timezone \
  && dpkg-reconfigure --frontend noninteractive tzdata

#========================================
# Add normal user with passwordless sudo
#========================================
RUN useradd -ou 0 -g 0 test \
         --shell /bin/bash  \
         --create-home \
  && echo 'ALL ALL = (ALL) NOPASSWD: ALL' >> /etc/sudoers \
  && echo 'test:123123' | chpasswd
ENV HOME=/home/test

#=======================================
# Create shared / common bin directory
#=======================================
RUN  mkdir -p /opt/bin 


#======================================
# Add script
#======================================
COPY entry_point.sh start_ssh.sh /opt/bin/
RUN chmod +x /opt/bin/entry_point.sh

#======================================
# Add Supervisor configuration file
#======================================
COPY supervisord.conf /etc
COPY ssh_service.conf /etc/supervisor/conf.d/



#======================================
# Setup ssh login
#======================================
RUN mkdir /var/run/sshd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin  yes /' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# SSH change port
RUN sed -i "s/#Port 80/#Port $ssh_port/" /etc/ssh/sshd_config

RUN  mkdir -p /var/run/supervisor /var/log/supervisor \
  && chmod -R 777 /var/run/supervisor /var/log/supervisor /etc/passwd \
  && chgrp -R 0 /var/run/supervisor /var/log/supervisor \
  && chmod -R g=u /var/run/supervisor /var/log/supervisor

# ===================================================
# Run the following commands as non-privileged user
# ===================================================

EXPOSE ${ssh_port}
EXPOSE ${file_system_port}

CMD ["/opt/bin/entry_point.sh"]
