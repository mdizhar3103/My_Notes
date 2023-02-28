### Using openSSL Certificate Server
**Certificate Authority (CA):** This is just a public and private key pair.
- Clients only need to trust the CA
- Servers request x509 certificate from the CA
- Servers present their own public key to make the signed certificate
- **Intermediary CAs** may be used within the path. CA -> Signs I-CA Public Key -> which in turn signs incoming server requests.


```bash
mkdir -m 700 mi-certs
cd mi-certs

# generating certificates
openssl genrsa -des -out mi_ca.key 4096
    # -des: encrypt password provided for private key
    # -nodes: no password
ls -l
file mi_ca.key    

# Creating the certificate from this private key
openssl req -x509 -new -key mi_ca.key -sha256 -days 365 -out mi_ca.crt 
        |                               |
        +--> request                    +---> algo to use
        # enter the details for prompt 
        # Enter Common Name carefully

ls -l
file my_ca.crt

# copying the certificates to trusted the location
cp mi_ca.crt /etc/pki/ca-trust/source/anchors/
ls -l /etc/pki/ca-trust/extracted/

update-ca-trust extract
```

#### Generating a CSR (Certificate Signing Request)
```bash
# Note: using ubuntu system
openssl genrsa -out izhar.key 
openssl req -new -key izhar.key -out izhar.csr
    # enter the details prompted for carefully
file izhar.csr
```
