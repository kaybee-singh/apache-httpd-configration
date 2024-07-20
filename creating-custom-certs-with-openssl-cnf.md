

# Create Self singed certificates

1. Install **openssl** package and create CA config file.
````bash
yum install openssl -y
mkdir -p ~/ca/{newcerts,private,cacerts};cd ~/ca
touch index.txt
echo 1000 > serial

vim ca_openssl.cnf

[ ca ]
default_ca = my_ca

[ my_ca ]
dir               = /root/ca
database          = $dir/index.txt
new_certs_dir     = $dir/newcerts
certificate       = $dir/cacerts/ca.pem
serial            = $dir/serial
private_key       = $dir/private/cakey.pem
default_md        = sha256
policy            = my_policy

[ my_policy ]
countryName             = optional
stateOrProvinceName     = optional
organizationName        = optional
commonName              = supplied

[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
C  = US
ST = State
L  = City
O  = Organization
OU = Organizational Unit
CN = my-ca
[ v3_ca ]
basicConstraints = CA:TRUE
keyUsage = critical, cRLSign, keyCertSign

````
2. Generate CA private key and self signed cert.

````bash
openssl genrsa -out ~/ca/private/cakey.pem 2048
openssl req -new -x509 -config ~/ca/ca_openssl.cnf -key ~/ca/private/cakey.pem -out ~/ca/cacerts/ca.pem -days 3650 -extensions v3_ca
````

3. Create the server CSR to get signed by the CA created in above steps.

````bash

mkdir -p ~/servercerts/{private,certs};cd ~/servercerts
````
4. In the server configuration file specify replace the **req_distinguished_name** parameters with your parameters, for example **C(country)**, **ST(State, province)**.

````bash
vim ~/servercerts/server-openssl.cnf
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
C  = US
ST = State
L  = City
O  = Organization
OU = Organizational Unit
CN = yourdomain.com
````
5. Now create the KEY and CSR.
````bash
openssl genrsa -out ~/servercerts/private/serverkey.pem 2048
openssl req -new -key ~/servercerts/private/serverkey.pem -out ~/servercerts/server.csr -config server-openssl.cnf
````
6. Sign the CSR by CA.
````bash
openssl ca -config ~/ca/ca_openssl.cnf -in ~/servercerts/server.csr -out ~/servercerts/certs/server.crt -days 365

````
7. Above created **serverkey.pem**, **server.crt** and **ca.pem** can be used with web server or other applications.
````bash
# ls -l ~/servercerts/{certs,private} ~/ca/cacerts/
/root/ca/cacerts/:
total 4
-rw-r--r--. 1 root root 1338 Jul 20 13:50 ca.pem

/root/servercerts/certs:
total 4
-rw-r--r--. 1 root root 3888 Jul 20 13:51 server.crt

/root/servercerts/private:
total 4
-rw-------. 1 root root 1704 Jul 20 13:48 serverkey.pem

`````
