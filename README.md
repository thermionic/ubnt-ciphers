# ubnt-ciphers
After implementing letsencrypt from https://github.com/photinus/ubnt-letsencrypt you might also want to improve the rating on 
https://www.ssllabs.com 

Updating the cipher list and adding the intermediate should get an A

Start by connecting to your ER/ERL via SSH and become root 
```bash
sudo -i
```
get the letsencrypt intermediate certificate
```bash
cd /etc/lighttpd
curl -O https://letsencrypt.org/certs/lets-encrypt-x1-cross-signed.pem
```
now edit the file that creates the server conf
change 
```bash
ssl_ciphers='ECDSA aRSA+HIGH !3DES +kEDH +kRSA !kSRP !kPSK !NULL !RC4'
```
to
```bash
ssl_ciphers='ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES256-GCM-SHA384 ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES128-SHA ECDHE-RSA-AES256-SHA384 ECDHE-RSA-AES256-SHA AES128-GCM-SHA256 AES256-GCM-SHA384 AES128-SHA256 AES128-SHA AES256-SHA256 AES256-SHA'
```
and now add the intermediate
```bash
\$SERVER["socket"] == "0.0.0.0:$port" {
        ssl.engine  = "enable"
        ssl.use-sslv3 = "disable"
        ssl.pemfile = "/etc/lighttpd/server.pem"
        $ssl_cipher_list
}

\$SERVER["socket"] == "[0::0]:80" { }

\$SERVER["socket"] == "[0::0]:$port" {
        ssl.engine  = "enable"
        ssl.use-sslv3 = "disable"
        ssl.pemfile = "/etc/lighttpd/server.pem"
        $ssl_cipher_list
}
```
to 
```bash
\$SERVER["socket"] == "0.0.0.0:$port" {
        ssl.engine  = "enable"
        ssl.use-sslv3 = "disable"
        ssl.pemfile = "/etc/lighttpd/server.pem"
        ssl.ca-file = "/etc/lighttpd/lets-encrypt-x1-cross-signed.pem"
        $ssl_cipher_list
}

\$SERVER["socket"] == "[0::0]:80" { }

\$SERVER["socket"] == "[0::0]:$port" {
        ssl.engine  = "enable"
        ssl.use-sslv3 = "disable"
        ssl.pemfile = "/etc/lighttpd/server.pem"
        ssl.ca-file = "/etc/lighttpd/lets-encrypt-x1-cross-signed.pem"
        $ssl_cipher_list
}
```
now reboot the ER/ERL so that /etc/lighttpd/conf-enabled/10-ssl.conf is recreated
