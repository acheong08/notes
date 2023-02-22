# Some notes on how to use the software

## NGINX

### `/etc/nginx/nginx.conf`
```nginx
stream {
    map $ssl_preread_server_name $name {
        duti.tech trojan;
        default nginx;
    }
    upstream trojan {
        server 127.0.0.1:4433;
    }
    upstream nginx {
        server 127.0.0.1:4432;
    }
    server {
        listen 443;
        listen [::]:443;
        proxy_pass $name;
        ssl_preread on;
    }
}
```

### `/etc/nginx/sites-enabled/*`
```nginx
server {
	root /path/to/root;
	index index.html index.htm index.nginx-debian.html;
	server_name example.com;
	location / {
		auth_basic "Administrator’s Area";
		auth_basic_user_file /etc/apache2/.htpasswd;
		try_files $uri $uri/ /index.html =404;
	}
    listen [::]:4432 ssl ipv6only=on; # managed by Certbot
    listen 4432 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = example.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	listen 80 ;
	listen [::]:80 ;

	server_name images.duti.tech;
    return 404; # managed by Certbot


}
```
