# Challenge
Contains the files for the Linux and Docker tasks
1. Installing Node Version manager which installs node

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
nvm install node
```

2. Installing MongoDB

```
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl start mongod
```
3. Installing Apache webserver

```
sudo apt update
sudo apt install apache2 -y
```

4.Create a direcotry inside apache server directory

```
mkdir -p var/www/html/nodeapp
cd var/www/html/nodeapp
vi app.js
node app.js &
```
app.js looks like

```
const express = require("express");
const http = require("http");
const app = express();
const server = http.createServer(app);
app.get('/', function(req,res){
        res.send("Hello World");
});
server.listen(3000, function(){
        console.log("Server is listening on port: 3000");
});
```

5. run Node app and see webserver is running at localhost
```
node app.js & 
 ```
 >node app running at localhost:3000

6. Now lets enable couple of modules which will make our apache as proxy in the host
```
sudo a2enmod ssl
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_html
sudo systemctl restart apache2
```

7. Now edit the virtual host config file using `sudo nano /etc/apache2/sites-available/000-default.conf` and add below content
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        ProxyPass / http://127.0.0.1:3000/
        ProxyPassReverse / http://127.0.0.1:3000/
        ProxyPreserveHost On
        Redirect "/" "https://127.0.0.1:3000/"
</VirtualHost>
```

8. See that our node app is now running in apache webserver at localhost
9. Lets create some SSL certificates and self sign them using openssl command. You need to enter couple of details as prompted and shown below. 
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```

```
Country Name (2 letter code) [XX]:DE
State or Province Name (full name) []:Example
Locality Name (eg, city) [Default City]:Example 
Organization Name (eg, company) [Default Company Ltd]:Example Inc
Organizational Unit Name (eg, section) []:Example Dept
Common Name (eg, your name or your server's hostname) []:your_domain_or_ip
Email Address []:webmaster@example.com
```

10.Now the certificates are stored at ssl folder. We need to add these references to virtual hosts file 
`sudo nano /etc/apache2/sites-available/default-ssl.conf.`

```
<VirtualHost *:443>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ProxyPass / http://127.0.0.1:3000/
        ProxyPassReverse / http://127.0.0.1:3000/
        ProxyPreserveHost On
        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
        SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

</VirtualHost>
```

11. Enable the modules and restart apache once again
```
sudo a2enmod headers
sudo a2ensite default-ssl
sudo a2enconf ssl-params
sudo systemctl restart apache2
```

12. See that the helloworld webpage is now encrypted and can be accessed from https://localhost.
