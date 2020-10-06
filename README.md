# SMSglue

For subscribers of VoIP.ms who use Acrobits Softphone or Groundwire, SMSglue will enable SMS text messaging.

## Quick Start

```
$ npm install smsglue
$ node
> require('smsglue').listen(5000)
```

Then point your browser to localhost on port 5000.

## Notes

For this to be useful, you'll want to create a reverse proxy with a 
proper domain and https encryption. It may also be a good idea to
strip sensitive information from your access logs (see example below),
as there will be access tokens and messages crossing the wire as GET
requests. 

## nginx Example

```conf
http {
  ...

  log_format filter '$remote_addr - $remote_user [$time_local] "$req" $status $body_bytes_sent "$http_referer" "$http_user_agent"';

  server {
    listen 443 ssl;
    server_name smsglue.com;

    ssl on;
    ssl_certificate     fullchain.cer;
    ssl_certificate_key privatekey.key;

    location / {

      # Strip everything after hyphen "-" in log request
      set $req $request;
      if ($req ~ (.+)\-(.*)) { set $req $1; }
      access_log access.log filter;
      
      # Reverse Proxy
      proxy_pass http://127.0.0.1:5000;
      
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
}
```

## Customization

It's possible to append custom HTML content to the end of the index.html
page by setting an environment variable named `BEFORE_CLOSING_BODY_TAG`.
For example, this can be used to include a footer and Google Analytics.

## Starting as a Service (systemctl | Ubuntu Server)

Create a new service file in /lib/systemd/system (let's just call it smsglue.service).

Copy the below contents and modify a couple entries:

```
[Unit]
Description=Node.js smsglue Server
After=syslog.target network-online.target

[Service]
Type=simple
# change the user name to the account where smsglue is installed
User=changeme
# change the working directory to the location of the smsglue root folder
WorkingDirectory=/home/changeme/smsglue
Environment=NODE_ENV=production PORT=5000
# change the folder to the same as the working directory
ExecStart=/usr/bin/node /home/changeme/smsglue/service.js
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```

Enable the service to run at startup.

`sudo systemctl enable smsglue.service`

Start the service.

`sudo systemctl start smsglue.service`

Check to verify that the service is active and running.

`sudo systemctl status smsglue.service`
