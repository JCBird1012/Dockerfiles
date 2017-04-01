FROM alpine:latest
MAINTAINER Jonathan Elias Caicedo [Jonathan@jcaicedo.com]

#ENV VARIABLES
ENV GHOST_SOURCE /usr/src/app
ENV GHOST_CONTENT /var/lib/ghost
ENV GHOST_VERSION 0.11.7

#Change WORKDIR to ghost directory
WORKDIR $GHOST_SOURCE

RUN apk --no-cache add nodejs tar tini \
    && apk --no-cache add --virtual devs gcc make python wget unzip ca-certificates \
    && wget -O ghost.zip https://ghost.org/zip/ghost-latest.zip \
    && unzip ghost.zip \
    && npm install --production \
    && rm ghost.zip \
    && apk del devs unzip make \
    && npm cache clean \
    && rm -rf /tmp/npm* /var/cache/apk/*

#Copy over, and grant executable permission to the startup script
COPY ./entrypoint.sh /
RUN chmod +x /entrypoint.sh

#Run Init System
ENTRYPOINT [ "/sbin/tini" ]

#Run Startup script
CMD [ "/entrypoint.sh" ]
