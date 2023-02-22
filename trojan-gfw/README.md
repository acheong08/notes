# NGINX

### `/etc/nginx/nginx.conf`
```nginx
stream {
    map $ssl_preread_server_name $name {
        example.com trojan;
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

	server_name example.com;
    return 404; # managed by Certbot


}
```

# Trojan

### Server
`/usr/local/etc/trojan/config.json`
```json
{
    "run_type": "server",
    "local_addr": "127.0.0.1",
    "local_port": 4433,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "<UUID4>",
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/letsencrypt/live/example.com/fullchain.pem",
        "key": "/etc/letsencrypt/live/example.com/privkey.pem",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "alpn_port_override": {
            "h2": 81
        },
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": "",
	"sni": "example.com"
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "<db_name>",
        "password": "<db_password>",
        "key": "",
        "cert": "",
        "ca": ""
    }
}
```

### Client
```json

{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "example.com",
    "remote_port": 443,
    "password": [
        "<UUID4>"
    ],
    "log_level": 1,
    "ssl": {
        "verify": true,
        "verify_hostname": true,
        "cert": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:AES128-SHA:AES256-SHA:DES-CBC3-SHA",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "sni": "",
        "alpn": [
            "h2",
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "curves": ""
    },
    "tcp": {
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    }
}
```

# Tools/Scripts:
- [easygfw](https://github.com/fluenceuwu/easygfw/blob/main/setup.sh)
- [trojan](https://github.com/trojan-gfw/trojan) - Unmaintained - TODO: fork
