
# Create Self singed certificates

1. Install openssl package and create the configuration file 
````bash
yum install openssl -y
mkdir ~/certs;cd ~/certs
````
2. Now create the KEY and CSR
````bash
openssl genrsa -out serverkey.pem 2048
openssl req -new -key serverkey.pem -out server.csr

ls -l ~/certs
total 8
-rw-r--r--. 1 root root 1037 Jul 20 12:24 server.csr
-rw-------. 1 root root 1704 Jul 20 12:23 serverkey.pem

````
3. Create the CA 

````bash
mkdir ~/ca;cd ~/ca
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt

ls -l
total 8
-rw-r--r--. 1 root root 2025 Jul 20 12:45 ca.crt
-rw-------. 1 root root 3268 Jul 20 12:44 ca.key

````
4. Now CA is created, so lets sign the CSR created in first step by the CA created in second steps
````bash
cp ~/certs/server.csr ~/ca;cd ~/ca
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key  -out server.crt -days 365
cp server.crt ~/certs
````
5. Above created **serverkey.pem**, **server.crt** and **ca.crt** can be used with web server and other applications.
````bash
ls -l ~/certs/
total 12
-rw-r--r--. 1 root root 1627 Jul 20 12:25 server.crt
-rw-r--r--. 1 root root 1037 Jul 20 12:24 server.csr
-rw-------. 1 root root 1704 Jul 20 12:23 serverkey.pem

`````

6. To import the CA cert to system so that we do not get the SSL self signed warning you can import the certificate to Anchors.

7. Before importing the certificate we get error while running curl command. In below output myblog.com is my webserver.
````bash
# curl https://myblog.com
curl: (60) SSL certificate problem: self-signed certificate in certificate chain
More details here: https://curl.se/docs/sslcerts.html
````
8. Now copy the cert and then run the curl command.
````bash
cp ca.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust 
curl https://myblog.com
# curl https://myblog.com
<html>
	<h1>	welcome to test server </h1>
	</html>
````


````
