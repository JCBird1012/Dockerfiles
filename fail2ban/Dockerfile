FROM alpine:edge
LABEL maintainer="Jonathan Elias Caicedo <Jonathan@jcaicedo.com>"

RUN apk update && apk add fail2ban ip6tables && mkdir /var/run/fail2ban

CMD ["/usr/bin/fail2ban-client", "-f", "start"]
