OBJECTIVE

To host a static HTML page using Docker and Nginx and restrict access so that only a specific IP address can access the website.

STRUCTURE

allowip/
-Dockerfile
-nginx.conf
-index.html

DOCKERFILE

FROM nginx

COPY index.html /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

This uses original nginx image from DockerHub, copies the html file into nginx's web root and replaces nginx's default configuration with the custom nginx.conf configuration.

NGINX.CONF

server {
    listen 80;
    location / {
        allow 10.10.10.30;
        deny all;
        root /usr/share/nginx/html;
        index index.html;
    }
}

* listen 80 → nginx listens on port 80.
* allow 10.10.10.30 → only the specified IP is allowed.
* deny all → all other IPs receive a 403 Forbidden response.
* root /usr/share/nginx/html → location of website files.
* index index.html → default page.

BUILD IMAGE

docker build -t allowipimage .

RUN CONTAINER

docker run -d --name allowcontainer -p 8080:80 allowipimage

VISIT

http://localhost:8080

EXPECTED BEHAVIOUR

* Allowed IP → Website loads.
* Other IP addresses → HTTP 403 Forbidden

OBSERVATION ON DOCKER DESKTOP FOR MAC

While testing, I observed that both my Mac and iPhone received a 403 response even though the iPhone IP was explicitly allowed.

This happens because Docker Desktop on macOS runs Linux containers inside a lightweight Linux virtual machine and performs network address translation (NAT). As a result, nginx inside the container may not see the original client IP address but instead sees traffic originating from Docker’s internal network.

Therefore, IP-based access control inside the container may behave differently on Docker Desktop for Mac compared to native Linux environments.

CONCEPTS LEARNED

* Docker images and containers
* Dockerfile
* Port mapping
* Nginx web root
* Nginx configuration files
* allow and deny directives
* HTTP 403 Forbidden
* Docker Desktop networking
* Network Address Translation (NAT)
* Debugging containerized applications
* Difference between macOS Docker Desktop and native Linux containers

CONCLUSION

This project demonstrates how nginx access control can be configured inside Docker and highlights how Docker Desktop networking on macOS affects client IP visibility. The same configuration would work normally on native Linux environments.

