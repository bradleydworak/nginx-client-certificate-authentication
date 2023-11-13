# nginx-client-certificate-authentication
## Install and setup NGINX client-certificate authentication on Ubuntu 22.04

These instructions will create certificates using highly secure RSA 4096-bit key lengths using the AES Rijndael 256-bit cipher. I recommend this implementation over elliptic curve if security is a priority over encryption/decryption speed.

### Step 1: Generate Certificate Authority (CA) root certificate

`openssl genrsa -aes256 -passout stdin -out certificate_authority.pass.key 4096`

`openssl rsa -passin stdin -in certificate_authority.pass.key -out certificate_authority.key`

`rm certificate_authority.pass.key`

`openssl req -new -x509 -days 3650 -key certificate_authority.key -out certificate_authority.pem`

### Step 2: Generate Client Certificate Signing Request (CSR)

`openssl genrsa -aes256 -passout stdin -out client.example.com.pass.key 4096`

`openssl rsa -passin stdin -in client.example.com.pass.key -out client.example.com.key`

`rm client.example.com.pass.key`

`openssl req -new -out client.example.com.csr -key client.example.com.key`

### Step 3: Generate Client Certificate signed by CA root certificate

`openssl x509 -sha512 -req -days 3650 -in client.example.com.csr -CA certificate_authority.pem -CAkey certificate_authority.key -set_serial 01 -out client.example.com.pem`

### Step 4: Combine the Client Key, Client Certificate, and CA root certificate into a single PEM file

`cat client.example.com.key client.example.com.pem certificate_authority.pem > client.example.com.full.pem`

You can view the signed certification to help verify that this action has been performed by:
`openssl x509 -in client.example.com.full.pem -text`

### Step 5: Convert PEM file into a PKCS #12 archive file format

Note: You will need to maintain a copy of the import password when importing this in the brower settings

`openssl pkcs12 -export -out client.example.com.full.pfx -inkey client.example.com.key -in client.example.com.full.pem -certfile certificate_authority.pem`

### Step 6: Add the following lines to your NGINX configuration file

Note: Restart NGINX after updating configuration file.

```
File: nginx.conf
Section: Server block

ssl_client_certificate /path/to/file/client.example.com.pem;
ssl_verify_client on;
```

### Step 7: Import client certificate into browser

Note: Must use import password generated in Step 5 above.
