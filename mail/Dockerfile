FROM ubuntu:16.04
LABEL maintainer="Jonathan Elias Caicedo <Jonathan@jcaicedo.com>"

# Packages
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update -q --fix-missing && \
  apt-get -y upgrade && \ 
  apt-get -y install --no-install-recommends \
    amavisd-new \
    arj \
    bzip2 \
    ca-certificates \
    curl \
    dirmngr \
    dovecot-core \
    dovecot-imapd \
    dovecot-lmtpd \
    dovecot-managesieved \
    dovecot-pop3d \
    dovecot-sieve \
    ed \
    fetchmail \
    file \
    gamin \
    gzip \
    iproute2 \
    libmail-spf-perl \
    libnet-dns-perl \
    libsasl2-modules \
    opendkim \
    opendkim-tools \
    opendmarc \
    p7zip \
    postfix \
    pyzor \
    razor \
    rsyslog \
    sasl2-bin \
    postgrey \
    unzip \
    && \
  curl -sk http://neuro.debian.net/lists/trusty.de-m.libre > /etc/apt/sources.list.d/neurodebian.sources.list && \
  apt-key adv --recv-keys --keyserver hkp://pool.sks-keyservers.net 0xA5D32F012649A5A9 && \
  curl https://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add - && \
  echo "deb http://packages.elastic.co/beats/apt stable main" | tee -a /etc/apt/sources.list.d/beats.list && \
  apt-get update -q --fix-missing && apt-get -y upgrade fail2ban filebeat && \ 
  apt-get autoremove --purge && apt-get clean && apt-get autoclean && rm -rf /var/lib/apt/lists/* && \
  rm -rf /usr/share/locale/* && rm -rf /usr/share/man/* && rm -rf /usr/share/doc/* && \
  touch /var/log/auth.log

# Configures Dovecot
RUN sed -i -e 's/include_try \/usr\/share\/dovecot\/protocols\.d/include_try \/etc\/dovecot\/protocols\.d/g' /etc/dovecot/dovecot.conf
RUN sed -i -e 's/#mail_plugins = \$mail_plugins/mail_plugins = \$mail_plugins sieve/g' /etc/dovecot/conf.d/15-lda.conf
RUN sed -i -e 's/^.*lda_mailbox_autocreate.*/lda_mailbox_autocreate = yes/g' /etc/dovecot/conf.d/15-lda.conf
RUN sed -i -e 's/^.*lda_mailbox_autosubscribe.*/lda_mailbox_autosubscribe = yes/g' /etc/dovecot/conf.d/15-lda.conf
RUN sed -i -e 's/^.*postmaster_address.*/postmaster_address = '${POSTMASTER_ADDRESS:="postmaster@domain.com"}'/g' /etc/dovecot/conf.d/15-lda.conf
COPY target/dovecot/auth-passwdfile.inc /etc/dovecot/conf.d/
COPY target/dovecot/??-*.conf /etc/dovecot/conf.d/

#Enables Postgrey
COPY target/postgrey/postgrey /etc/default/postgrey
COPY target/postgrey/postgrey.init /etc/init.d/postgrey
RUN chmod 755 /etc/init.d/postgrey
RUN mkdir /var/run/postgrey
RUN chown postgrey:postgrey /var/run/postgrey

# Enables Amavis
RUN sed -i -r 's/#(@|   \\%)bypass/\1bypass/g' /etc/amavis/conf.d/15-content_filter_mode
RUN useradd -u 5000 -d /home/docker -s /bin/bash -p $(echo docker | openssl passwd -1 -stdin) docker

# Enables Pyzor and Razor
USER amavis
RUN razor-admin -create && razor-admin -register && pyzor discover
USER root

# Configure DKIM (opendkim)
# DKIM config files
COPY target/opendkim/opendkim.conf /etc/opendkim.conf
COPY target/opendkim/default-opendkim /etc/default/opendkim

# Configure DMARC (opendmarc)
COPY target/opendmarc/opendmarc.conf /etc/opendmarc.conf
COPY target/opendmarc/default-opendmarc /etc/default/opendmarc
COPY target/opendmarc/ignore.hosts /etc/opendmarc/ignore.hosts

# Configure fetchmail
COPY target/fetchmail/fetchmailrc /etc/fetchmailrc_general
RUN sed -i 's/START_DAEMON=no/START_DAEMON=yes/g' /etc/default/fetchmail

# Configures Postfix
COPY target/postfix/main.cf target/postfix/master.cf /etc/postfix/
RUN echo "" > /etc/aliases

# Configuring Logs
RUN mkdir -p /var/log/mail && chown syslog:root /var/log/mail && \
  sed -i -r 's|/var/log/mail|/var/log/mail/mail|g' /etc/rsyslog.d/50-default.conf && \
  sed -i -r 's|;auth,authpriv.none|;mail.none;mail.error;auth,authpriv.none|g' /etc/rsyslog.d/50-default.conf && \
  sed -i -r 's|/var/log/mail|/var/log/mail/mail|g' /etc/logrotate.d/rsyslog

COPY ./target/bin /usr/local/bin
# Start-mailserver script
COPY ./target/start-mailserver.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/*

EXPOSE 25 587 995 

CMD /usr/local/bin/start-mailserver.sh


ADD target/filebeat.yml.tmpl /etc/filebeat/filebeat.yml.tmpl

