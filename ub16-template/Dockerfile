FROM ubuntu:16.04
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install apt-utils -y && \
    apt-get install wget curl tzdata dialog -y && \
    apt-get install -qq -y libcurl3 libenchant1c2a libxpm4 libc-client2007e libmcrypt4 libaspell15 librecode0 \
                           libvpx-dev libtidy-0.99-0 libxslt1.1 libmysqlclient20 libpng12-0 libjpeg-turbo8 libfreetype6

RUN wget -O -  https://get.acme.sh | sh

ARG OLS_VERSION
ARG PHP_VERSION

RUN wget https://openlitespeed.org/preuse/openlitespeed-$OLS_VERSION.tgz && \
    tar xvf openlitespeed-$OLS_VERSION.tgz && cd openlitespeed && \
    LINENUM=$(grep -n "LSPHPVER=" _in.sh | cut -d: -f 1) && \
    sed -i "${LINENUM}d" _in.sh && \
    sed -i "${LINENUM}iLSPHPVER=${PHP_VERSION}" _in.sh && \
    ./install.sh && \
	rm -rf ../openlitespeed ../openlitespeed-$OLS_VERSION.tgz

RUN apt-get install -y lsphp$PHP_VERSION lsphp$PHP_VERSION-curl lsphp$PHP_VERSION-dbg lsphp$PHP_VERSION-gd lsphp$PHP_VERSION-gmp lsphp$PHP_VERSION-imap \
                       lsphp$PHP_VERSION-intl lsphp$PHP_VERSION-ldap lsphp$PHP_VERSION-mcrypt lsphp$PHP_VERSION-mysql lsphp$PHP_VERSION-pspell lsphp$PHP_VERSION-recode lsphp$PHP_VERSION-sqlite \
                       lsphp$PHP_VERSION-tidy lsphp$PHP_VERSION-xmlrpc lsphp$PHP_VERSION-xsl

RUN wget -O /usr/local/lsws/admin/misc/lsup.sh \
    https://raw.githubusercontent.com/litespeedtech/openlitespeed/master/dist/admin/misc/lsup.sh && \
    chmod +x /usr/local/lsws/admin/misc/lsup.sh && \
    ln -s /usr/local/lsws/lsphp$PHP_VERSION/bin/php /usr/bin/php

EXPOSE 7080

ADD docker.conf /usr/local/lsws/conf/templates/docker.conf
ADD httpd_config.sh /usr/local/lsws/bin/httpd_config.sh
ADD httpd_config.xml /usr/local/lsws/conf/httpd_config.xml
ADD htpasswd /usr/local/lsws/admin/conf/htpasswd
RUN chmod +x /usr/local/lsws/bin/httpd_config.sh && bash /usr/local/lsws/bin/httpd_config.sh && /usr/local/lsws/bin/httpd_config.sh
RUN chown 999:999 /usr/local/lsws/conf -R
RUN cp -RP /usr/local/lsws/conf/ /usr/local/lsws/.conf/
RUN cp -RP /usr/local/lsws/admin/conf /usr/local/lsws/admin/.conf/
RUN ln -sf /usr/local/lsws/lsphp$PHP_VERSION/bin/lsphp /usr/local/lsws/fcgi-bin/lsphp5
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /var/www/vhosts/
CMD ["/usr/local/lsws/bin/lswsctrl","start"]