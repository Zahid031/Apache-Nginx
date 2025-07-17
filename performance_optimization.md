# Nginx Performance Optimization

Optimizing Nginx is key to improving the performance of your web server. A well-configured Nginx can handle a high volume of traffic efficiently. This guide provides simple yet effective optimization techniques with examples.

---

### 1. Worker Processes and Connections

-   **`worker_processes`**: This directive determines how many worker processes Nginx will use. A good starting point is to set this to the number of CPU cores your server has. Setting it to `auto` lets Nginx handle this automatically.

-   **`worker_connections`**: This sets the maximum number of simultaneous connections that can be opened by a single worker process. The default is usually sufficient, but you can increase it if you have high traffic.

**Configuration Example:**

```nginx
# Sets worker processes to the number of available CPU cores
worker_processes auto;

events {
    # Sets the maximum connections per worker process
    worker_connections 1024;
}
```

---

### 2. Keep-Alive Connections

-   **`keepalive_timeout`**: This directive sets how long a keep-alive connection will stay open between the server and the client. Keeping connections open reduces the overhead of establishing new ones for subsequent requests. A value between `60s` and `75s` is generally recommended.

**Configuration Example:**

```nginx
http {
    # Sets the timeout for keep-alive connections
    keepalive_timeout 65s;
}
```

---

### 3. Gzip Compression

-   **`gzip`**: Compressing responses before sending them to the client can significantly reduce the amount of data transferred. This is especially effective for text-based files like HTML, CSS, and JavaScript.

**Configuration Example:**

```nginx
http {
    # Enables gzip compression
    gzip on;

    # Compresses data for clients that connect via proxies
    gzip_proxied any;

    # Sets the compression level (1-9)
    gzip_comp_level 4;

    # Specifies the file types to compress
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

---

### 4. Caching Static Assets

-   **`expires`**: You can instruct browsers to cache static files (like images, CSS, and JavaScript) for a long time. This reduces the number of requests to your server for returning visitors.

**Configuration Example:**

```nginx
http {
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        # Caches these file types for one year
        expires 365d;
    }
}
```

---

### 5. Buffering

-   **`client_body_buffer_size`**: Sets the buffer size for the client request body. If the body is larger than the buffer, it will be written to a temporary file.
-   **`client_max_body_size`**: Defines the maximum allowed size for a client request body.
-   **`client_header_buffer_size`**: Sets the buffer size for the client request header.

**Configuration Example:**

```nginx
http {
    # Buffer size for POST submissions
    client_body_buffer_size 128k;

    # Maximum size for file uploads
    client_max_body_size 10m;

    # Buffer size for headers
    client_header_buffer_size 1k;
}
```

---

### 6. Timeouts

-   **`client_body_timeout`**: Maximum time the server will wait for a client to send the request body.
-   **`client_header_timeout`**: Maximum time the server will wait for a client to send the request header.
-   **`send_timeout`**: Maximum time the server will wait for a client to accept a response.

**Configuration Example:**

```nginx
http {
    # Timeout for reading the client body
    client_body_timeout 12s;

    # Timeout for reading the client header
    client_header_timeout 12s;

    # Timeout for transmitting a response to the client
    send_timeout 10s;
}
```

---

### 7. `sendfile` and `tcp_nopush`

-   **`sendfile`**: This directive enables the use of the `sendfile()` system call, which copies data directly from one file descriptor to another. This is more efficient than reading a file into a buffer and then writing it to the socket.
-   **`tcp_nopush`**: This directive, used with `sendfile on;`, tells Nginx to send HTTP response headers in one packet.

**Configuration Example:**

```nginx
http {
    # Enables more efficient file transmission
    sendfile on;

    # Optimizes packet delivery
    tcp_nopush on;
}
```