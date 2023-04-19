# How To Upstream Reroute in NGinx

This is a simple Nginx routing example, where traffic arriving at port 80 is rerouted based on the request. If the request starts with *api* it is rerouted to port 5000. Traffic arriving at */* will go to port 3000. 

````
upstream client {
    server client:3000;
}

upstream api {
    server api:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://client;
    }

    location /api {
        rewrite /api/(.*) /$1 break;
        proxy_pass http://api;
    }
}
````

