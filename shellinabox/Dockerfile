FROM ubuntu:16.04

ENV SIAB_VERSION=2.19 \
  SIAB_USERCSS="Normal:-/etc/shellinabox/options-enabled/00_White-On-Black.css,Reverse:+/etc/shellinabox/options-enabled/00+Black-on-White.css;Colors:+/etc/shellinabox/options-enabled/01+Color-Terminal.css,Monochrome:-/etc/shellinabox/options-enabled/01_Monochrome.css" \
  SIAB_PORT=4200 \
  SIAB_ADDUSER=true \
  SIAB_USER=Jonathan \
  SIAB_USERID=1000 \
  SIAB_GROUP=Jonathan \
  SIAB_GROUPID=1000 \
  SIAB_PASSWORD=Tango \
  SIAB_SHELL=/bin/bash \
  SIAB_HOME=/home/guest \
  SIAB_SUDO=false \
  SIAB_SSL=false \
  SIAB_SERVICE=/:LOGIN \
  SIAB_PKGS=none \
  SIAB_SCRIPT=none

RUN apt-get update && apt-get install -y curl openssh-client \
      shellinabox=${SIAB_VERSION} && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
  ln -sf '/etc/shellinabox/options-enabled/00+Black on White.css' \
    /etc/shellinabox/options-enabled/00+Black-on-White.css && \
  ln -sf '/etc/shellinabox/options-enabled/00_White On Black.css' \
    /etc/shellinabox/options-enabled/00_White-On-Black.css && \
  ln -sf '/etc/shellinabox/options-enabled/01+Color Terminal.css' \
    /etc/shellinabox/options-enabled/01+Color-Terminal.css

EXPOSE 4200

VOLUME /etc/shellinabox /var/log/supervisor /home

ADD assets/entrypoint.sh /usr/local/sbin/

ENTRYPOINT ["entrypoint.sh"]
CMD ["shellinabox"]
