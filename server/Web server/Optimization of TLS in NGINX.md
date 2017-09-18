Optimization of TLS in NGINX

TLS (Transport Layer Security) is a protocol for protecting Web pages that has replaced SSL. In fact, TLS = SSL is the next generation (version) of the obsolete and prone to POODLE-attacks standard. TLS / SSL

TLS is able not only to encrypt data, but also to authenticate users on the server and check information for integrity. The protocol is mandatory for use by commercial sites and organizations that want to confirm their authenticity. Well and ensuring the privacy of users.

To install a secure channel, use the TLS Handshake protocol, you also need to obtain a certificate from the CA and load the crypto library, OpenSSL in this case.

To enable TLS and configure the HTTPS server in Nginx, you must include the SSL parameter in the server section:

server {
    listen              443 ssl;
    server_name         example.com;
    ssl_certificate     example.com.crt;
    ssl_certificate_key example.com.key;
    ssl_protocols       TLSv1.1 TLSv1.2;
    ssl_ciphers AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;
    ...
}
# SSL on port 443, certificate, key, TLS protocols and ciphers used

Here, the ssl_protocols directive restricts the protocols used, and ssl_ciphers describes the allowed ciphers in the format supported by the OpenSSL library. For example:

ssl_ciphers ALL:!aNULL:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
# example of using all ciphers in the OpenSSL format

It is important to understand that TLS / SSL operations are quite resource intensive, so even several workflows on multiprocessor systems are required. The most "heavy" and in terms of load, and on the time of execution, is the operation TLS Handshake. Therefore, the best solution would be to optimize the session: persistent connections, caching, static keys, and enabling OCSP Stapling.

Keepalive

The use of permanent connections makes it possible to process several requests in the conditions of a single connection. TLS keepalive

To do this, add the following to the server section of the configuration file:


server {

...
keepalive_timeout   70;

…

}
#including keepalive

Session Caching

Caching is necessary to re-use the keys, so as not to repeat the handheld. TLS caching

To do this, in the http section of the Nginx configuration file, add:


http {
    ssl_session_cache   shared:SSL:100m;
    ssl_session_timeout 1h;
…
}
# Enable a shared cache of 100 MB and a session timeout of 1 hour

In 1 MB cache is placed about 4000 sessions, so the presented example will store up to 400,000 sessions.

Session Tickets or session mandate

TLS can use session tickets to resume a session, provided that the client supports them (browsers of the Chromium family and Firefox). To this end, the TLS server sends the client a session state, encrypting it with its own key, and the key identifier. The client resumes the secure session by sending the last ticket to the server during the initialization of the TLS Handshake procedure. And the server, in turn, resumes the session in accordance with the saved parameters. TLS session tickets

Everything is very simple. In the Nginx configuration in the already familiar server section , add:


server {

…
ssl_session_ticket_key current.key;
ssl_session_ticket_key prev.key;
ssl_session_ticket_key prevprev.key;

…
}
# enable static keys

OCSP Stapling

Online Certificate Status Protocol - mechanism for verifying the relevance of the SSL certificate, which replaced the less fast CRL (Certificate Revocation List). When using CRL, the browser downloads a list of revoked certificates and checks the current certificate, which increases the connection time. Whereas when using OCSP, the browser sends a test request to the OCSP address and in response receives a certificate status, which can heavily burden the certification authority server. OCSP Stapling

To use the protocol, OCSP Stapling is used - the owner of the certificate interrogates the OCSP server at a certain interval and caches a response that contains an electronic signature. The answer itself "stitches" with TLS Handshake through the extension of the Certificate Status Request. So, the servers of CAs do not receive a huge number of requests, which also contain information about the user's views.

To enable OCSP Stapling, you need to add a few lines of code to the Nginx configuration file:


server {

…
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/nginx/ssl/example.com.crtt;
…
}
# Enabling OCSP Stapling and connecting a trusted certificate

Additional functions

To force the browser to use the HTTPS protocol, there is a mechanism called Strict Transport Security:


add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
# including Strict-Transport-Security, age in seconds

With the help of this function, the browser will understand that you need to use the HTTPS protocol even by going to the HTTP link. This will help prevent some attacks, especially if the server does not have a redirect from HTTP to HTTPS.

The most important thing

Optimizing TLS does not take much time, but it will significantly speed up the protocol and connect users to secure sites. And the introduction of OCSP Stapling and HSTS will also increase the security of the connection.