INVESTIGATING CLIENT IP VISIBILTY IN DOCKER DESKTOP

While implementing IP-based access control using Nginx, I noticed that both my MacBook and iPhone received a 403 Forbidden response even though the iPhone IP address was explicitly allowed.

Instead of assuming there was a configuration error, I decided to investigate what IP address the Nginx container was actually seeing.

TEMPORARY CONFIGURATION

I temporarily replaced the original configuration with:
server {
    listen 80;
    location / {
        default_type text/plain;
        return 200 "$remote_addr\n";
    }
}

This returns the client IP address seen by Nginx.

TEST SETUP

Actual device IP addresses:
iPhone  : 192.xxx.x.xx
MacBook : 192.xxx.x.xx

WEBSITE ACCESS

iPhone  → http://MacbookIP:8080
MacBook → http://localhost:8080

OBSERVATION

Surprisingly, both devices produced the same output:

192.xxx.xx.x
(An IP that is different from both my iPhone IP and MacBook IP)
Even though the devices had different IP addresses, the Nginx container could only see Docker’s internal network address.

CONCLUSION

Docker Desktop for macOS runs Linux containers inside a lightweight Linux virtual machine and performs Network Address Translation (NAT).

Because of this, both my MacBook and iPhone appeared to the Nginx container as:
192.xxx.xx.x (IP printed when I did http://localhost:8080 in MacBook and http://MacbookIP:8080 in iPhone)
As a result, IP-based filtering inside the container could not distinguish between the two devices.

CONCEPTS EXPLORED

* Docker Desktop networking
* Nginx $remote_addr
* Client IP visibility
* Network Address Translation (NAT)
* Container debugging
* Docker internal networking

--KEY TAKEAWAY--

The issue was not caused by an incorrect Nginx configuration. The experiment showed that Docker Desktop on macOS hides the original client IP addresses from containers, causing multiple devices to appear as the same internal address.
