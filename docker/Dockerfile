FROM nginx:1.19.4
ENV TZ=Asia/Shanghai
ENV SSL_CERTIFICATE=/app/ssl/cert.crt
ENV SSL_CERTIFICATE_KEY/app/ssl/cert.key
ENV HOSTNAME=yourdomain.com
ENV PASSWORD=123456
ENV CONTEXT_PATH=/random_me/
COPY ./config/nginx.conf /etc/nginx/nginx.conf
COPY ./ssl/* /app/ssl/



