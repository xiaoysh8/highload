Let's Encrypt is a free, open and automated certification authority, which has issued more than 5 million of certificates. Key benefits include:

Free;
Automated for configuration and certificate updating;
Security — uses best TLS practices;
Transparance — public records about certificates receiving and reneval;
Open source for implemention in a large number of services and systems.
Let's Encrypt Nginx
Let's Encrypt provides a simplified for obtaining a certificate to enable encrypted connection with the Web server and website. Service is needed for implementing TLS. It is noteworthy that the process of obtaining and installing a certificate on Apache is fully automated, so we will look at the installation, configuration, and usage of the Let's Encrypt and Nginx on Ubuntu.

On the other hand, Let's Encrypt also has its disadvantages. First, the certificates are issued only for 90 days. Certificates for a longer period are not provided. Secondly, the service came out of beta status in production only 3 months young ago. Third, there's a large number of dependencies and plugins for obtaining the certificates. This in itself is not terrible, but when you have to renew the certificate every 3 months, there will be inevitable problems. In addition, Let's Encrypt is still in active development, so there may be bugs and errors.

If pros overweight the cons, then we're ready to install and configure the tool.

Installation

First you need to choose a Let's Encrypt client. The founders of the project recommend Certbot, but you can use any of the alternative clients.

Let's Encrypt is in the operating system packages, so the installation will be as simple as:

$ sudo aptitude install letsencrypt
# The new versions will contain it like a certbot

The client is available for various operating systems and web servers, and as the source:

wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto --help
# Uses existing and additional dependencies in Python virtual environment

Obtaining a certificate

Let's Encrypt issues certificates by using a variety of clients and methods. Unfortunately, a complete plug-in for Nginx is experimental, so not always works properly and may lead to crashes of the web server. If you are ready to face problems, it is necessary to assemble Certbot from a source on the GitHub, then install Nginx plugin and use it as --nginx option.

But at this point we recommend a Webroot plugin.

Using Webroot

The plugin is useful if you can independently change the files on the server or has your own production server. It creates a temporary file for each domain:

${webroot-path}/.well-known/acme-challenge
# Do not forget to configure the server to serve files in hidden directories

A validation server checks them in the form:

66.133.109.36 - - [05/Jan/2016:20:11:24 -0500] "GET /.well-known/acme-challenge/HGr8U1IeTW4kY_Z6UIyaakzOkyQgPr_7ArlLgtZE8SX HTTP/1.1" 200 87 "-" "Mozilla/5.0 (compatible; Let's Encrypt validation server; +https://www.letsencrypt.org)"
# It verifies that the DNS for the requested domain leads to the server with certbot

You may also want to configure Nginx virtual hosts file /etc/nginx/sites-available/default:

location ~ /.well-known {
                allow all;
        }
# Add to the server section with enabled TLS

Thereafter, you can obtain the certificate:

$ letsencrypt certonly --webroot -w /var/www/somesite -d somesite.com -d www.somesite.com -w /var/www/other -d other.somesite.net
# Obtaining certificates and placing them in needed directories

Let's take a look at this command:

letsencrypt certonly --webroot is a basic command for obtaining certificate with webroot;
-w specifies web root directory;
-d specifies domains for which you want to receive the certificate.
This starts the issuing process, you will need to specify e-mail for recovery, accept the terms. The message about the successful receipt will look like this:

Output:
IMPORTANT NOTES:
 - If you lose your account credentials, you can recover through
   e-mails sent to user@somesite.com
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/example.com/fullchain.pem. Your
   cert will expire on 2016-09-15. To obtain a new version of the
   certificate in the future, simply run Let's Encrypt again.
 - Your account credentials have been saved in your Let's Encrypt
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also, contain certificates and private keys obtained by Let's
   Encrypt so making regular backups of this folder is ideal.
 - If like Let's Encrypt, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
# Points to the certificate files, among the other data

A directory /etc/letsencrypt/live/your_domain_name will contain:

Cert.pem — domain certificate;
Chain.pem — Let's Encrypt certificate chain;
Fullchain.pem — cert.pem and chain.pem combination;
Privkey.pem — certificate private key.
Enabling TLS/SSL in Nginx

Now you can enable and optimize the TLS of Nginx. And do not forget to do a 301 redirect from the unprotected http:// to the secure https://. Additionally, you may want to implement the Diffie-Hellman key exchange protocol:

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
# Uses OpenSSL library, and a 2048-bit encryption

Automatic certificate renewal

Note that certificates are issued by Let's Encrypt only for 90 days. Therefore we recommend to configure automatic certificate renewal. To do this, put the command in the crontab or systemd:

sudo crontab -e
* * 30 * * letsencrypt renew >> /var/log/le-renew.log
# Automatic checking and certificate renewal every 30 days

The most important

Be sure to include and optimize TLS on the Web server. A Let's Encrypt will help obtain proven certificates fast and free.