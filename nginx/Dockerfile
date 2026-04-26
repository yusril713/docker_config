FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
COPY sites /etc/nginx/conf.d

RUN mkdir -p /var/log/nginx

EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
