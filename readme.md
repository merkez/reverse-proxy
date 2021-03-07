## Reverse Proxy and Let's Encrypt 

This reverse proxy is currently [07.03.2021]  operating at [api.ntp-event.dk](api.ntp-event.dk)

Haaukins API was normally using its own webserver to serve files and information however, it was not possible to retrieve client information through built-in `http` module which is available in Go package. Since we wanted to have more control over the API and in future on Haaukins platform, implementing a reverse proxy in front of the application gave us more power to track clients and block them if they are trying to access outside of specified countries in Nginx configuration file. 

This repository is containning a custom nginx image which includes ip2gep module to block visitors to website outside of defined countries. 

`/guacamole` - This path is proxied to guacamole docker container which is managed by Haaukins itself, since it is a streaming connection between client and server, proxy_buffer it set to off.
The configuration for `/guacamole` path is taken from its official page, in its offical page following information below provided regarding to reverse proxy. 


> The proxy configuration belongs within a dedicated location block, declaring the backend hosting Guacamole and explicitly specifying the "Connection" and "Upgrade" headers mentioned earlier:

```conf
location /guacamole/ {
    proxy_pass http://HOSTNAME:8080/guacamole/;
    proxy_buffering off;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    access_log off;
}
```
> ## Changing the path

> If you wish to serve Guacamole through Nginx under a path other than /guacamole/, the configuration will need to be altered slightly to take cookies into account. Although Guacamole does not rely on receipt of cookies in general, cookies are required for the proper operation of the HTTP tunnel. If the HTTP tunnel is used, and cookies cannot be set, users may be unexpectedly denied access to their connections.

> Regardless of the location specified for the proxy, cookies set by Guacamole will be set using its own absolute path within the backend (/guacamole/). If this path differs from that used by Nginx, the path in the cookie needs to be modified using proxy_cookie_path:
```conf
location /new-path/ {
    proxy_pass http://HOSTNAME:8080/guacamole/;
    proxy_buffering off;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    proxy_cookie_path /guacamole/ /new-path/;
    access_log off;
}
```

`/challengesFrontend` - This path is basically a websocket, for this reason it has proxy_buffer set to off. 

`/`   - Main path for the application does not need to have proxy_pass set to off. 

The application instead does not use SSL/TLS configuration from Go webserver and it is not publicly available to outside without reverse-proxy, reverse proxy enables to have a connection between application and client. While providing connection betweeen two, it ensures connection stability, security, traffic load and etc. Currently there is no load balancing mechanism set it up however, it might be set it up soon. 

