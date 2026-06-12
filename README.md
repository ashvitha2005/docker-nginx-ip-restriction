# Docker + Nginx IP Restriction Lab

## Objective

The goal of this project was to host a custom HTML page inside an nginx container and restrict access to only specific devices using IP-based access control.

This experiment was also used to understand how Docker Desktop on macOS differs from Docker running natively on Linux and how Network Address Translation (NAT) affects client IP visibility.

---

## Technologies Used

- Docker
- nginx
- Dockerfile
- Ubuntu
- Kali Linux
- UTM Virtual Machine
- Bridged Networking
- Docker Desktop for macOS

---

# Project Structure

```
.
├── Dockerfile
├── nginx.conf
└── index.html
```

---

# Dockerfile

```dockerfile
FROM nginx

COPY index.html /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

The image is built from the official nginx image.

The custom HTML page is copied to nginx's web root and the default nginx configuration is replaced with a custom configuration.

---

# nginx Configuration

Example:

```nginx
server {
    listen 80;

    location / {

        allow PHONE_IP;
        deny all;

        root /usr/share/nginx/html;
        index index.html;
    }
}
```

This configuration allows only the specified IP address and blocks every other client.

---

# Initial Experiment: Docker Desktop on macOS

## Setup

```
Phone
MacBook
     ↓
Docker Desktop
     ↓
Linux VM
     ↓
nginx container
```

Docker Desktop on macOS runs Linux containers inside a lightweight Linux virtual machine.

Because containers require a Linux kernel, Docker Desktop creates a hidden Linux VM and runs Docker Engine inside it.

---

# Unexpected Result

Both the MacBook and the phone received:

```
403 Forbidden
```

even though only one device was allowed.

---

# Debugging Using `$remote_addr`

To understand what nginx was seeing, the configuration was temporarily changed to:

```nginx
location / {
    default_type text/plain;
    return 200 "$remote_addr\n";
}
```

Surprisingly, both devices printed:

```
10.10.10.50
```

instead of their real WiFi addresses.

---

# Understanding NAT

Docker Desktop introduces an additional network layer:

```
Phone (10.10.10.20)
MacBook (10.10.10.30)
            ↓
      Docker Desktop
            ↓
      Linux VM
       (10.10.10.50)
            ↓
      nginx container
```

Traffic entering the Linux VM is translated by Docker Desktop.

As a result, nginx sees all requests as coming from:

```
10.10.10.50
```

instead of the original client addresses.

Therefore, IP-based filtering becomes ineffective because multiple devices appear to have the same IP.

---

# Second Experiment: Ubuntu VM using NAT

Setup:

```
Mac
 ↓
UTM NAT
 ↓
Ubuntu
 ↓
Docker
 ↓
nginx
```

Result:

IP restriction failed again.

Reason:

NAT translated client addresses before they reached nginx.

---

# Final Experiment: Bridged Networking

## Setup

```
Phone
MacBook
Kali Linux
      ↓
Ubuntu VM (Bridged)
      ↓
Docker
      ↓
nginx
```

In bridged mode, the Ubuntu VM becomes another device on the physical network.

Docker runs natively inside Linux and no additional VM layer exists between Docker and the operating system.

This allows nginx to see the original client IP addresses.

---

# Results

Allowed phone (IP:10.10.10.20):

```
Custom HTML page displayed successfully
```

Second phone:

```
403 Forbidden
```

Kali Linux:

```
403 Forbidden
```

MacBook:

```
403 Forbidden
```

Ubuntu:

```
Access allowed
```

This proved that nginx IP restriction works correctly when the original client IP addresses are preserved.

---

# Why Docker Desktop Failed

Docker Desktop does not run containers directly on macOS.

Instead:

```
macOS
 ↓
Hidden Linux VM
 ↓
Docker Engine
 ↓
Container
```

The Linux VM is isolated from the physical network and Docker Desktop acts as a router between the host and the VM.

To forward traffic between these two networks, Docker Desktop performs Network Address Translation (NAT).

NAT rewrites source IP addresses, causing nginx to lose visibility of the original client.

---

# Native Linux vs Docker Desktop

## Native Linux

```
Phone
 ↓
Ubuntu
 ↓
Docker
 ↓
Container
```

nginx sees:

```
10.10.10.20
```

and IP restriction works by allowing only the phone with the above IP.

---

## Docker Desktop

```
Phone
 ↓
macOS
 ↓
Docker Desktop
 ↓
Linux VM
 ↓
Container
```

nginx sees:

```
10.10.10.50
```

for multiple devices.

---

# Important Concepts Learned

- Containers require a Linux kernel.
- Docker Desktop creates a lightweight Linux VM on macOS.
- Docker Desktop performs NAT between macOS and the Linux VM.
- NAT hides original client IP addresses.
- nginx `allow` and `deny` depend on the client IP seen by nginx.
- `$remote_addr` is useful for debugging IP-related issues.
- Bridged networking preserves real client IP addresses.
- Docker running natively on Linux behaves differently from Docker Desktop.

---

# Conclusion

IP-based access control using nginx works correctly inside Docker containers when the original client IP addresses are preserved.

Docker Desktop on macOS hides client IPs because traffic passes through a hidden Linux VM and undergoes Network Address Translation (NAT).

Using bridged networking and Docker running natively on Linux preserves the original client IP addresses and allows nginx access control rules to function as expected.

---

## Key Takeaway

The issue was not caused by nginx, Docker, or the configuration itself.

The root cause was the loss of client IP visibility introduced by NAT.
