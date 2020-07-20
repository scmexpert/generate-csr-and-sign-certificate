# Generate CSR and Sign the Certificate

## Create Private Key 
  You can create a new private key with RSA-2048 encryption in the PEM format 
  
      'openssl genrsa -out <PRIVATE_KEY_FILE> 2048'
      
   NOTE : Replace PRIVATE_KEY_FILE with the path and filename for the new private key file.

## Create a CSR

After you have a private key, you can generate a certificate signing request (CSR) in the PEM format using OpenSSL. 

Your CSR must meet the following criteria:
* It must be in PEM format.
* It must have a common name (CN) or a subject alternative name (SAN) attribute. Practically speaking, your certificate should contain both CN and SAN attributes.

* Create an OpenSSL configuration file. In the following example, subject alternative names are defined in the [sans_list].

```
cat <<'EOF' >CONFIG_FILE
[req]
default_bits              = 2048
req_extensions            = extension_requirements
distinguished_name        = dn_requirements

[extension_requirements]
basicConstraints          = CA:FALSE
keyUsage                  = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName            = @sans_list

[dn_requirements]
countryName               = Country Name (2 letter code)
stateOrProvinceName       = State or Province Name (full name)
localityName              = Locality Name (eg, city)
0.organizationName        = Organization Name (eg, company)
organizationalUnitName    = Organizational Unit Name (eg, section)
commonName                = Common Name (e.g. server FQDN or YOUR name)
emailAddress              = Email Address

[sans_list]
DNS.1                     = SUBJECT_ALTERNATIVE_NAME_1
DNS.2                     = SUBJECT_ALTERNATIVE_NAME_2

EOF
```

## Create a certificate signing request (CSR) file. 

The command is interactive; you are prompted for attributes except for the subject alternative names, which you defined in the [sans_list] of the CONFIG_FILE in the previous step.

```
openssl req -new -key PRIVATE_KEY_FILE \
    -out CSR_FILE \
    -config CONFIG_FILE
```

## Sign the CSR

When a Certificate Authority (CA) signs your CSR, it uses its own private key to create a certificate:

If you manage your own CA, or if you want to create a self-signed certificate for testing, you can use the following OpenSSL command

```
openssl x509 -req \
    -signkey PRIVATE_KEY_FILE \
    -in CSR_FILE \
    -out CERTIFICATE_FILE \
    -days TERM
 ```

Replace the placeholders with valid values:

    PRIVATE_KEY_FILE: The path to the private key for your CA; if creating a self-signed certificate for testing, this private key is the same as the one used to create the CSR.
    CSR_FILE: The path to the CSR
    CERTIFICATE_FILE: The path to the certificate file to create
    TERM: The number of days, from now, during which the certificate should be considered valid by clients that verify it


**NOTE :**

Your self-managed SSL certificates can use a wildcard in the common name. 

For example, a certificate with the common name *.example.com. matches the hostnames www.example.com and foo.example.com, but not a.b.example.com or example.com. 

Certificates with wildcard fragments, such as f*.example.com, aren't supported.
