# Investigating Client IP Visibility in Docker Desktop

## Objective

While implementing IP-based access control using nginx, I noticed that both my MacBook and iPhone received an HTTP 403 Forbidden response even though the iPhone IP address was explicitly allowed.

Instead of assuming that the nginx configuration was incorrect, I decided to investigate what IP address the container was actually seeing.

---

## Temporary Configuration

The original configuration was temporarily replaced with:

```nginx
server {
    listen 80;

    location / {
        default_type text/plain;
        return 200 "$remote_addr\n";
    }
}
```

This configuration returns the client IP address seen by nginx.

---

## Test Setup

### Device IP Addresses

| Device | IP Address |
|----------|------------|
| iPhone | 10.10.10.20 |
| MacBook | 10.10.10.30 |

---

## Website Access

### From iPhone

```text
http://10.10.10.30:8080
```

### From MacBook

```text
http://localhost:8080
```

---

## Observation

Surprisingly, both devices produced the same output:

```text
10.10.10.50
```

This address was different from both the iPhone and MacBook IP addresses.

Even though the devices had different IP addresses, the nginx container could only see Docker Desktop's internal network address.

---

## Understanding the Result

Docker Desktop on macOS runs Linux containers inside a lightweight Linux virtual machine.

The architecture can be simplified as:

```text
iPhone (10.10.10.20)
MacBook (10.10.10.30)
          ↓
     Docker Desktop
          ↓
      Linux VM
     (10.10.10.50)
          ↓
     nginx container
```

Docker Desktop performs Network Address Translation (NAT) while forwarding traffic into the Linux virtual machine.

Because of this, both requests appeared to nginx as coming from:

```text
10.10.10.50
```

which was the same address printed when visiting:

```text
http://localhost:8080
```

from the MacBook and

```text
http://10.10.10.30:8080
```

from the iPhone.

As a result, nginx could not distinguish between the two devices.

---

## Why IP-Based Filtering Failed

Since both devices appeared to nginx as:

```text
10.10.10.50
```

the following configuration became ineffective:

```nginx
location / {

    allow 10.10.10.30;
    deny all;

}
```

because nginx no longer had visibility of the original client IP addresses.

---

## Concepts Explored

- Docker Desktop networking
- nginx `$remote_addr`
- Client IP visibility
- Network Address Translation (NAT)
- Container debugging
- Docker internal networking
- Linux virtual machines

---

## Key Takeaway

The issue was not caused by an incorrect nginx configuration.

The experiment showed that Docker Desktop on macOS hides the original client IP addresses from containers by performing Network Address Translation (NAT).

As a result, multiple devices may appear to the container as the same internal address, making IP-based filtering unreliable in this environment.
