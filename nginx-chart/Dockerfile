FROM nginx:latest

WORKDIR ./nginx-chart

COPY nginx_config.conf /etc/nginx/nginx.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
