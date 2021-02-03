FROM vm_base

COPY start_live-server.sh /opt/bin/
COPY live-server-service.conf /etc/supervisor/conf.d/

#======================================
# Add ti-sdk
#======================================
COPY ti-sdk-03.02.zip /opt

#======================================
# Settup workdir
#======================================
WORKDIR /home/test

RUN mkdir /home/test/project
RUN unzip /opt/ti-sdk-03.02.zip

VOLUME  ["/home/test/project"]