OBJECTIVE

To host a static HTML page using Docker and Nginx and restrict access so that only a specific IP address can access the website.

STRUCTURE

allowip/
├── Dockerfile
├── nginx.conf
└── index.html

DOCKERFILE

FROM nginx

COPY index.html /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

This uses original nginx image from DockerHub, copies the html file into nginx's web root and replaces nginx's default configuration with the custom nginx.conf configuration.

NGINX.CONF

server {
    listen 80;

    location / {
        allow 192.xxx.x.xx;
        deny all;

        root /usr/share/nginx/html;
        index index.html;
    }
}

