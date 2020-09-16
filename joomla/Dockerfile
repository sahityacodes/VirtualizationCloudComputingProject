FROM kwindsor/php7-apache-ubuntu
WORKDIR /var/www/html
RUN apt -y update
RUN apt install -y wget mariadb-client unzip
RUN mkdir /var/www/tmp
RUN mkdir /var/www/logs
#RUN mkdir /var/www/html/volume
#RUN chown -R www-data:www-data /var/www/html
#RUN chown -R www-data:www-data /var/www/tmp
#RUN chown -R www-data:www-data /var/www/logs
RUN rm -f /var/www/html/index.html
#RUN rm -f 
RUN wget https://github.com/joomla/joomla-cms/releases/download/3.6.4/Joomla_3.6.4-Stable-Full_Package.zip -P /var/tmp/
#RUN unzip /var/tmp/Joomla_3.6.4-Stable-Full_Package.zip -d /var/www/html
#RUN rm -f /var/tmp/Joomla_3.6.4-Stable-Full_Package.zip
#RUN mv installation volume/installation
#RUN touch volume/configuration.php
#RUN ln -s volume/configuration.php
ADD wait-for-it.sh /usr/local/bin/wait-for-it.sh
RUN chmod +x /usr/local/bin/wait-for-it.sh
ADD firstrun.sh /usr/local/bin/firstrun.sh
RUN chmod +x /usr/local/bin/firstrun.sh
RUN sed -i '3i/usr/local/bin/firstrun.sh' /usr/local/bin/docker-php-entrypoint