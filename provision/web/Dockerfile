FROM nginx:latest

RUN rm /etc/nginx/nginx.conf

COPY nginx/*.conf /etc/nginx/
COPY nginx/conf.d/*.conf /etc/nginx/conf.d/
COPY nginx/global /etc/nginx/global

RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log

# App dir
RUN mkdir -p /var/www
WORKDIR /var/www
VOLUME /var/www

EXPOSE 80 443
