ARG BUILD_FROM=ghcr.io/hassio-addons/debian-base:7.7.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Add env
ENV TERM="xterm-256color"

# Copy Python requirements file
COPY requirements.txt /tmp/

# Setup base
ARG BUILD_ARCH=amd64
ARG HA_CLI_VERSION="4.37.0"
ARG TTYD_VERSION="1.7.7"
# hadolint ignore=DL3003,DL3042

# Ensure the docker group exists with GID 102, this is important since that's the same as on the host (at the time of writing...)
RUN if getent group docker >/dev/null; then \
        current_gid=$(getent group docker | cut -d: -f3); \
        if [ "$current_gid" -ne 102 ]; then \
            groupmod -g 102 docker || { echo "Failed modifying docker group GID from ${current_gid} to 102"; exit 1; }; \
        fi; \
    else \
        groupadd -g 102 docker || { echo "Failed creating docker group with GID 102"; exit 1; }; \
    fi

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        docker.io \
        git \
        libffi-dev \
        libjson-c-dev \
        libssl-dev \
        libuv1-dev \
        python3-dev \
        zlib1g-dev \
        ack \
        alsa-utils \
        bash-completion \
        bind9-utils \
        bluez \
        colordiff \
        docker-compose \
        htop \
        libltdl7 \
        libstdc++6 \
        libuv1 \
        libxml2-utils \
        mariadb-client \
        mosh \
        nano \
        ncurses-bin \
        net-tools \
        nmap \
        openssh-client \
        openssh-server \
        openssl \
        pwgen \
        python3-pip \
        python3 \
        rsync \
        sqlite3 \
        sudo \
        tmux \
        vim \
        wget \
        zip \
        zsh \
  \
  && git clone --depth 1 \
        https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh \
    \
    && curl -L -s -o /usr/bin/ha \
        "https://github.com/home-assistant/cli/releases/download/${HA_CLI_VERSION}/ha_${BUILD_ARCH}" \
    \
    && chmod a+x /usr/bin/ha \
    && ha completion bash > /usr/share/bash-completion/completions/ha \
    \
    && sed -i -e "s#/bin/ash#/bin/zsh#" /etc/passwd \
  \
  && git clone --branch "v4.3.3" --depth=1 \
        https://github.com/warmcat/libwebsockets.git /tmp/libwebsockets \
    && mkdir -p /tmp/libwebsockets/build \
    && cd /tmp/libwebsockets/build \
    && cmake .. \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
        -DLWS_IPV6=ON \
        -DLWS_STATIC_PIC=ON \
        -DLWS_UNIX_SOCK=OFF \
        -DLWS_WITH_LIBUV=ON \
        -DLWS_WITH_SHARED=ON \
        -DLWS_WITHOUT_TESTAPPS=ON \
    && make \
    && make install \
  \
  && git clone --branch main --single-branch \
        https://github.com/tsl0922/ttyd.git /tmp/ttyd \
    && git -C /tmp/ttyd checkout "${TTYD_VERSION}" \
    \
    && mkdir -p /tmp/ttyd/build \
    && cd /tmp/ttyd/build \
    && cmake .. \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
    && make \
    && make install \
    \
    && ln -s /usr/bin/docker /usr/local/bin/.undocked \
    \
    && pip3 install --break-system-packages --no-cache-dir -r /tmp/requirements.txt \
    \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Copy root filesystem
COPY rootfs /

# Create privilege separation directory for ssh: /run/sshd
RUN mkdir /run/sshd

# Ensure right permissions
RUN \
    chmod 0750 /etc/sudo.conf \
    && chmod 0750 -R /etc/sudoers.d \
    && chmod 0640 /etc/sudoers.d

# Allow users sudo access without password
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
