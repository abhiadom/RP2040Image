FROM ubuntu:latest

ENV DEBIAN_FRONTEND=noninteractive

# Update package lists and install necessary tools
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    curl \
    wget \
    unzip \
    vim \
    nano \
    openocd \
    libusb-1.0-0-dev \
    libnewlib-arm-none-eabi \
    gdb-multiarch \
    gcc-arm-none-eabi \
    python3 \
    python3-pip \
    tmux \
    openssh-server \
    pkg-config \
    ca-certificates

# Install code-server
RUN curl -fsSL https://code-server.dev/install.sh | sh

# Install the Pico SDK
RUN mkdir -p /opt/pico-sdk
WORKDIR /opt/pico-sdk
RUN git clone https://github.com/raspberrypi/pico-sdk.git .
RUN git submodule update --init

# Set environment variables
ENV PICO_SDK_PATH=/opt/pico-sdk
ENV PATH=$PICO_SDK_PATH/bin:$PATH

# Clone and build picotool
RUN git clone https://github.com/raspberrypi/picotool.git /opt/picotool
WORKDIR /opt/picotool

# Build picotool
RUN mkdir build
WORKDIR /opt/picotool/build
RUN cmake .. && make

# Copy the built binary to /usr/local/bin
RUN cp picotool /usr/local/bin/

# Create a known password for root user
RUN echo 'root:adom@123' | chpasswd

# Configure SSH
RUN mkdir -p /root/.ssh
COPY id_ed25519.pub /root/.ssh/authorized_keys
RUN chmod 600 /root/.ssh/authorized_keys
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Create config directory for code-server and disable authentication
RUN mkdir -p /root/.config/code-server
RUN echo "bind-addr: 0.0.0.0:8080\n" > /root/.config/code-server/config.yaml
RUN echo "auth: none\n" >> /root/.config/code-server/config.yaml

# Copy settings.json and tasks.json
COPY .vscode/settings.json /root/.local/share/code-server/User/settings.json
COPY .vscode/tasks.json /root/.local/share/code-server/User/tasks.json

# Install open-vsx extension manager
RUN wget https://github.com/eclipse/openvsx/releases/download/v0.1.0/openvsx-linux-x64.tar.gz && \
    tar -xzvf openvsx-linux-x64.tar.gz && \
    mv openvsx /usr/local/bin/openvsx

# Add entrypoint script to install extensions on startup
RUN echo '#!/bin/sh\n\
code-server --extensions-dir /root/.local/share/code-server/extensions \n\
openvsx install golang.go@0.30.0\n\
openvsx install ms-python.python@2022.8.0\n\
openvsx install c-cpp-compile-run@0.7.0\n\
openvsx install activitusbar@0.5.0\n\
code-server "$@"' > /usr/local/bin/code-server-entrypoint.sh

RUN chmod +x /usr/local/bin/code-server-entrypoint.sh

# Expose Ports
EXPOSE 22
EXPOSE 8080

CMD ["/bin/sh", "-c", "/usr/sbin/sshd -D & /usr/local/bin/code-server-entrypoint.sh --bind-addr 0.0.0.0:8080 /workspace"]