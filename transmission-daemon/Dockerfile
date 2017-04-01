FROM alpine:latest

MAINTAINER Jonathan Elias Caicedo [Jonathan@jcaicedo.com]

RUN apk add --update \
    transmission-daemon \
    && rm -rf /var/cache/apk/*

RUN mkdir -p /transmission/downloads \
  && mkdir -p /transmission/incomplete

COPY start-transmission.sh .

VOLUME ["/transmission/downloads"]
VOLUME ["/transmission/incomplete"]

EXPOSE 51413/tcp 51413/udp

RUN chmod +x /start-transmission.sh
CMD ["/start-transmission.sh"]

