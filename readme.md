# Installing n8n on Ubuntu: Simple steps and with free SSL

## Prerequisites

1. Docker
2. Nginx
3. Certbot (For SSL setup)

## Step 1: Installing Docker

First, you need to update your system's package list by running:

```shell
sudo apt update
```

Then, you can install Docker by running:

```shell
sudo snap install docker
```

## Step 2: Starting n8n in Docker

To start n8n in Docker, you can use the following command:

```shell
sudo docker run -d --restart unless-stopped -it \
    --name n8n \
    -p 5678:5678 \
    -e N8N_HOST="your-domain.com" \
    -e VUE_APP_URL_BASE_API="https://your-domain.com/" \
    -e WEBHOOK_TUNNEL_URL="https://your-domain.com/" \
    -v ~/.n8n:/root/.n8n \
    n8nio/n8n
```

Please replace `your-domain.com` with your actual domain. After this, you can access n8n on `your-domain.com:5678`. 

## Step 3: Installing Nginx

You can install Nginx by running:

```shell
sudo apt install nginx
```

## Step 4: Configuring Nginx

After installing Nginx, you need to configure it to reverse proxy the n8n web interface. Here's an example configuration:

First, open a new file using a text editor. Here we'll use nano:

```shell
sudo nano /etc/nginx/sites-available/n8n
```

Then, paste the following content:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
    }
}
```

Again, replace `your-domain.com` with your actual domain.

After that, you can create a symbolic link of this file in the `sites-enabled` directory:

```shell
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
```

Then, test the configuration and restart Nginx:

```shell
sudo nginx -t
sudo systemctl restart nginx
```

At this point, you should be able to access n8n on `your-domain.com`.

## Step 5: Setting up SSL with Certbot

First, install Certbot and the Nginx plugin by running:

```shell
sudo apt install certbot python3-certbot-nginx
```

Then, you can run Certbot and follow the on-screen instructions:

```shell
sudo certbot --nginx -d your-domain.com
```

After finishing these steps, n8n should be set up with HTTPS on your domain.

Note: Don't forget to set up DNS A record for `your-domain.com` to point to your server IP address. You would also need to allow ports 80, 443 (for HTTPS), and 5678 (for n8n) in your firewall.