FROM ubuntu

ENV container docker
ENV SHELL /bin/bash

# Don't start any optional services except for the few we need.
RUN find /etc/systemd/system \
    /lib/systemd/system \
    -path '*.wants/*' \
    -not -name '*journald*' \
    -not -name '*systemd-tmpfiles*' \
    -not -name '*systemd-user-sessions*' \
    -exec rm \{} \;

RUN apt-get update
RUN apt-get install -y vim sudo curl glibc groff less unzip dbus systemd software-properties-common vim sudo curl wget unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN echo 'APT::Periodic::Update-Package-Lists "0";' > /etc/apt/apt.conf.d/20auto-upgrades
RUN echo 'APT::Periodic::Unattended-Upgrade "0";' >> /etc/apt/apt.conf.d/20auto-upgrades

RUN systemctl set-default multi-user.target
RUN systemctl mask dev-hugepages.mount sys-fs-fuse-connections.mount

RUN  mkdir -p /build
COPY ./build /build

RUN useradd -m docker && echo "docker:docker" | chpasswd && \
    adduser docker sudo && \
    echo "docker ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
    chmod +x /build/*.sh && \
    for script in $(ls /build/*.sh | sort ); do FROMDOCKER=1 $script; done

STOPSIGNAL SIGRTMIN+3


RUN useradd -m -s /bin/bash -G sudo coder
USER coder

COPY myCA.crt /usr/local/share/ca-certificates
RUN sudo update-ca-certificates

# Let's add a fancy prompt!
RUN echo "PS1='🐳 \[\033[1;36m\]\h \[\033[1;34m\]\W\[\033[0;35m\] \[\033[1;36m\]# \[\033[0m\]'" > ~/.bashrc

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-$(arch).zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    sudo ./aws/install

# Workaround for docker/docker#27202, technique based on comments from docker/docker#9212
CMD ["/bin/bash", "-c", "exec /sbin/init --log-target=journal 3>&1"]
