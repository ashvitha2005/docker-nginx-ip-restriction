# Docker + Nginx IP Restriction

## Objective

To host a static HTML page using Docker and Nginx and restrict access so that only a specific IP address can access the website.

---

## Project Structure

```text
allowip/
├── Dockerfile
├── nginx.conf
└── index.html
```

---

## Dockerfile

```dockerfile
FROM nginx

COPY index.html /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

This uses the official nginx image from Docker Hub, copies the HTML file into nginx's web root, and replaces nginx's default configuration with a custom configuration.

---

## nginx.conf

```nginx
server {
    listen 80;

    location / {
        allow 10.10.10.20;
        deny all;

        root /usr/share/nginx/html;
        index index.html;
    }
}
```

### Explanation

- `listen 80` → nginx listens on port 80.
- `allow 10.10.10.20` → only the specified IP address is allowed.
- `deny all` → all other clients receive an HTTP 403 Forbidden response.
- `root /usr/share/nginx/html` → location of website files.
- `index index.html` → default page served by nginx.

---

## Build Image

```bash
docker build -t allowipimage .
```

---

## Run Container

```bash
docker run -d --name allowcontainer -p 8080:80 allowipimage
```

---

## Access Website

Open:

```text
http://localhost:8080
```

---

## Expected Behaviour

| Client | Result |
|----------|--------|
| Allowed IP | Website loads |
| Other IP addresses | HTTP 403 Forbidden |

---

# Observation on Docker Desktop for macOS

While testing, both the MacBook and the iPhone received a **403 Forbidden** response even though the iPhone IP was explicitly allowed.

To investigate this, the nginx configuration was temporarily modified to return the client IP address:

```nginx
location / {
    default_type text/plain;
    return 200 "$remote_addr\n";
}
```

Surprisingly, nginx displayed the same internal address for requests originating from both devices.

This happens because Docker Desktop on macOS runs Linux containers inside a lightweight Linux virtual machine and performs **Network Address Translation (NAT)**. As a result, nginx inside the container does not always see the original client IP address but instead sees traffic originating from Docker Desktop's internal network.

Therefore, IP-based access control inside containers may behave differently on Docker Desktop compared to native Linux environments.

---

## Concepts Learned

- Docker images and containers
- Dockerfile
- Port mapping
- nginx web root
- nginx configuration files
- `allow` and `deny` directives
- HTTP 403 Forbidden
- Docker Desktop networking
- Network Address Translation (NAT)
- Debugging containerized applications
- Client IP visibility
- Using `$remote_addr` for troubleshooting
- Difference between Docker Desktop on macOS and Docker running natively on Linux

---

## Conclusion

This project demonstrates how nginx access control can be configured inside Docker and highlights how Docker Desktop networking on macOS affects client IP visibility.

Although the configuration itself is correct, Docker Desktop's use of a lightweight Linux virtual machine and Network Address Translation (NAT) causes multiple clients to appear as the same internal address from the container's perspective.

The same configuration is expected to work normally on native Linux environments where the original client IP addresses are preserved.
