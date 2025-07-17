# A Guide to Nginx Variables

Nginx variables are powerful tools that allow you to access request and connection information, and use it to dynamically control your server's behavior. Variables are identified by the `$` prefix followed by a name.

This guide covers the most common and useful Nginx variables, with examples of how to use them in your configurations.

---

## Core Nginx Variables

These variables provide access to fundamental request properties.

### `$uri` vs. `$request_uri`

-   **`$uri`**: This variable contains the current URI in its normalized form. It does not include the query string (arguments). The value of `$uri` can change during request processing (e.g., due to an internal redirect).
-   **`$request_uri`**: This variable contains the complete, original request URI, including the query string. It never changes during request processing.

**Example:**
For a request to `http://example.com/path/to/page?arg1=val1&arg2=val2`:
-   `$uri` will be `/path/to/page`.
-   `$request_uri` will be `/path/to/page?arg1=val1&arg2=val2`.

**Use Case:** Logging or routing based on the original request.

```nginx
location / {
    # Log the original request URI
    access_log /var/log/nginx/access.log main;

    # Use the normalized URI for file lookups
    try_files $uri $uri/ =404;
}
```

### `$args` and `$arg_`

-   **`$args`**: Contains the full query string (the part of the URL after `?`).
-   **`$arg_<name>`**: Contains the value of a specific argument in the query string. For example, `$arg_username` would get the value of the `username` parameter.

**Example:**
For a request to `/login?user=john&pass=secret`:
-   `$args` will be `user=john&pass=secret`.
-   `$arg_user` will be `john`.
-   `$arg_pass` will be `secret`.

**Use Case:** Caching based on query parameters.

```nginx
location /api/ {
    # Create a cache key based on the URI and specific arguments
    proxy_cache_key "$uri?$arg_id";
    proxy_pass http://backend;
}
```

---

## HTTP Header Variables

Nginx makes HTTP request headers available as variables prefixed with `$http_`. Dashes (`-`) in the header name are converted to underscores (`_`).

-   **`$http_user_agent`**: The `User-Agent` header from the client's request.
-   **`$http_cookie`**: The `Cookie` header.
-   **`$http_referer`**: The `Referer` header, which indicates the page that linked to the current one.
-   **`$http_x_forwarded_for`**: The `X-Forwarded-For` header, often used by proxies to indicate the original client IP.

**Example:** Blocking requests with an empty User-Agent.

```nginx
server {
    # ...
    if ($http_user_agent = "") {
        return 403; # Forbidden
    }
    # ...
}
```

---

## Request and Connection Variables

These variables provide information about the client's connection and the server itself.

### Client Information

-   **`$remote_addr`**: The IP address of the client.
-   **`$remote_port`**: The port number of the client.

### Server Information

-   **`$host`**: The hostname from the `Host` request header. If the header is missing, it's the server name that processed the request.
-   **`$server_name`**: The name of the server block that processed the request.
-   **`$server_addr`**: The IP address of the server that accepted the request.
-   **`$server_port`**: The port number of the server that accepted the request.
-   **`$server_protocol`**: The protocol used for the request, usually `HTTP/1.0`, `HTTP/1.1`, or `HTTP/2.0`.

**Use Case:** Setting headers in a proxy pass.

```nginx
location /api/ {
    proxy_pass http://backend_server;

    # Pass the original host and client IP to the backend
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```
*Note: `$proxy_add_x_forwarded_for` is a special variable that appends `$remote_addr` to the existing `$http_x_forwarded_for` header.*

---

## File Path Variables

These variables are useful for locating files on the server's filesystem.

-   **`$document_root`**: The root directory for the current request, as defined by the `root` directive.
-   **`$realpath_root`**: The absolute path corresponding to the `$document_root`, with all symbolic links resolved.

**Use Case:** Passing the full path of a script to a FastCGI processor like PHP-FPM.

```nginx
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_pass unix:/run/php/php-fpm.sock;

    # Construct the full script filename
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```
*Note: `$fastcgi_script_name` is another variable that captures the request URI, often used in this context.*

---

## Creating Custom Variables with `set`

You can define your own variables using the `set` directive. This is useful for improving readability and reusing values.

**Syntax:** `set $variable_name value;`

**Example:** Creating a custom cache key.

```nginx
location /content/ {
    # Set a default language
    set $lang "en";

    # Override language if the 'lang' cookie is present
    if ($http_cookie ~* "lang=([a-z]{2})") {
        set $lang $1; # $1 is a capture group from the regex
    }

    # Use the custom variable in a cache key
    proxy_cache_key "$uri-$lang";
    proxy_pass http://content_backend;
}
```

This example demonstrates how variables can be conditionally modified, making your Nginx configuration much more flexible and powerful.
