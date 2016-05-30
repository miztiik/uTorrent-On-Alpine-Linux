##################################################################################
## 
## VERSION		:	0.0.2
## DATE			:	30May2016
##                				
## USAGE		:	Dockerfile to build a super lightweight container to run uTorrent in Linux with mounted volumes for data storage
## Ref [1]		:	https://github.com/miztiik/cloudera-On-Docker/blob/master/docker-image-build/uTor/centos/Dockerfile
## Ref [2]		:	https://github.com/jeanblanchard/docker-alpine-glibc/blob/master/Dockerfile
##################################################################################

FROM alpine:latest
MAINTAINER Mystique

# Here we install GNU libc (aka glibc) and set C.UTF-8 locale as default.
# - Download & Install glibc (needed for utserver)
# - Setup sym links, remove un-used packages, clean cachce, Setup up entry points

RUN ALPINE_GLIBC_BASE_URL="https://github.com/andyshinn/alpine-pkg-glibc/releases/download" && \
    ALPINE_GLIBC_PACKAGE_VERSION="2.23-r1" && \
    ALPINE_GLIBC_BASE_PACKAGE_FILENAME="glibc-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_BIN_PACKAGE_FILENAME="glibc-bin-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    ALPINE_GLIBC_I18N_PACKAGE_FILENAME="glibc-i18n-$ALPINE_GLIBC_PACKAGE_VERSION.apk" && \
    apk add --update --no-cache --virtual=build-dependencies wget libgcc openssl ca-certificates && \
    wget \
        "https://raw.githubusercontent.com/andyshinn/alpine-pkg-glibc/master/andyshinn.rsa.pub" \
        -O "/etc/apk/keys/andyshinn.rsa.pub" && \
    wget \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BASE_URL/$ALPINE_GLIBC_PACKAGE_VERSION/$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    apk add --no-cache \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
    rm "/etc/apk/keys/andyshinn.rsa.pub" && \
    /usr/glibc-compat/bin/localedef --force --inputfile POSIX --charmap UTF-8 C.UTF-8 || true && \
    echo "export LANG=C.UTF-8" > /etc/profile.d/locale.sh && \
    \
    apk del glibc-i18n && \
    \
    apk del build-dependencies && \
    rm \
        "$ALPINE_GLIBC_BASE_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_BIN_PACKAGE_FILENAME" \
        "$ALPINE_GLIBC_I18N_PACKAGE_FILENAME" && \
    \
    echo 'hosts: files mdns4_minimal [NOTFOUND=return] dns mdns4' >> /etc/nsswitch.conf && \
    apk del wget && \
    rm -rf /var/cache/apk/* && \
    mkdir -p /opt/utorrent;
		
# Install necessary packages & Create sym link for libssl because the CentOS 6 has newer version than what uTorrent requires
RUN ln -s /usr/lib/libssl.so.1.0.0 /usr/lib/libssl.so.0.9.8 && \
    ln -s /usr/lib/libcrypto.so.1.0.0 /usr/lib/libcrypto.so.0.9.8

ENV LANG=C.UTF-8

# Expose external connectivity port & the web interface
EXPOSE 2891 8085

# Setting up the entry point to allow for graceful exit when the container is stopped
ENTRYPOINT ["/opt/utorrent/utserver" , "-settingspath" , "/opt/utorrent/" , "-configfile"]
CMD ["/opt/utorrent/utserver.conf", "&"]
