FROM nginx:latest

COPY ./nginx-chart/nginx_config.conf /etc/nginx/nginx.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
