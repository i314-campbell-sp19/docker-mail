FROM alpine

# Postfix install 
RUN apk add --no-cache  bash postfix \
    && touch /var/log/mail.log
COPY docker-entrypoint.sh /
COPY docker-entrypoint.d/* /docker-entrypoint.d/
#ONBUILD COPY /docker-entrypoint.d/* /docker-entrypoint.d/

# Allow SMTP port to be visible
EXPOSE 25 587

ENTRYPOINT ["/docker-entrypoint.sh"]
# Start postfix server and leave the container running
CMD /usr/sbin/postfix start && tail -f /var/log/mail.log
