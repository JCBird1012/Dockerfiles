FROM alpine:edge
LABEL maintainer="Jonathan Elias Caicedo [Jonathan@jcaicedo.com]"

ARG NGINX_VERSION=1.13.0
ARG LIBRESSL_VERSION=2.5.4
ENV GPG_KEYS \
  B0F4253373F8F6F510D42178520A9993A1C052F8 \
  A1EB079B8D3EB92B4EBD3139663AF51BD5E4D8D5

RUN \
  build_pkgs="build-base git linux-headers pcre-dev curl zlib-dev gnupg" \
  && runtime_pkgs="ca-certificates pcre zlib" \
  && apk --no-cache add ${build_pkgs} ${runtime_pkgs} \
  && for key in $GPG_KEYS; do \
		gpg --keyserver pgp.mit.edu --recv-keys "$key"; \
	done \
  && mkdir /tmp/libressl \
  && cd /tmp/libressl \
  && curl -fSL http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-${LIBRESSL_VERSION}.tar.gz -o libressl.tar.gz \
  && curl -fSL http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-${LIBRESSL_VERSION}.tar.gz.asc -o libressl.tar.gz.asc \
  && gpg --batch --verify libressl.tar.gz.asc libressl.tar.gz \
  && tar -zxf libressl.tar.gz \
  && cd libressl-${LIBRESSL_VERSION} \
  && ./config \
    --prefix=/tmp/libressl/.libressl/ \
  && make -j $(getconf _NPROCESSORS_ONLN) \
  && mkdir -p /tmp/src \
  && cd /tmp/src \
  && git clone https://github.com/aperezdc/ngx-fancyindex.git ngx-fancyindex \
  && curl -fSL http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz -o nginx.tar.gz \
  && curl -fSL http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz.asc -o nginx.tar.gz.asc \
  && gpg --batch --verify nginx.tar.gz.asc nginx.tar.gz \
  && tar -zxf nginx.tar.gz \
  && cd nginx-${NGINX_VERSION} \
  && ./configure \
    --user=www-data \
    --group=www-data \
    --sbin-path=/usr/sbin/nginx \
    --with-http_ssl_module \
    --with-openssl=/tmp/libressl/libressl-${LIBRESSL_VERSION} \
    --with-ld-opt="-lrt" \
    --with-cc-opt="-O3" \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    --with-file-aio \
    --with-ipv6 \
    --with-threads \
    --with-stream \
    --with-stream_ssl_module \
    --with-http_v2_module \
    --prefix=/etc/nginx \
    --http-log-path=/var/log/nginx/access.log \
    --error-log-path=/var/log/nginx/error.log \
    --add-module=../ngx-fancyindex \
  && make -j $(getconf _NPROCESSORS_ONLN) \
  && make install \
  && make clean \
  && rm -rf /tmp/ /root/.gnupg \
  && strip -s /usr/sbin/nginx \
  && apk del ${build_pkgs} \
  && adduser -D www-data \
  && ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log

RUN rm -rf /var/www && mkdir /var/www && mkdir /var/www/website && mkdir /var/www/Downloads

VOLUME ["/etc/ssl/nginx"]
VOLUME ["/var/www/website"]
VOLUME ["/var/www/Downloads"]

CMD ["nginx"]
