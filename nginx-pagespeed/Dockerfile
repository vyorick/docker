# Use phusion/baseimage as base image. To make your builds
# reproducible, make sure you lock down to a specific version, not
# to `latest`! See
# https://github.com/phusion/baseimage-docker/blob/master/Changelog.md
# for a list of version numbers.
FROM phusion/baseimage:0.9.17
MAINTAINER Vladimir Lavrentiev <vladimir.lavrentiev@gmail.com>

# Set correct environment variables.
ENV HOME /root
ENV DEBIAN_FRONTEND noninteractive
ENV DOCKER_NAME rev_proxy_pagespeed

# https://github.com/pagespeed/ngx_pagespeed/releases
ENV PAGESPEED_VERSION 1.9.32.4

# http://nginx.org/en/download.html
ENV NGINX_VERSION 1.9.3

# https://www.openssl.org/source
ENV OPENSSL_VERSION 1.0.1p

# Enable ssh access
RUN rm -f /etc/service/sshd/down

# Regenerate SSH host keys. baseimage-docker does not contain any, so you
# have to do that yourself. You may also comment out this instruction; the
# init system will auto-generate one during boot.
RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

# !!! Enabling the insecure key permanently !!! REMOVE ME AT PRODUCTION
RUN /usr/sbin/enable_insecure_key

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

### ...put your own build instructions here... ###

EXPOSE 80 443

# RUN apt-get update
# RUN DEBIAN_FRONTEND=noninteractive sudo apt-get upgrade -y

RUN useradd -r -s /usr/sbin/nologin nginx && mkdir -p /var/log/nginx /var/cache/nginx && \
	apt-get update && DEBIAN_FRONTEND=noninteractive apt-get upgrade -y
RUN apt-get -y install wget build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-dev libgoogle-perftools-dev libperl-dev
RUN echo "Downloading ngx_pagespeed v${PAGESPEED_VERSION} ..." && wget -qO - https://github.com/pagespeed/ngx_pagespeed/archive/v${PAGESPEED_VERSION}-beta.tar.gz | tar zxf - -C /tmp
RUN echo "Downloading pagespeed psol v${PAGESPEED_VERSION} ..." && wget -qO - https://dl.google.com/dl/page-speed/psol/${PAGESPEED_VERSION}.tar.gz | tar xzf  - -C /tmp/ngx_pagespeed-${PAGESPEED_VERSION}-beta
RUN	echo "Downloading nginx v${NGINX_VERSION} ..." && wget -qO - http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz | tar zxf - -C /tmp
RUN	echo "Downloading openssl v${OPENSSL_VERSION} ..." && wget -qO - https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz | tar xzf  - -C /tmp
RUN cd /tmp/nginx-${NGINX_VERSION} && ./configure \
		--prefix=/etc/nginx  \
		--sbin-path=/usr/sbin/nginx  \
		--conf-path=/etc/nginx/nginx.conf  \
		--error-log-path=/var/log/nginx/error.log \
		--http-log-path=/var/log/nginx/access.log \
		--pid-path=/var/run/nginx.pid \
		--lock-path=/var/run/nginx.lock \
		--http-client-body-temp-path=/var/cache/nginx/client_temp \
		--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
		--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp  \
		--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp  \
		--http-scgi-temp-path=/var/cache/nginx/scgi_temp  \
		--user=nginx  \
		--group=nginx  \
		--with-http_ssl_module  \
		--with-http_realip_module  \
		--with-http_addition_module  \
		--with-http_sub_module  \
		--with-http_dav_module  \
		--with-http_flv_module  \
		--with-http_mp4_module  \
		--with-http_gunzip_module  \
		--with-http_gzip_static_module  \
		--with-http_random_index_module  \
		--with-http_secure_link_module \
		--with-http_stub_status_module  \
		--with-http_auth_request_module  \
		--without-http_autoindex_module \
		--without-http_ssi_module \
		--with-threads  \
		--with-stream  \
		--with-stream_ssl_module  \
		--with-mail  \
		--with-mail_ssl_module  \
		--with-file-aio  \
		--with-http_spdy_module  \
		--with-cc-opt='-g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2'  \
		--with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,--as-needed' \
		--with-ipv6 \
		--with-pcre-jit \
		--with-openssl=/tmp/openssl-${OPENSSL_VERSION} \
		--add-module=/tmp/ngx_pagespeed-${PAGESPEED_VERSION}-beta && make && make install
RUN rm -Rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
	apt-get purge -yqq wget build-essential && \
	apt-get autoremove -yqq && \
	apt-get clean

# for mount tmpfs volumes
VOLUME ["/var/cache/nginx"]
VOLUME ["/var/cache/pagespeed"]

# for mount configs
VOLUME ["/etc/nginx/sites-enabled"]

ENV DEFAULT_APP_USER nginx
ENV DEFAULT_APP_GROUP nginx
ENV	DEFAULT_UPLOAD_MAX_SIZE 30M
ENV DEFAULT_NGINX_MAX_WORKER_PROCESSES 8
ENV DEFAULT_CHOWN_APP_DIR true

COPY . /

RUN chmod 750 /etc/service/nginx/run /boot.sh /init_nginx.sh

RUN /init_nginx.sh

ENTRYPOINT ["/boot.sh"]

CMD ["/sbin/my_init"]

