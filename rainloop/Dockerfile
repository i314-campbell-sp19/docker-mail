FROM debian:stretch-slim

RUN apt update && apt install -y wget apache2 php-common libapache2-mod-php php-cli php-curl php-xml
RUN cd /var/www && wget -qO- https://repository.rainloop.net/installer.php | php
RUN cd /var/www/rainloop && find . -type d -exec chmod 755 {} \; && find . -type f -exec chmod 644 {} \; && chown -R www-data:www-data . && cd .. && mv data html/data && mv rainloop html/rainloop && mv index.php html/index.php && cd html && rm index.html

ENTRYPOINT service apache2 start && tail -f /dev/null

EXPOSE 80 8080
