# Secure Odoo Web Access with SSL by Letsencrypt and Nginx Reverse Proxy in Ubuntu 16.04 Xenial

1. Install Nginx and Certbot

	```
	apt update
	apt install -y software-properties-common
	add-apt-repository ppa:certbot/certbot
	apt update
	apt install -y python-certbot-nginx nginx
	```

---

2. Modify file ```/etc/nginx/sites-available/default``` using editor ```vim``` or ```nano```
	
	a. Find line:
	
	```
	...
	server_name _;
	...
	```
	
	Replace the underscore character with your domains:
	
	```
	...
	server_name example.com www.example.com;
	...
	```
	
	b. Find these line:

	```
	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
	```
	
	Change to:
	
	```
	# root /var/www/html;

	# index index.html index.htm index.nginx-debian.html;

    # location / {
       # try_files $uri $uri/ =404;
    # }
	```

	c. Add these lines under ```server_name example.com www.example.com;```

	```
	location / {
        proxy_pass  http://127.0.0.1:8069;
        # force timeouts if the backend dies
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        # set headers
        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto https;
    }
	```

	d. Save and exit editor

---

3. Generate the certificate, make sure port ```80```, and ```443``` are open to public

	```
	certbot --nginx -d example.com -d www.example.com --agree-tos -m user@example.com --redirect 
	```

	And follow the instruction.

---

4. If the SSL succesfully obtained, certbot will print th following message:

	```
	IMPORTANT NOTES:
	- Congratulations! Your certificate and chain have been saved at:
	/etc/letsencrypt/live/example.com/fullchain.pem
	Your key file has been saved at:
	/etc/letsencrypt/live/example.com/privkey.pem
	Your cert will expire on XXXX-XX-XX. To obtain a new or tweaked
	version of this certificate in the future, simply run certbot
	again. To non-interactively renew *all* of your certificates, run
	"certbot renew"
	- If you like Certbot, please consider supporting our work by:

	Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
	Donating to EFF:                    https://eff.org/donate-le
	```
	
	Note: You need to enter E-mail address, it used to notify you when the SSL certificate is expiring.

---

## Optional

1. Manually generate DHParam.pem

	```
	openssl dhparam -out dhparam.pem 4096
	```

Reference:

- [Odoo nginx reverse proxy](https://linuxize.com/post/configure-odoo-with-nginx-as-a-reverse-proxy/)