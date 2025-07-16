# Deploying a Static Website with Nginx

This guide provides step-by-step instructions for deploying a static website on a server using Nginx.

## Prerequisites

- A server (e.g., an AWS EC2 instance) with SSH access.
- Your static website files ready for deployment.
- An SSH key pair for connecting to your server.

## 1. Install Nginx

First, connect to your server and install Nginx using the following commands:

```bash
sudo apt update
sudo apt install nginx
```

Once the installation is complete, enable Nginx to start on boot:

```bash
sudo systemctl enable nginx
```

## 2. Upload Website Files

Next, upload your static website files to the server. You can use the `scp` command to securely copy the files. Replace `"your_key.pem"` with the path to your private key and `your_server_ip` with your server's IP address or hostname.

```bash
scp -i "your_key.pem" -r bloggingtemplate/ ubuntu@your_server_ip:/home/ubuntu/
```

This command copies the `bloggingtemplate` directory to the `/home/ubuntu/` directory on your server.

## 3. Configure Nginx

Now, you need to configure Nginx to serve your website.

### Back up the Default Configuration

It's a good practice to back up the default Nginx configuration file before making changes:

```bash
cd /etc/nginx/
sudo cp nginx.conf nginx.conf.backup
```

### Edit the Nginx Configuration

Open the main Nginx configuration file with a text editor like `nano`:

```bash
sudo nano /etc/nginx/nginx.conf
```

Replace the contents of the file with the following configuration. This sets up a server block to listen on port 80 and serve your website files. Make sure to replace `your_server_ip` with your server's actual IP address or domain name.

```nginx
events {
    # multi_accept on;
}

http {
    
    include       /etc/nginx/mime.types;


    server {
        listen 80;
        server_name your_server_ip;

        root /home/ubuntu/bloggingtemplate;

        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

**Note:** For the `root` directive, ensure the path points to the directory containing your `index.html` file.

## 4. Set File Permissions

If you encounter permission issues, you may need to adjust the ownership and permissions of your website files.

Set the ownership to the `www-data` user, which is the user Nginx runs as:

```bash
sudo chown -R www-data:www-data /home/ubuntu/bloggingtemplate
```

Set the appropriate permissions for the directories and files:

```bash
sudo chmod -R 755 /home/ubuntu/bloggingtemplate
```

You may also need to grant execute permissions to the parent directories:
```bash
chmod o+x /home
chmod o+x /home/ubuntu
```

## 5. Test and Reload Nginx

After saving your configuration, test it for syntax errors:

```bash
sudo nginx -t
```

If the test is successful, reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

Alternatively, you can restart the Nginx service:

```bash
sudo systemctl restart nginx
```

## 6. Access Your Website

Your website should now be accessible via your server's IP address or domain name in a web browser.

### Troubleshooting CSS/JS Issues

If your CSS, JavaScript, or images are not loading correctly, it might be due to incorrect MIME types. The configuration provided in Step 3 (`include /etc/nginx/mime.types;`) should handle this.

If you still face issues, ensure the `mime.types` file is included correctly in your `nginx.conf`. Also, try clearing your browser cache or opening the website in a new incognito window to avoid loading stale files.
