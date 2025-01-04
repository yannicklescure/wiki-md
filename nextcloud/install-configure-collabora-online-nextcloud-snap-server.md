# How to Install and Configure Collabora Online on a Separate Server for Nextcloud Snap

In this guide, we’ll walk through installing Collabora Online on a separate Ubuntu server and
integrating it with a Nextcloud Snap instance. This setup allows both Nextcloud and Collabora to run
on the same network without conflict. The process includes configuring Apache, setting up firewall
rules, and securing Collabora with SSL. Here's how to do it.

## 1. Collabora Online installation

### Add Collabora Repository

- Import the Collabora CODE signing key:

  ```bash
  cd /usr/share/keyrings
  sudo wget https://collaboraoffice.com/downloads/gpg/collaboraonline-release-keyring.gpg
  ```

- Create a file for the Collabora CODE package repository:

  ```bash
  sudo nano /etc/apt/sources.list.d/collaboraonline.sources
  ```

- Add the repository configuration:

  ```bash
  Types: deb
  URIs: https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb
  Suites: ./
  Signed-By: /usr/share/keyrings/collaboraonline-release-keyring.gpg
  ```

- Install the CODE packages:

  ```bash
  sudo apt install coolwsd code-brand
  ```

## Configure Collabora Online WebSocket Daemon

By default, coolwsd enables TLS connection. However, it didn’t find a TLS certificate file, hence
the start failure. It’s better to disable TLS in coolwsd and terminate TLS at a reverse proxy. The
coolwsd configuration file is located at `/etc/coolwsd/coolwsd.xml`. However, it’s an XML file,
which is not easy to read and edit. We can use the `coolconfig` tool to change configurations.

- Create a bash script `collabora.sh` that you'll place into `/etc/init.d/` folder.

  ```bash
  sudo nano /etc/init.d/collabora.sh
  ```

  ```bash
  coolconfig set ssl.enable false
  coolconfig set ssl.termination true
  coolconfig set net.proto IPv4
  coolconfig set net.listen 127.0.0.1
  coolconfig set storage.wopi.host cloud.yourdomain.com # nextcloud url
  # (Option) If you use several domains
  # coolconfig set storage.wopi.alias_groups[@mode] groups
  # coolconfig set storage.wopi.alias_groups.group[0].host[@allow] true
  # coolconfig set storage.wopi.alias_groups.group[0].host cloud.yourdomain.com
  # coolconfig set storage.wopi.alias_groups.group[0].alias[0] https://cloud.yourdomain1.com
  # coolconfig set storage.wopi.alias_groups.group[0].alias[1] https://collabora.yourdomain.com
  # coolconfig set storage.wopi.alias_groups.group[0].alias[2] https://collabora.yourdomain1.com
  systemctl restart coolwsd
  sleep 10
  systemctl status coolwsd

  exit 0
  ```

- Make the script executable

  ```bash
  sudo chmod +x /etc/init.d/collabora.sh
  ```

- Add a cron job (as root)

  ```bash
  sudo crontab -e
  ```

  At the end of the file, add the following instruction:

  ```bash
  @reboot /etc/init.d/collabora.sh
  ```

- Test if everything works as expected:

  ```bash
  sudo /etc/init.d/./collabora.sh
  ```

These steps ensure that changes made to Collabora persist after each update.

## 2. Configure Apache as Reverse Proxy for Collabora

Set up a reverse proxy configuration for Apache by creating a virtual host file:

```bash
sudo nano /etc/apache2/sites-available/collabora.yourdomain.com.conf
```

### Configure SSL and point to the Collabora service

Since Nextcloud Snap typically uses port **80** for HTTP, we need to configure Collabora to listen
on **port 8443** for secure connections.

