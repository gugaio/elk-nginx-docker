# Dockerfile.nginx

FROM nginx:latest
COPY nginx.conf /etc/nginx/nginx.conf
COPY reverse-proxy.conf /etc/nginx/conf.d/reverse-proxy.conf
EXPOSE 8020
EXPOSE 80
STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
