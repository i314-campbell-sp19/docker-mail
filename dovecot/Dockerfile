FROM alpine 

RUN apk add --no-cache dovecot \
    && addgroup -S -g 5000 vmail \
    && adduser -S -u 5000 -g vmail -h /home/vmail vmail \
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
