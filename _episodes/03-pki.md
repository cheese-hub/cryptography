---
title: "Public Key Infrastructure Lab"
teaching: 30
exercises: 60
questions:
- "What is Certificate Authority?"
- "How is PKI used in the web?"
- "How does HTTPS work?"
- "How do you sign certificates ?"
objectives:
- "Follow the instructions to become the root CA, sign certificates, and manage HTTPS websites in Apache server. "
keypoints:
- "OpenSSL and Apache web server has already been installed in the container."
---

## Public Key Infrastructure Lab

### Acknowledgment 
This lab was originally designed by [SEEDLabs](https://seedsecuritylabs.org/Labs_16.04/Crypto/Crypto_PKI/) and Dr. Wenliang Du.

### Lab Overview
Public key cryptography is the foundation of today’s secure communication, but it is subject to man-in-the- middle attacks when one side of the communication sends its public key to the other side. The fundamental problem is that there is no easy way to verify the ownership of a public key, i.e., given a public key and its claimed owner information, how do we ensure that the public key is indeed owned by the claimed owner?  
The Public Key Infrastructure (PKI) is a practical solution to this problem. 
The learning objective of this lab is for students to gain the first-hand experience on PKI. By doing the tasks in this lab, students should be able to gain a better understanding of how PKI works, how PKI is used to protect the Web, and how Man-in-the-middle attacks can be defeated by PKI. Moreover, students will be able to understand the root of the trust in the public-key infrastructure, and identify the problems that will arise when the root trust is broken. This lab covers the **following topics**:  
- Public-key encryption 
- Public-Key Infrastructure (PKI)
- Certificate Authority (CA) and root CA 
- X.509 certificate and self-signed certificate 
- Apache, HTTP, and HTTPS 
- Man-in-the-middle attacks

### Lab Tasks
#### Becoming a Certificate Authority (CA)
A Certificate Authority (CA) is a trusted entity that issues digital certificates. The digital certificate certifies the ownership of a public key by the named subject of the certificate. In this lab, we need to create digital certificates, but we are not going to pay any commercial CA. We will become a root CA ourselves and then use this CA to issue certificate for others. Unlike other certificates, which are usually signed by another CA, the root CA’s certificates are self-signed. Root CA’s certificates are usually pre-loaded into most operating systems, web browsers, and other software that rely on PKI. Root CA’s certificates are unconditionally trusted.   
#### The configuration file (openssl.cnf)
In order to use OpenSSL to create certificates, you have to have a configuration file. The configuration file usually has an extension .cnf. It is used by three OpenSSL commands: ca, req and x509. The configuration file is located in **/usr/lib/ssl/openssl.cnf** however, openssl.cnf has already been copied to **/home/user/demoCA/openssl.cnf** in the container. Openssl requires the following sub-directories however, the configuration has already been completed in the container. 
``` source
dir             = ./demoCA        # Where everything is kept
certs           = $dir/certs      # Where the issued certs are keptcrl_dir	        = $dir/crl        # Where the issued crl are kept
new_certs_dir   = $dir/newcerts   # default place for new certs.
database        = $dir/index.txt  # database index file.
serial          = $dir/serial     # The current serial number
```
For the index.txt file, an empty file has been created. For the serial file, a single number in string format (e.g. 1000) has been entered in the file.
> ## Openssl Configuration
> The required directories are configured under **demoCA** directory, which is located under 
> **/home/user** directory.  
> In order to understand how openssl is configured, please take a look at **demoCA** directory and compare them to
> openssl.cnf file.
{: .callout}

#### Certificate Authority (CA)
As we described before, we need to generate a self-signed certificate for our CA. This means that this CA is totally trusted, and its certificate will serve as the root certificate.  
Run the following command in the **demoCA** directory to generate a self-signed certificate for the CA:

~~~
openssl req -new -x509 -keyout ca.key -out ca.crt -config openssl.cnf
~~~
{: .language-bash}
You will be prompted for information and a password. **Do not lose this password**, because you will have to type the passphrase each time you want to use this CA to sign certificates for others. You will also be asked to fill in some information, such as the Country Name, Common Name, etc. The output of the command are stored in two files: **ca.key** and **ca.crt**. The file **ca.key** contains the CA’s **private key**, while **ca.crt** contains the **public-key certificate**.

#### Creating a Certificate for SEEDPKILab2018.com
Now that we have become a root CA, we are ready to sign digital certificates for our customers. Our first customer is a company called SEEDPKILab2018.com. In order for this company to get a digital certificate from a CA, it needs to go through three steps.  
##### Step 1: Generate public/private key pair 
The company needs to first create its own public/private key pair. We can run the following command to generate an RSA key pair (both private and public keys). You will also be required to provide a password to encrypt the private key (using the AES-128 encryption algorithm, as is specified in the command option). The keys will be stored in the file server.key:
~~~
openssl genrsa -aes128 -out server.key 1024
~~~
{: .language-bash}
The server.key is an encoded text file (also encrypted), so you will not be able to see the actual
content, such as the modulus, private exponents, etc. To see those, you can run the following command:
~~~
openssl rsa -in server.key -text
~~~
{: .language-bash}
##### Step 2: Generate a Certificate Signing Request (CSR)
Once the company has the key file, it should generates a Certificate Signing Request (CSR), which basically includes the company’s public key. The CSR will be sent to the CA, who will generate a certificate for the key (usually after ensuring that identity information in the CSR matches with the server’s true identity). Please use **SEEDPKILab2018.com** as the common name of the certificate request.
~~~
openssl req -new -key server.key -out server.csr -config openssl.cnf
~~~
{: .language-bash}
It should be noted that the above command is quite similar to the one we used in creating the self-signed certificate for the CA. The only difference is the *-x509* option. Without it, the command generates a request; with it, the command generates a self-signed certificate.












