You can reference the official Apache documentation
[here](https://httpd.apache.org/docs/2.4/bind.html) for more details on how to modify the Apache
listening ports.

```bash
<VirtualHost *:8080>
  ServerName collabora.yourdomain.com
  ServerAlias collabora.yourdomain1.com

  Redirect permanent / https://collabora.yourdomain.com:8443/
  RewriteEngine on
  RewriteCond %{SERVER_NAME} =collabora.yourdomain.com [OR]
  RewriteCond %{SERVER_NAME} =collabora.yourdomain1.com
  RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

<VirtualHost *:8443>
  ServerName collabora.yourdomain.com
  ServerAlias collabora.yourdomain1.com

  SSLEngine on
  SSLCertificateFile /etc/letsencrypt/live/yourdomain.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/yourdomain.com/privkey.pem

  Options -Indexes

  ErrorLog "/var/log/apache2/collabora_error"
  # Encoded slashes need to be allowed
  AllowEncodedSlashes NoDecode

  # keep the host
  ProxyPreserveHost On

  Header always unset X-Frame-Options

  # static html, js, images, etc. served from coolwsd
  # loleaflet/browser is the client part of Collabora Online
  ProxyPass           /loleaflet http://127.0.0.1:9980/loleaflet retry=0
  ProxyPassReverse    /loleaflet http://127.0.0.1:9980/loleaflet
  ProxyPass           /browser http://127.0.0.1:9980/browser retry=0
  ProxyPassReverse    /browser http://127.0.0.1:9980/browser

  # WOPI discovery URL
  ProxyPass           /hosting/discovery http://127.0.0.1:9980/hosting/discovery retry=0
  ProxyPassReverse    /hosting/discovery http://127.0.0.1:9980/hosting/discovery

  # Capabilities
  ProxyPass           /hosting/capabilities http://127.0.0.1:9980/hosting/capabilities retry=0
  ProxyPassReverse    /hosting/capabilities http://127.0.0.1:9980/hosting/capabilities

  # Main websocket
  ProxyPassMatch "/cool/(.*)/ws$" ws://127.0.0.1:9980/cool/$1/ws nocanon

  # Admin Console websocket
  ProxyPass   /cool/adminws ws://127.0.0.1:9980/cool/adminws

  # Download as, Fullscreen presentation and Image upload operations
  ProxyPass           /cool http://127.0.0.1:9980/cool
  ProxyPassReverse    /cool http://127.0.0.1:9980/cool
</VirtualHost>
```

The `privkey.pem` and `fullchain.pem` files will be generated in the next step.

## 3. Secure with SSL

To secure the Collabora server, use **Let's Encrypt** to generate an SSL certificate. Since you are
running multiple services on the same network, DNS-based validation is recommended to avoid issues
with ports.

- Install Certbot for Let's Encrypt:

  ```bash
  sudo apt install certbot python3-certbot-apache
  ```

- Get an SSL certificate for your domain:

  ```bash
  sudo certbot --apache -d collabora.yourdomain.com
  ```

Full instructions available in this
[guide](https://www.digitalocean.com/community/tutorials/how-to-acquire-a-let-s-encrypt-certificate-using-dns-validation-with-acme-dns-certbot-on-ubuntu-18-04)
to generate a certificate using DNS validation and **acme-dns-certbot**.

## 4. Allow Firewall Access

To allow traffic on the required ports, configure the **UFW** (Uncomplicated Firewall) on your
Ubuntu server to open ports **8080** (for HTTP) and **8443** (for HTTPS).

- Allow the ports with UFW:

  ```bash
  sudo ufw allow 8080
  sudo ufw allow 8443
  ```

- Reload UFW for the changes to take effect:

  ```bash
  sudo ufw reload
  ```

You can refer to this article on
[how to set up a firewall with UFW](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)
for more guidance.

## 5. Restart Apache and Collabora

- Save and close the file. To be able to proxy traffic using Apache, we need to enable some Apache
  modules.

  ```bash
  sudo a2enmod proxy proxy_wstunnel proxy_http ssl
  ```

- Enable this virtual host with the following command:

  ```bash
  sudo a2ensite collabora.yourdomain.com.conf
  ```

- Restart Apache and Collabora

  ```bash
  sudo systemctl restart apache2 coolwsd
  ```

## 6. Port Forwarding on Router

If you have both Nextcloud and Collabora running behind the same public IP, you'll need to configure
**port forwarding** on your router.

- Forward port **80** to the server running Nextcloud.
- Forward port **8443** to the server running Collabora.

This will ensure that traffic is correctly routed to the appropriate server for each service.

## 7. Configure Collabora in Nextcloud

1. Go to **Administration settings** → **Office**.
2. Select `Use your own server` and enter the domain name of your Collabora Online
   (`https://collabora.yourdomain.com:8443`), including https:// prefix, then click Save button.
3. In the advance settings, you can also set OOXML as the default format, so the files will be
   compatible with Microsoft Office software.

## Conclusion

This setup allows you to run Collabora and Nextcloud on different servers with the same public IP.
By configuring Apache to listen on port **8443** for Collabora, opening the necessary ports in the
firewall, and securing the connection with SSL, you ensure that both services can function without
conflicts.

<hr />

### Sources

- [https://www.linuxbabe.com/ubuntu/integrate-collabora-onlinenextcloud-without-docker](https://www.linuxbabe.com/ubuntu/integrate-collabora-onlinenextcloud-without-docker)
- [https://httpd.apache.org/docs/2.4/bind.html](https://httpd.apache.org/docs/2.4/bind.html)
- [https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu)
- [https://www.digitalocean.com/community/tutorials/how-to-acquire-a-let-s-encrypt-certificate-using-dns-validation-with-acme-dns-certbot-on-ubuntu-18-04](https://www.digitalocean.com/community/tutorials/how-to-acquire-a-let-s-encrypt-certificate-using-dns-validation-with-acme-dns-certbot-on-ubuntu-18-04)
- [https://musaamin.web.id/how-to-install-collabora-office-ubuntu2404/](https://musaamin.web.id/how-to-install-collabora-office-ubuntu2404/)
- [https://sdk.collaboraonline.com/docs/installation/Configuration.html](https://sdk.collaboraonline.com/docs/installation/Configuration.html)

</div>
