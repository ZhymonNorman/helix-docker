# --------------------------------------------------------------------------------
# Docker configuration for P4D
# --------------------------------------------------------------------------------

FROM rockylinux:8

LABEL vendor="AgentsOfSWARM"
LABEL maintainer="Simon Norman (@zhymon.bsky.social)"

# Update Rocky Linux, add Perforce Package Source, and import GPG key
RUN dnf update -y && \
    dnf install -y dnf-plugins-core && \
    dnf config-manager --add-repo http://package.perforce.com/yum/rhel/8/x86_64 && \
    rpm --import https://package.perforce.com/perforce.pubkey && \
    dnf install python3 -y && \
    dnf clean all && \
    dnf makecache

# --------------------------------------------------------------------------------
# Docker BUILD
# --------------------------------------------------------------------------------

# Create perforce user and install Perforce Server
# Note: You may need to adjust the version numbers for Rocky Linux compatibility
RUN dnf install -y helix-p4d-2024.2 helix-swarm-triggers-2024.5
# setup default python
RUN ls -ls /usr/bin/python* & ls -ls /usr/local/bin/python*
# Add external files
COPY files/restore.sh /usr/local/bin/restore.sh
COPY files/setup.sh /usr/local/bin/setup.sh
COPY files/init.sh /usr/local/bin/init.sh
COPY files/latest_checkpoint.sh /usr/local/bin/latest_checkpoint.sh

RUN \
  chmod +x /usr/local/bin/restore.sh && \
  chmod +x /usr/local/bin/setup.sh && \
  chmod +x /usr/local/bin/init.sh && \
  chmod +x /usr/local/bin/latest_checkpoint.sh

# --------------------------------------------------------------------------------
# Docker ENVIRONMENT
# --------------------------------------------------------------------------------

# Default Environment
ARG NAME=perforce-server
ARG P4NAME=master
ARG P4TCP=1666
ARG P4USER=admin
ARG P4PASSWD=pass12349ers
ARG P4CASE=-C0
ARG P4CHARSET=none

# Dynamic Environment
ENV NAME=$NAME \
  P4NAME=$P4NAME \
  P4TCP=$P4TCP \
  P4PORT=$P4TCP \
  P4USER=$P4USER \
  P4PASSWD=$P4PASSWD \
  P4CASE=$P4CASE \
  P4CHARSET=$P4CHARSET \
  JNL_PREFIX=$P4NAME

# Base Environment
ENV P4HOME=/p4

# Derived Environment
ENV P4ROOT=$P4HOME/root \
  P4DEPOTS=$P4HOME/depots \
  P4CKP=$P4HOME/checkpoints

# Expose Perforce; TCP port and volumes
EXPOSE $P4TCP
VOLUME $P4HOME

# --------------------------------------------------------------------------------
# Docker RUN
# --------------------------------------------------------------------------------

ENTRYPOINT \
  init.sh && \
  /usr/bin/tail -F $P4ROOT/logs/log

HEALTHCHECK \
  --interval=2m \
  --timeout=10s \
  CMD p4 info -s > /dev/null || exit 1
