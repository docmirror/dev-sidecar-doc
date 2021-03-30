FROM nginx:1.19.4
ENV TZ=Asia/Shanghai
ENV PASSWORD=''
COPY ./config/nginx.conf /etc/nginx/nginx-template.conf
COPY ./config/start.sh /app/start.sh
RUN chmod +x /app/start.sh

CMD ["/app/start.sh"]
