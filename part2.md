## Stage and Install Binary

To begin, upload the binary to your droplet using SFTP

`sftp user@your_droplet_ip`

When connected, use `put` to upload the file from your local computer to the remote droplet

`put Downloads/hello-server`

Exit sftp:

`quit`

Once the binary has been uploaded to your server, move it to an appropriate location such as `/usr/local/bin/hello-server`:

`sudo mv hello-server /usr/local/bin/hello-server`

Ensure the file is executable

`sudo chmod +x /usr/local/bin/hello-server`

Now that the binary is properly installed, we can move on to making it a service.

---

## Systemd configuration

Create a new unit file for your service:

`sudo vim /etc/systemd/system/hello-server.service`

In it, place the following configuration:

```
[Unit]
Description=Hello Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/hello-server
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

This configuration makes a very simple service that starts after the server is ready for multiple users to log in, after the network is up and running, and that will restart if it crashes.

Reload the systemctl daemon to load the unit file:

`sudo systemctl daemon-reload`

Now that systemd has the unit file loaded, the service is ready to enable and start:

`sudo systemctl enable --now hello-server`

Inspect the service status to ensure it started correctly and is running:

`sudo systemctl status hello-server`

![Screenshot from 2024-04-09 10-10-30](https://github.com/ljackson330/nginx-2420/assets/153872676/63214121-27c3-4adb-a2a0-1ca4bde9ee24)


---

## Nginx configuration

To reverse proxy requests from `http://droplet_ip/hello` and `http://droplet_ip/echo` to the service which is running at `127.0.0.1:8080`, we need to create a new nginx server block in `/etc/nginx/sites-available`:

`sudo vim /etc/nginx/sites-available/hello-server`

Paste the following into the file:

```
server {
    listen 80;  # Listen for incoming HTTP requests on port 80
    server_name 159.203.26.85;  # Droplet IP

    location /hey {  # Handle requests for URI path "/hey"
        proxy_pass http://127.0.0.1:8080/hey;  # Proxy requests to http://127.0.0.1:8080/hey
        proxy_set_header Host $host;  # Set Host header to Host header from original request
        proxy_set_header X-Real-IP $remote_addr;  # Set X-Real-IP header to client's IP address
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # Append client's IP to X-Forwarded-For header
        proxy_set_header X-Forwarded-Proto $scheme;  # Set protocol header to original protocol (http)
    }

    location /echo {  # Handle requests for URI path "/echo"
        proxy_pass http://127.0.0.1:8080/echo;  # Proxy requests to http://127.0.0.1:8080/echo
        proxy_set_header Host $host;  # Set Host header to Host header from original request
        proxy_set_header X-Real-IP $remote_addr;  # Set X-Real-IP header to client's IP address
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  # Append client's IP to X-Forwarded-For header
        proxy_set_header X-Forwarded-Proto $scheme;  # Set protocol header to original protocol (http)
    }
}

```

This is a very simple reverse proxy configuration to proxy requests to the backend service while retaining important request headers.

Once you have created that server block, enable it with a symbolic link:

`sudo ln -s /etc/nginx/sites-available/hello-server /etc/nginx/sites-enabled/`

Once you have enabled the server, reload nginx configuration:

`sudo systemctl reload nginx`

---

## UFW Setup

To install ufw, run the following commands:

`sudo pacman -Syu`

`sudo pacman -S ufw`

Once it has finished installing, we need to allow both port 80 and port 22 through our firewall. It is *very* import to allow 22 so that we can still SSH in to our droplet

`sudo ufw allow 22 80`

Enable the firewall with the following command:

`sudo ufw enable`

View the status of the firewall and its configured rules:

`sudo ufw status verbose`

![Screenshot from 2024-04-09 10-05-56](https://github.com/ljackson330/nginx-2420/assets/153872676/292f2bd2-a32d-44be-b006-5b1c02a4d5ac)

---

**Testing**

Our server is now running the hello-server binary, and nginx is reverse proxying requests from `http://droplet_ip/hello` and `http://droplet_ip/echo` to the binary which is running at `127.0.0.1:8080`

We can test this by running the following commands:

```
curl -X POST -H "Content-Type: application/json" \
-d '{"message": "Hello from your server"}' \
http://159.203.26.85/echo
{"message": "Hello from your server"}
```


`curl http://159.203.26.85/hey`

The output should look as follows:

![Screenshot from 2024-04-09 09-41-52](https://github.com/ljackson330/nginx-2420/assets/153872676/34b3e111-e6fa-42da-bd0a-a6c5303d48a5)
