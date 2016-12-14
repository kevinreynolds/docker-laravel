FROM nginx:1.11-alpine

MAINTAINER Alexander Diachenko

COPY conf.d/*.conf /etc/nginx/conf.d/

EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
