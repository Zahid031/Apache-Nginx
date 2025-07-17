# Understanding Nginx Location Blocks

The `location` block is a fundamental part of an Nginx configuration. It allows you to control how Nginx responds to different request URIs. This guide explains the various types of `location` blocks and how they work.

---

## Syntax

The basic syntax for a `location` block is:

```nginx
location [modifier] /uri/ {
  # ... configuration directives ...
}
```

-   **`modifier`** (optional): A symbol that changes how Nginx matches the URI.
-   **`/uri/`**: The URI pattern to match against the request.

Nginx selects the most specific location block that matches the request URI. The order of `location` blocks in the configuration file matters, especially for regular expressions.

---

## Location Block Types

Here are the different types of location blocks, with examples:

### 1. Prefix Match (`/`)

This is the most common type of location block. It matches any request that starts with the specified URI.

**Syntax:** `location /uri/ { ... }`

**Example:**

```nginx
location / {
    root /var/www/html;
    index index.html;
    try_files $uri $uri/ =404;
}
```

-   **Explanation:** This is a catch-all location. If no other more specific location block matches, this one will be used. It serves files from the `/var/www/html` directory.

**Another Example:**

```nginx
location /images/ {
    root /var/www/static;
}
```

-   **Explanation:** This will match any request starting with `/images/`, such as `/images/logo.png` or `/images/backgrounds/main.jpg`.

### 2. Exact Match (`=`)

This block matches only the exact URI specified. It is the most specific and is checked first.

**Syntax:** `location = /uri { ... }`

**Example:**

```nginx
location = /about {
    return 200 "This is the About page.";
}
```

-   **Explanation:** This block will *only* match the request for `/about`. It will not match `/about/` or `/about/us`.

### 3. Regular Expression Match (`~` and `~*`)

These blocks use regular expressions to match the URI.

-   `~`: Case-sensitive regular expression match.
-   `~*`: Case-insensitive regular expression match.

**Syntax:**
- `location ~ \.regex$ { ... }`
- `location ~* \.regex$ { ... }`

**Case-Sensitive Example:**

```nginx
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

-   **Explanation:** This matches any request URI ending in `.php` (case-sensitive).

**Case-Insensitive Example:**

```nginx
location ~* \.(jpg|jpeg|gif|png)$ {
    access_log off;
    expires 30d;
}
```

-   **Explanation:** This matches any request URI ending in `.jpg`, `.jpeg`, `.gif`, or `.png`, regardless of case (e.g., `/images/PHOTO.JPG` will match).

### 4. Preferential Prefix Match (`^~`)

If this prefix match is the best match, Nginx will not check for regular expression matches.

**Syntax:** `location ^~ /uri/ { ... }`

**Example:**

```nginx
location ^~ /uploads/ {
    # Don't process any regex matches for this path
    deny all;
}
```

-   **Explanation:** If a request starts with `/uploads/`, Nginx will use this block immediately and will not bother checking any of the regex locations. This is useful for performance and for ensuring certain directories are handled in a specific way.

---

## Order of Precedence

Nginx checks for location matches in the following order:

1.  **Exact Match (`=`)**: If the URI is an exact match, this block is used.
2.  **Preferential Prefix Match (`^~`)**: If the URI is a prefix match, Nginx stops searching and uses this block.
3.  **Regular Expression Matches (`~`, `~*`)**: Nginx checks regex locations in the order they appear in the configuration file. The first matching regex is used.
4.  **Prefix Match (`/`)**: If no regex matches, the most specific prefix match is used.

---

## Common Use Cases

### Proxying Requests

You can use a location block to pass requests to a backend server.

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:5000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### Handling Errors

You can define a custom page for errors like 404 (Not Found).

```nginx
error_page 404 /404.html;

location = /404.html {
    root /var/www/errors;
    internal; # This ensures the page is only accessible through Nginx, not directly by users.
}
```

### Denying Access

You can deny access to certain files or directories.

```nginx
location ~ /\. {
    deny all;
}
```
-   **Explanation:** This denies access to hidden files (e.g., `.htaccess`, `.git`).

### Redirects

You can redirect a URI to another location.

```nginx
location /old-page {
    return 301 /new-page;
}
```
-   **Explanation:** This will permanently redirect any request for `/old-page` to `/new-page`.