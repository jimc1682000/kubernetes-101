FROM nginx:1.17.10-alpine

# nginx.conf
ENV WORKER_PROCESS_NUM=1
ENV SEND_FILE=off

COPY nginx/uwsgi_param /etc/nginx/
COPY nginx/mysite_nginx.conf /etc/nginx/conf.d/default.conf
COPY demo/static /usr/share/nginx/html/static