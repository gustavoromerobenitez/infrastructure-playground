# Example Generation of a CSR for a Wildcard Certificate with SAN's with OpenSSL

The TLS Certificate creation process involves the following steps:

- Generate a Private Key and store it safely
- Create an OpenSSL configuration for the wildcard certificate.
- Generate a *Certificate Signing Request* (CSR).
- Submit the CSR for signature to the an internal or external *Certification Authority* (CA).
- Download the signed public key (certificate) and store it in a secrets' management system.

## Toolset

To create the Private Key and the CSR, we will use the [OpenSSL](https://www.openssl.org/) tool,
widely available as a Linux package and also available for Windows.

## Private Key

First we need to create a strong private key compatible with the company's security policies. 
An RSA key with a 2048 bit encryption key should be enough. 

To create the private key, you can do so from the system that will host the certificate. 
In complex scenarios, like machine clusters (GKE), you may create the private key locally
as long as you store it safely in a secret's management system and remove any local copies.

In the following example, we will store the key in a file called _subdomain.domain.com.key_

```
openssl genrsa -out subdomain.domain.com.key 2048
```

## OpenSSL configuration

In order to generate the right certificate, we need to provide a number of configuration parameters to the openssl tool.

The most important parts are the encription key bits (default_bits), the certificate Distinguished Name or DN (req_distinguished_name)
and the Subject Alternative Names or SANs (alt_names).

Here's an example config file for _*.subdomain.domain.com_:

```
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = req_distinguished_name
req_extensions     = v3_req

[ req_distinguished_name ]
CN = *.subdomain.domain.com
O = Company Org
OU = Company OU
L = Location
ST = State
C = Country

[ v3_req ]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = *.subdomain.domain.com
DNS.2 = *.alias-domain.com
DNS.3 = *.sub-subdomain.another-subdomain.domain.com
```

## Generate the CSR

```
openssl req -config subdomain.domain.com.csr.cnf -new -key subdomain.domain.com.key -nodes -out subdomain.domain.com.csr
```

## Display and Verify the CSR

```
openssl req -text -noout -verify -in subdomain.domain.com.csr
```

Which will display the following information about the Certificate Signing Request.

```
verify OK
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN = *.subdomain.domain.com, O = Company Org, OU = Company OU, L = Location, ST = State, C = Country
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus: [...]                    
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Key Usage:
                Key Encipherment, Data Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Subject Alternative Name:
                DNS:subdomain.domain.com, DNS:alias-domain.com, DNS:sub-subdomain.subdomain.domain.com
    Signature Algorithm: sha256WithRSAEncryption
         [...]
```
