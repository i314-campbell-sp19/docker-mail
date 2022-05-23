FROM debian:stable-slim

ENV DEBIAN_FRONTEND noninteractive

# Postfix install 
RUN  echo "postfix postfix/main_mailer_type string 'No Configuration'" | debconf-set-selections \
    && apt update && apt install -y bash postfix \
    && ln -s /etc/resolv.conf /var/spool/postfix/resolv.conf
COPY docker-entrypoint.sh /
COPY docker-entrypoint.d/* /docker-entrypoint.d/
#ONBUILD COPY /docker-entrypoint.d/* /docker-entrypoint.d/

# Allow SMTP port to be visible
EXPOSE 25 587

ENTRYPOINT ["/docker-entrypoint.sh"]
# Start postfix server and leave the container running
CMD /usr/sbin/postfix start-fg
