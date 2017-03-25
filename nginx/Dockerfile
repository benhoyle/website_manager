FROM aarch64/alpine:latest

RUN apk add --update nginx && \
    mkdir -p /run/nginx/ && \
    rm /etc/nginx/conf.d/default.conf

COPY app.conf /etc/nginx/conf.d/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]