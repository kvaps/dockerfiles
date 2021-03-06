FROM alpine:3.4
MAINTAINER Wonderfall <wonderfall@schrodinger.io>

ARG MPFR_VERSION=3.1.4
ARG MPC_VERSION=1.0.3

ARG GPG_MPFR="07F3 DBBE CC1A 3960 5078  094D 980C 1976 98C3 739D"
ARG GPG_MPC="AD17 A21E F8AE D8F1 CC02  DBD9 F7D5 C9BF 765C 61E3"
ARG SHA1_MPC="b8be66396c726fdc36ebb0f692ed8a8cca3bcc66"

ENV UID=1000 \
    GID=1000 \
    HOSTNAME=svr04 \
    DL_LIMIT=10485760 \
    FACING_IP=9.9.9.9 \
    JSON_LOG=False

RUN echo "@commuedge https://nl.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
 && BUILD_DEPS=" \
    build-base \
    libtool \
    libffi-dev \
    openssl-dev \
    python-dev \
    gmp-dev \
    py-pip \
    tar \
    gnupg" \
 && apk -U add \
    ${BUILD_DEPS} \
    libffi \
    gmp \
    openssl \
    python \
    py-setuptools \
    openssh-client \
    tini@commuedge \
    su-exec \
 && cd /tmp && wget -q http://www.mpfr.org/mpfr-current/mpfr-${MPFR_VERSION}.tar.gz \
 && echo "Verifying authenticity of mpfr-${MPFR_VERSION}.tar.gz..." \
 && wget -q http://www.mpfr.org/mpfr-current/mpfr-${MPFR_VERSION}.tar.gz.asc \
 && gpg --recv-keys 98C3739D \
 && FINGERPRINT="$(LANG=C gpg --verify mpfr-${MPFR_VERSION}.tar.gz.asc mpfr-${MPFR_VERSION}.tar.gz 2>&1 \
  | sed -n "s#Primary key fingerprint: \(.*\)#\1#p")" \
 && if [ -z "${FINGERPRINT}" ]; then echo "Warning! Invalid GPG signature!" && exit 1; fi \
 && if [ "${FINGERPRINT}" != "${GPG_MPFR}" ]; then echo "Warning! Wrong GPG fingerprint!" && exit 1; fi \
 && echo "All seems good, now unpacking mpfr-${MPFR_VERSION}.tar.gz..." \
 && tar xzf mpfr-${MPFR_VERSION}.tar.gz && cd mpfr-${MPFR_VERSION} \
 && ./configure && make && make install \
 && cd /tmp && wget -q ftp://ftp.gnu.org/gnu/mpc/mpc-${MPC_VERSION}.tar.gz \
 && echo "Verifying both integrity and authenticity of mpc-${MPC_VERSION}.tar.gz..." \
 && CHECKSUM=$(sha1sum mpc-${MPC_VERSION}.tar.gz | awk '{print $1}') \
 && if [ "${CHECKSUM}" != "${SHA1_MPC}" ]; then echo "Warning! Checksum does not match!" && exit 1; fi \
 && wget -q ftp://ftp.gnu.org/gnu/mpc/mpc-${MPC_VERSION}.tar.gz.sig \
 && gpg --recv-keys 0xF7D5C9BF765C61E3 \
 && FINGERPRINT="$(LANG=C gpg --verify mpc-${MPC_VERSION}.tar.gz.sig mpc-${MPC_VERSION}.tar.gz 2>&1 \
  | sed -n "s#Primary key fingerprint: \(.*\)#\1#p")" \
 && if [ -z "${FINGERPRINT}" ]; then echo "Warning! Invalid GPG signature!" && exit 1; fi \
 && if [ "${FINGERPRINT}" != "${GPG_MPC}" ]; then echo "Warning! Wrong GPG fingerprint!" && exit 1; fi \
 && echo "All seems good, now unpacking mpc-${MPC_VERSION}.tar.gz..." \
 && tar xzf mpc-${MPC_VERSION}.tar.gz && cd mpc-${MPC_VERSION} \
 && ./configure --with-mpfr-lib=/usr/local/lib --with-mpfr-include=/usr/local/include \
 && make && make install \
 && pip install --no-cache \
    pyasn1 \
    service_identity \
    pyOpenSSL \
    twisted \
    pycrypto \
    cryptography \
    gmpy2 \
 && mkdir /cowrie && cd /cowrie \
 && wget -qO- https://github.com/micheloosterhof/cowrie/archive/master.tar.gz | tar xz --strip 1 \
 && mv cowrie.cfg.dist cowrie.cfg \
 && apk del ${BUILD_DEPS} \
 && rm -rf /var/cache/apk/* /tmp/* /root/.gnupg

COPY run.sh /usr/local/bin/run.sh

RUN chmod +x /usr/local/bin/run.sh

VOLUME /cowrie/log /cowrie/dl /cowrie/custom
EXPOSE 2222
CMD ["/sbin/tini","--","run.sh"]
