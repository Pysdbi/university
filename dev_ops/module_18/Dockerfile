FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    openssh-server \
    sudo \
    curl \
    && rm -rf /var/lib/apt/lists/*
RUN useradd -m -s /bin/bash sshuser && \
    echo 'sshuser:password123' | chpasswd && \
    usermod -aG sudo sshuser
RUN mkdir /var/run/sshd && \
    sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
RUN mkdir -p /home/sshuser/.ssh && \
    chown sshuser:sshuser /home/sshuser/.ssh && \
    chmod 700 /home/sshuser/.ssh

EXPOSE 2222
CMD ["/usr/sbin/sshd", "-D"]

