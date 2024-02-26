# self-signed-certificate
This repository contains steps to create self-signed-certificate using OpenSSL

# What is a Self Signed Certificate?
A self-signed certificate is an SSL/TSL certificate not signed by a public or private certificate authority. Instead, 
it is signed by the creator’s own personal or root CA certificate.

# Here is what we do to request paid SSL/TLS certificate from a well-known Certificate Authority.
 -> Create a certificate signing request (CSR) with a private key. A CSR contains details about location, organization, and FQDN (Fully Qualified Domain Name).

 -> Send the CSR to the trusted CA authority.

 -> The CA authority will send us the SSL certificate signed by their root certificate authority and private key.

 -> We can then validate and use the SSL certificate with our applications.

# For a self-signed certificate, here is what we do.
 ->  Create our own root CA certificate & CA private key (We act as a CA on our own)
  
 -> Create a server private key to generate CSR
  
 -> Create an SSL certificate with CSR using our root CA and CA private key.
  
 -> Install the CA certificate in the browser or Operating system to avoid security warnings.

  # Steps to create Create Certificate Authority(CA)
  # 1. create a directory to store all the required files(this is optional we can use any directory)
     -> mkdir openssl && cd openssl
  # 2. Execute the following openssl command to create the rootCA.keyand rootCA.crt
     -> openssl req -x509 \ -sha256 -days 356 \ -nodes \ -newkey rsa:2048 \ -subj "/CN=localhost/C=IN/L=Noida" \ -keyout rootCA.key -out rootCA.crt
     
    # Explanation
    -x509: This option specifies that a self-signed certificate is to be generated.

    -sha256: This option specifies the hash function to use, in this case, SHA-256.

    -days 356: This option sets the validity period of the certificate in days. In this example, the certificate is valid for 356 days.

    -nodes: This option specifies not to encrypt the private key. If this option is not used, it will prompt for a passphrase to protect the private key.

    -newkey rsa:2048: This option generates a new RSA private key of 2048 bits.

    -subj "/CN=localhost/C=IN/L=Noida": This option sets the subject of the certificate. Here, it sets the Common Name (CN) to "localhost," the Country (C) to "IN" (India), and the       
           Location (L) to "Noida."

    -keyout rootCA.key: This option specifies the file where the private key will be saved.

    -out rootCA.crt: This option specifies the file where the generated certificate will be saved.

    # We will use the rootCA.keyand rootCA.crt to sign the SSL certificate.

  # Create Self-Signed Certificates using OpenSSL
  1. Create the Server Private Key.
     -> openssl genrsa -out server.key 2048
    
  2. Create Certificate Signing Request Configuration
     We will create a csr.conf file with the cotent given below to have all the information to generate the CSR.
     
     cat > csr.conf <<EOF 
     [ req ] 
      default_bits = 2048 
      prompt = no 
      default_md = sha256 
      req_extensions = req_ext 
      distinguished_name = dn 

      [ dn ] 
      C = IN (country)
      ST = U.P (state)
      L = Noida (locality)
      O = <organization_name> 
      OU = <organization_unit> 
      CN = localhost (Common Name (the domain for which the certificate is being issued).)

      [ req_ext ] 
      subjectAltName = @alt_names 

      [ alt_names ] 
      DNS.1 = localhost
      IP.1 = 127.0.0.1

      EOF

  3. Generate Certificate Signing Request (CSR)(server.csr) Using Server Private Key
        openssl req -new -key server.key -out server.csr -config csr.conf
  
  4. Create a external file
      cat > cert.conf <<EOF
      authorityKeyIdentifier=keyid, issuer 
      basicConstraints=CA:FALSE 
      keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment 
      subjectAltName = @alt_names 

      [alt_names] 
      DNS.1 = localhost 

      EOF

     # Explanation

     authorityKeyIdentifier:
     This field identifies the public key to be used to verify the signature on this certificate. The keyid and issuer options specify that the key identifier          should be used,         and it should be derived from the issuer's public key.

     basicConstraints:
     This field defines whether the certificate is a Certificate Authority (CA) or not. CA:FALSE indicates that this certificate is not a CA.

     keyUsage:
     This field specifies the key usage purposes for which the public key can be used. In this case, the allowed usages are digitalSignature, nonRepudiation, keyEncipherment, and              dataEncipherment.

     subjectAltName:
     This field allows you to specify alternative names for the subject of the certificate. In this case, you are using the @alt_names section to define additional names.
     [alt_names]:

  This section defines alternative names (subject alternative names). Here, we have specified DNS.1 = localhost, indicating that localhost is a valid alternative DNS name for the           subject.This configuration suggests that the certificate is intended for use with a service running on localhost. The SAN field is particularly useful when you want the certificate to    be valid for multiple domain names or IP addresses.



5. Generate SSL certificate With self signed CA
   -> openssl x509 -req -in server.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out server.crt -days 365 -sha256 -extfile cert.conf

# Conversions 
Convert crt to pem
  -> openssl x509 -outform pem -in rootCA.crt -out rootCA.pem

Convert crt to der
  -> openssl x509 -in rootCA.crt -out rootCA.der -outform DER
  
Convert pem to crt
  -> openssl x509 -in rootCA.pem -inform PEM -out rootCA.crt
  
Convert pem to der
  -> openssl x509 -outform der -in rootCA.pem -out rootCA.der
   
Combine Certificate and Private Key into a PKCS12 Keystore:
  -> openssl pkcs12 -export -out server.p12 -inkey server.key -in server.crt

# Configure Spring Boot Application
  server.port=8443
  server.ssl.key-store-type=PKCS12
  server.ssl.key-store= <server.p12 file-location/server.p12>
  server.ssl.key-store-password= <your-export-password>
  server.ssl.key-alias=1  # This is usually the default alias for the first key in the keystore



