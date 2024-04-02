**Nginx Demo on Arch Linux**

This tutorial will serve as a guide to provision a demo nginx server on a DigitalOcean droplet. This tutorial assumes an Arch Linux droplet has already been created, and is started immediately post-install.

**Initial Configuration**

To begin, SSH into your droplet:

`ssh root@your_server_ip`

The first step should be to synchronize the pacman package database and run a system update:

`pacman -Syu`

We will be using vim as our text editor for this tutorial, but any text editor will work. Continue by installing vim and nginx

`pacman -S vim nginx`

Create the root directory for your project:

`mkdir -p /web/html/nginx-2420`

This is where your website documents will be stored. For this demo, we will be using the following HTML document:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        * {
            color: #db4b4b;
            background: #16161e;
        }
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
            font-family: sans-serif;
        }
    </style>
</head>
<body>
    <h1>All your base are belong to us</h1>
</body>
</html>
```

We will just copy and paste this document into `./nginx-2420/index.html`, however you could also use FTP or another file transfer tool to upload this document.

At this point, the server is staged for nginx to be configured and deployed.

**Nginx Configuration**

Create a new server block configuration for the site page. This are configuration files that are stored in `/etc/nginx/sites-available/nginx-2420`. We will create a new server block for this site, with the absolute path of `/etc/nginx/sites-available/nginx-2420` by running the following command:
`vim /etc/nginx/sites-available/nginx-2420`

In this file, enter the following configuration details. Make sure to replace `your_server_ip` with your droplet's public IP address.

```
# New server block
server { 
    # Instructs nginx to listen on port 80
    listen 80; 
    # Instructs nginx what IP address the server is running on
    server_name your_server_ip; 

    # Point nginx to the created root directory
    root /web/html/nginx-2420; 
    # Point nginx to the desired index page filename
    index index.html; 

    # Location block for handling requests
    # Attempt to serve URIs by first looking for an exact match of the filename. If that does not exist, try serving it as a directory by appending a trailing slash. If neither the filename nor directory exist, return a 404
    location / { 
        try_files $uri $uri/ =404; 
    }
}
```

We now have a valid nginx configuration file that is set up to serve our index.html file. The final configuration we must do with nginx is to create a symbolic link from our `./etc/nginx/sites-available/nginx-2420` directory to `/etc/nginx/sites-enabled/`. This is where nginx looks for sites to actively use and serve. You can do this with the following command:

`ln -s /etc/nginx/sites-available/nginx-2420 /etc/nginx/sites-enabled`

**systemd configuration**

We will use systemd as our service manager to handle running nginx and in turn our website. It uses "service files" to define and configure services we'd like it to handle. System-wide service files (like our nginx service) are stored in `/etc/systemd/system` directory. Create your nginx service file with the following command:

`vim /etc/systemd/system/nginx.service`

In this file, enter the following:

```
[Unit]
Description=ACIT-2420 Nginx Demo
After=network.target

[Service]
Type=forking
ExecStartPre=/usr/bin/nginx -t
ExecStart=/usr/bin/nginx
ExecReload=/usr/bin/nginx -s reload
ExecStop=/usr/bin/nginx -s stop

[Install]
WantedBy=multi-user.target
```

Unit Section:

`Description`: provides a description of the service
`After`: Specifies that this service should start after the network has been initialized

Service Section:
`Type`: specifies the type of service; in this case `forking` indicates the service forks into the background
`ExecStartPre`: Specifies a command to run before starting the service, in this case running the command to test the nginx configuration file for syntax errors
`ExecStart`: Specifies the command to start the nginx service
`ExecReload`: specifies the command to reload the nginx configuration without stopping the service
`ExecStop`: specifies the command to stop the nginx service

Install Section:
`WantedBy`: specifies the target that triggers the automatic startup during boot, in this case `multi-user.target`

This system file ensures that nginx is run as a forking service, with commands for starting, reloading, and stopping, with installation and dependency information.

Run the following command to reload the systemctl daemon and apply the changes:

`systemctl daemon-reload`

Run the following command to start the nginx service:

`systemctl start nginx`

Enable nginx to start on boot:

`systemctl enable nginx`




![image](https://github.com/ljackson330/nginx-2420/assets/153872676/140124f4-6026-4143-b7bd-91947a668798)
