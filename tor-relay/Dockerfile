FROM alpine:edge
LABEL maintainer="Jonathan Elias Caicedo <Jonathan@jcaicedo.com>"

# Based on https://github.com/jfrazelle/dockerfiles/tree/master/tor-relay
ARG TOR_VERSION=0.3.0.7
ENV GPG_KEYS \
  6AFEE6D49E92B601 
  
# Note: Tor is only in testing repo -> http://pkgs.alpinelinux.org/packages?package=emacs&repo=all&arch=x86_64
RUN build_pkgs="build-base libevent-dev libressl-dev zlib-dev curl gnupg" \
    runtime_pkgs="musl libressl2.5-libcrypto libevent libressl2.5-libssl zlib" \
    && apk --no-cache add ${build_pkgs} ${runtime_pkgs} \
    && for key in $GPG_KEYS; do \
		gpg --keyserver pgp.mit.edu --recv-keys "$key"; \
	done \
    && mkdir /tmp/tor \
    && cd /tmp/tor \
    && curl -fSL https://www.torproject.org/dist/tor-${TOR_VERSION}.tar.gz -o tor.tar.gz \
    && curl -fSL https://dist.torproject.org/tor-${TOR_VERSION}.tar.gz.asc -o tor.tar.gz.asc \
    && gpg --batch --verify tor.tar.gz.asc tor.tar.gz \
    && tar -zxf tor.tar.gz \ 
    && cd tor-${TOR_VERSION} \
    && ./configure \
       --prefix=/usr \
       --sysconfdir=/etc \
       --mandir=/usr/share/man \
       --infodir=/usr/share/info \
       --localstatedir=/var \
    && make install -j $(getconf _NPROCESSORS_ONLN) \
    && adduser -D tor \
    && strip -s /usr/bin/tor \
    && apk del ${build_pkgs} \
    && rm -rf /tmp/ /root/.gnupg/ 

# default port to used for incoming Tor connections
# can be changed by changing 'ORPort' in torrc
EXPOSE 9001

# copy in our torrc files
COPY torrc.bridge /etc/tor/torrc.bridge
COPY torrc.middle /etc/tor/torrc.middle
COPY torrc.exit /etc/tor/torrc.exit

# make sure files are owned by tor user
RUN chown -R tor /etc/tor

# Add launcher
COPY ./config.sh /etc/tor/config.sh

# Start Tor
RUN chmod +x /etc/tor/config.sh

RUN mkdir -p /home/tor
VOLUME /home/tor/.tor
RUN chown -R tor /home/tor/

CMD su - tor -c "/usr/bin/tor -f /etc/tor/torrc.middle"
