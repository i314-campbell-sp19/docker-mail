FROM debian:stable-slim 

ENV DEBIAN_FRONTEND noninteractive

RUN apt update && apt install -y dovecot-imapd dovecot-lmtpd \
    && adduser --system --group --disabled-login --home /home/vmail vmail \
    && touch /var/log/dovecot.log /var/log/dovecot-info.log \ 
    && mkdir -p /Maildir \
    && chown vmail:vmail /var/log/dovecot.log /var/log/dovecot-info.log /Maildir

COPY dovecot.conf /etc/dovecot/
COPY users /etc/dovecot/

COPY docker-entrypoint.sh /
COPY docker-entrypoint.d/* /docker-entrypoint.d/
#ONBUILD COPY /docker-entrypoint.d/* /docker-entrypoint.d/

EXPOSE 143
 
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["/usr/sbin/dovecot", "-Fc", "/etc/dovecot/dovecot.conf"]
