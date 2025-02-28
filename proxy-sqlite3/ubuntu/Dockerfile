FROM ubuntu:focal

LABEL org.opencontainers.image.title="Zabbix proxy (SQLite3)" \
      org.opencontainers.image.authors="Alexey Pustovalov <alexey.pustovalov@zabbix.com>" \
      org.opencontainers.image.vendor="Zabbix LLC" \
      org.opencontainers.image.url="https://zabbix.com/" \
      org.opencontainers.image.description="Zabbix proxy with SQLite3 database support" \
      org.opencontainers.image.licenses="GPL v2.0"

STOPSIGNAL SIGTERM

RUN set -eux && \
    ARCH_SUFFIX="$(arch)"; \
    case "$ARCH_SUFFIX" in \
        i686) export ARCH_SUFFIX='i386' ;; \
        x86_64) [ -f /lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 ] && export ARCH_SUFFIX='amd64' || export ARCH_SUFFIX='i386' ;; \
        aarch64) export ARCH_SUFFIX='arm64' ;; \
        armv7l) export ARCH_SUFFIX='armhf' ;; \
        ppc64el|ppc64le) export ARCH_SUFFIX='ppc64le' ;; \
        s390x) export ARCH_SUFFIX='s390x' ;; \
        *) echo "Unknown ARCH_SUFFIX=${ARCH_SUFFIX-}"; exit 1 ;; \
    esac; \
    echo "#!/bin/sh\nexit 101" > /usr/sbin/policy-rc.d && \
    groupadd --system --gid 1995 zabbix && \
    useradd \
            --system --comment "Zabbix monitoring system" \
            -g zabbix -G root \
            --uid 1997 \
            --shell /sbin/nologin \
            --home-dir /var/lib/zabbix/ \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /var/lib/zabbix && \
    mkdir -p /var/lib/zabbix/enc && \
    mkdir -p /var/lib/zabbix/mibs && \
    mkdir -p /var/lib/zabbix/modules && \
    mkdir -p /var/lib/zabbix/snmptraps && \
    mkdir -p /var/lib/zabbix/ssh_keys && \
    mkdir -p /var/lib/zabbix/ssl && \
    mkdir -p /var/lib/zabbix/ssl/certs && \
    mkdir -p /var/lib/zabbix/ssl/keys && \
    mkdir -p /var/lib/zabbix/ssl/ssl_ca && \
    mkdir -p /usr/lib/zabbix/externalscripts && \
    apt-get -y update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
            tini \
            ca-certificates \
            fping \
            libcurl4 \
            libevent-2.1 \
            libopenipmi0 \
            libpcre3 \
            libsnmp35 \
            libsqlite3-0 \
            libssh-4 \
            libssl1.1 \
            libxml2 \
            snmp-mibs-downloader \
            unixodbc && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

ARG MAJOR_VERSION=5.0
ARG ZBX_VERSION=${MAJOR_VERSION}.7
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm MIBDIRS=/var/lib/snmp/mibs/ietf:/var/lib/snmp/mibs/iana:/usr/share/snmp/mibs:/var/lib/zabbix/mibs MIBS=+ALL \
    ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.opencontainers.image.documentation="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.opencontainers.image.version="${ZBX_VERSION}" \
      org.opencontainers.image.source="${ZBX_SOURCES}"

RUN set -eux && \
    apt-get -y update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y --no-install-recommends install \
            autoconf \
            automake \
            gcc \
            libc6-dev \
            libcurl4-openssl-dev \
            libevent-dev \
            libldap2-dev \
            libopenipmi-dev \
            libpcre3-dev \
            libsnmp-dev \
            libsqlite3-dev \
            libssh-dev \
            libxml2-dev \
            make \
            pkg-config \
            git \
            unixodbc-dev && \
    cd /tmp/ && \
    git clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    cd /tmp/zabbix-${ZBX_VERSION} && \
    zabbix_revision=`git rev-parse --short HEAD` && \
    sed -i "s/{ZABBIX_REVISION}/$zabbix_revision/g" include/version.h && \
    ./bootstrap.sh && \
    export CFLAGS="-fPIC -pie -Wl,-z,relro -Wl,-z,now" && \
    ./configure \
            --datadir=/usr/lib \
            --libdir=/usr/lib/zabbix \
            --sysconfdir=/etc/zabbix \
            --prefix=/usr \
            --enable-agent \
            --enable-proxy \
            --with-sqlite3 \
            --with-ldap \
            --with-libcurl \
            --with-libxml2 \
            --with-net-snmp \
            --with-openipmi \
            --with-openssl \
            --with-ssh \
            --with-unixodbc \
            --enable-ipv6 \
            --silent && \
    make -j"$(nproc)" -s dbschema && \
    make -j"$(nproc)" -s && \
    cp src/zabbix_proxy/zabbix_proxy /usr/sbin/zabbix_proxy && \
    cp src/zabbix_get/zabbix_get /usr/bin/zabbix_get && \
    cp src/zabbix_sender/zabbix_sender /usr/bin/zabbix_sender && \
    cp conf/zabbix_proxy.conf /etc/zabbix/zabbix_proxy.conf && \
    cd /tmp/ && \
    rm -rf /tmp/zabbix-${ZBX_VERSION}/ && \
    chown --quiet -R zabbix:root /etc/zabbix/ /var/lib/zabbix/ && \
    chgrp -R 0 /etc/zabbix/ /var/lib/zabbix/ && \
    chmod -R g=u /etc/zabbix/ /var/lib/zabbix/ && \
    DEBIAN_FRONTEND=noninteractive apt-get -y purge \
            autoconf \
            automake \
            gcc \
            libc6-dev \
            libcurl4-openssl-dev \
            libevent-dev \
            libldap2-dev \
            libopenipmi-dev \
            libpcre3-dev \
            libsnmp-dev \
            libsqlite3-dev \
            libssh-dev \
            libxml2-dev \
            make \
            pkg-config \
            git \
            unixodbc-dev && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 10051/TCP
            
WORKDIR /var/lib/zabbix
            
VOLUME ["/var/lib/zabbix/snmptraps"]
            
COPY ["docker-entrypoint.sh", "/usr/bin/"]
            
ENTRYPOINT ["/usr/bin/tini", "--", "/usr/bin/docker-entrypoint.sh"]
            
USER 1997
    
CMD ["/usr/sbin/zabbix_proxy", "--foreground", "-c", "/etc/zabbix/zabbix_proxy.conf"]
