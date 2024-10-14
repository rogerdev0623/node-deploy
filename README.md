# Node.js Deployment

> Steps to deploy a Node.js app to Vultr using PM2, NGINX as a reverse proxy and an SSL from LetsEncrypt on Ubuntu

## 1. Sign up for Vultr

Are you looking for a reliable VPS service? Try [Vultr](https://www.vultr.com/?ref=8218452)! With top-notch performance and a wide range of server locations, Vultr makes it easy to deploy your applications.

👉 [Get started now!](https://www.vultr.com/?ref=8218452)

## 2. Create an Ubuntu Server on Vultr and Log In via SSH. Example: Ubuntu 22.04

I will be using the root user, but would suggest creating a new user

## 3. Install Node/NPM
```
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -

sudo apt install nodejs

node --version
```

## 4. Clone your project from Github
There are a few ways to get your files on to the server, I would suggest using Git
```
git clone yourproject.git
```

### 5. Install dependencies and test app
```
cd yourproject
npm install
npm start (or whatever your start command)
# stop app
ctrl+C
```
## 6. Setup PM2 process manager to keep your app running
```
sudo npm i pm2 -g
pm2 start app (or whatever your file name)

# Other pm2 commands
pm2 show app
pm2 status
pm2 restart app
pm2 stop app
pm2 logs (Show log stream)
pm2 flush (Clear logs)

# To make sure app starts when reboot
pm2 startup ubuntu
```
### You should now be able to access your app using your IP and port. Now we want to setup a firewall blocking that port and setup NGINX as a reverse proxy so we can access it directly using port 80 (http)

## 7. Setup ufw firewall
```
sudo ufw enable
sudo ufw status
sudo ufw allow ssh (Port 22)
sudo ufw allow http (Port 80)
sudo ufw allow https (Port 443)
```

## 8. Install NGINX and configure
```
sudo apt install nginx

sudo nano /etc/nginx/sites-available/default
```
Add the following to the location part of the server block
```
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000; #whatever port your app runs on
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
```
```
# Check NGINX config
sudo nginx -t

# Restart NGINX
sudo service nginx restart
```

### You should now be able to visit your IP with no port (port 80) and see your app. Now let's add a domain

## 9. Pointing Your Namecheap Domain to Your Server's IP

This guide explains how to point your Namecheap domain to your Vultr server.

### Step 1: Add A Records

1. **Log in** to your Namecheap account.
2. Go to the **Domain List** and select your domain.
3. Click on the **Advanced DNS** tab.
4. Add the following A records:

   - **Host:** @  
     **Value:** [Your Server's IP]
   
   - **Host:** www  
     **Value:** [Your Server's IP]

5. Save changes and wait for DNS propagation.

### Step 2: Test with Ping

To check if your domain is set up correctly, run the following command in your terminal or command prompt:

ping yourdomain.com

Replace `yourdomain.com` with your actual domain name. If you see your server's IP address in the response, your setup is successful!

10. Add SSL with LetsEncrypt
```
sudo apt-get update
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Only valid for 90 days, test the renewal process with
certbot renew --dry-run
```

Now visit https://yourdomain.com and you should see your Node app
