# Nginx Reverse Proxy: A Comprehensive Guide

This document provides a detailed guide to understanding and configuring an Nginx reverse proxy. It uses a real-world configuration example to illustrate key concepts, from basic setup to advanced directives.

## Table of Contents

1.  [What is a Reverse Proxy?](#what-is-a-reverse-proxy)
    *   [Key Benefits](#key-benefits)
2.  [Nginx Configuration Analysis](#nginx-configuration-analysis)
    *   [Global Directives](#global-directives)
    *   [Events Block](#events-block)
    *   [HTTP Block](#http-block)
    *   [Server Block: The Core of the Reverse Proxy](#server-block-the-core-of-the-reverse-proxy)
3.  [Deep Dive: `proxy_pass` and `proxy_set_header`](#deep-dive-proxy_pass-and_proxy_set_header)
    *   [The `proxy_pass` Directive](#the-proxy_pass-directive)
    *   [The `proxy_set_header` Directive](#the-proxy_set_header-directive)
4.  [Understanding Caching and HTTP Status Codes](#understanding-caching-and-http-status-codes)
    *   [`200 OK`](#200-ok)
    *   [`304 Not Modified`](#304-not-modified)
5.  [Final Configuration Example](#final-configuration-example)

---

## What is a Reverse Proxy?

A reverse proxy is an intermediary server that sits between clients and one or more backend servers. It receives client requests, forwards them to the appropriate backend server, and returns the server's response to the client. This architecture makes it seem to the client as if they are communicating directly with the reverse proxy itself.

This setup provides a layer of abstraction that is fundamental to modern web architecture.

### Key Benefits

*   **Load Balancing**: Distributes incoming traffic across multiple backend servers, preventing any single server from becoming a bottleneck and improving application scalability and fault tolerance.
*   **Enhanced Security**: Acts as a shield for backend servers. By hiding their IP addresses and network topology, it protects them from direct internet exposure and potential attacks like DDoS.
*   **SSL/TLS Termination**: Offloads the computational overhead of encrypting and decrypting traffic. The reverse proxy can handle SSL/TLS connections from clients, forwarding unencrypted requests to the backend servers over a secure internal network.
*   **Caching**: Stores copies of frequently requested content. When a client requests this content, the reverse proxy can serve it directly from its cache, reducing latency and decreasing the load on backend servers.
*   **Request Routing & URL Rewriting**: Intelligently routes requests to different backend servers based on the request's URL, headers, or other properties. This is essential for microservices architectures.
*   **Compression**: Can compress server responses before sending them to the client, reducing bandwidth usage and speeding up load times.

## Nginx Configuration Analysis

The following is a breakdown of a typical Nginx configuration file for a reverse proxy.

### Global Directives

These directives are defined at the top level of the `nginx.conf` file and affect the core functionality of Nginx.

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;
```

*   `user www-data;`: Specifies the user and group under which the worker processes will run. This is a security best practice to run workers with non-root privileges.
*   `worker_processes auto;`: Instructs Nginx to automatically determine the optimal number of worker processes based on the number of available CPU cores.
*   `pid /run/nginx.pid;`: Defines the file where the process ID of the main Nginx process will be stored.
*   `error_log ...;`: Configures the path and severity level for the main error log.
*   `include ...;`: Allows for including other configuration files, promoting a modular and organized configuration structure.

### Events Block

The `events` block contains directives that configure Nginx's connection processing model.

```nginx
events {
    worker_connections 768;
}
```

*   `worker_connections 768;`: Sets the maximum number of simultaneous connections that can be opened by a single worker process.

### HTTP Block

This block contains directives for handling HTTP and HTTPS traffic.

```nginx
http {
    sendfile on;
    tcp_nopush on;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    access_log /var/log/nginx/access.log;
    gzip on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

*   `sendfile on;`: Enables the use of the `sendfile()` system call, which allows for more efficient file transmission by copying data directly between file descriptors.
*   `tcp_nopush on;`: Optimizes the sending of response headers and the start of a file's content in a single packet. It's often used with `sendfile`.
*   `include /etc/nginx/mime.types;`: Includes a file that maps file extensions to MIME types, so the browser knows how to handle different file formats.
*   `access_log ...;`: Defines the location of the access log, which records every request made to the server.
*   `gzip on;`: Enables on-the-fly compression of responses.

### Server Block: The Core of the Reverse Proxy

The `server` block defines a virtual server that handles requests for a specific `server_name`. This is where the reverse proxy logic is implemented.

```nginx
server {
    listen 80;
    server_name 35.173.231.106;

    location = / {
        proxy_pass http://ec2-34-228-20-228.compute-1.amazonaws.com;
    }

    location /py/ {
        proxy_pass https://zahid03.pythonanywhere.com/;
        proxy_set_header Host zahid03.pythonanywhere.com;
    }
}
```

*   `listen 80;`: Tells Nginx to listen for incoming connections on port 80 (the standard port for HTTP).
*   `server_name ...;`: Specifies which server block should handle a request based on the `Host` header of the incoming request.
*   `location` blocks: These define how to process requests based on their URI.
    *   `location = /`: This handles requests for the exact root URI (`/`). The `=` modifier ensures that only this URI matches.
    *   `location /py/`: This handles any request URI that begins with `/py/`.

## Deep Dive: `proxy_pass` and `proxy_set_header`

These two directives are the heart of the reverse proxy functionality.

### The `proxy_pass` Directive

`proxy_pass` forwards a request to the specified backend server.

> `proxy_pass http://ec2-34-228-20-228.compute-1.amazonaws.com;`

In this example, any request matching the location block is sent to the backend server at the given address.

### The `proxy_set_header` Directive

`proxy_set_header` allows you to modify or add headers to the request that is sent to the backend server.

> `proxy_set_header Host zahid03.pythonanywhere.com;`

This is a critical directive. Many backend applications rely on the `Host` header to identify which site or tenant is being requested. When Nginx proxies a request, the default `Host` header it sends is the hostname of the proxy server itself. This directive overwrites that default and passes the original hostname requested by the client, ensuring the backend application can respond correctly.

## Understanding Caching and HTTP Status Codes

The status codes `200 OK` and `304 Not Modified` are central to how web caching works.

### `200 OK`

This is the standard successful response. It means the server has found the requested resource and is sending it in the response body. This happens on the first visit to a page or after a forced refresh.

### `304 Not Modified`

This response indicates that the resource requested by the browser has not been modified since the last time it was accessed. The browser sends conditional headers like `If-Modified-Since` or `If-None-Match` with its request. If the server determines the content is unchanged, it sends a `304` response with an empty body. The browser then loads the resource from its local cache, saving bandwidth and reducing latency.

## Final Configuration Example

Here is the complete, cleaned-up `nginx.conf` for reference.

```nginx
# /etc/nginx/nginx.conf

# --- Global Directives ---
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

# --- Events Block ---
events {
    worker_connections 768;
}

# --- HTTP Block ---
http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # SSL Settings
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    # Logging Settings
    access_log /var/log/nginx/access.log;

    # Gzip Settings
    gzip on;
    # For more granular control, you can uncomment and configure these:
    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Virtual Host Configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

    # --- Server Block for Reverse Proxy ---
    server {
        listen 80;
        server_name 35.173.231.106; # Public IP of the reverse proxy

        # Route root requests to a static content server
        location = / {
            proxy_pass http://ec2-34-228-20-228.compute-1.amazonaws.com;
            # It's good practice to set headers even for simple proxies
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Route requests for /py/ to a Python application
        location /py/ {
            proxy_pass https://zahid03.pythonanywhere.com/;
            proxy_set_header Host zahid03.pythonanywhere.com;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```