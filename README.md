# Radicale Simple

This repository provides a debian package that ensures the installation of the
latest version of radicale. It then sets up the following:

* A systemd service unit that starts enabled.
* A unix user `radicale` under which radicale runs.
* A nginx reverse-proxy configuration.
* A default radicale configuration.
* An empty users file.

## Adding Users

This radicale package keeps its users file at `/var/radicale/users`. This is
done outside of `/etc/` because we assume that the users should be backed up in
the way way as `/var/radicale/collections`. Additionally this package sets up
radicale to use bcrypt password encryption. Therefore adding a user can be done
with the following command:

```bash
sudo  htpasswd -B -c /var/radicale/users {{username}}
```

## Adding SSL support.

This package does not configure radicale for the use of SSL. Instead it relies
on using Nginx as a reverse proxy. SSL should be terminated there. This can be
done by altering the proxy pass server to use SSL then adding a redirect server
on port 80. Example below:

```nginx
server {
	listen 80;
	listen [::]:80;

	server_name radicale.server.com;
	rewrite ^/(.*) https://radicale.server.com/$1;
}

server {
	listen 443 ssl;
	listen [::]:443 ssl;

	server_name radicale.server.com;

	ssl_certificate /etc/letsencrypt/live/server.com/cert.pem;
	ssl_certificate_key /etc/letsencrypt/live/server.com/privkey.pem;
	
	location / {
		proxy_pass http://127.0.0.1:5232/;
	}
}
```

## Uninstall

To protect user data, this package does not remove either the `radicale` user or
the content of `/var/radicale` on uninstall. Purging these from your system will
be a manual process.

