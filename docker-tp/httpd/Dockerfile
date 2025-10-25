FROM httpd:2.4-alpine

COPY ./index.html /usr/local/apache2/htdocs/index.html
COPY ./my-proxy.conf /usr/local/apache2/conf.d/my-proxy.conf
RUN echo "IncludeOptional conf.d/*.conf" >> /usr/local/apache2/conf/httpd.conf