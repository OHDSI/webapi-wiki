# Using OpenSSL to create an SSL Certificate

This page will detail how to use [OpenSSL](https://www.openssl.org/) to create a Certificate Authority and how to generate a Secure Socket Layer (SSL) certificate for use with a web server such as Apache Tomcat. This is useful when you need to use SSL on a local development machine. As a general rule of thumb, if you need an SSL certificate for use on either a production or non-production instance, you should contact your organization's IT team to obtain a certificate that will work properly in your environment.

Before moving ahead, please make sure that you have created your keystore and CSR (Certificate Signing Request) as detailed in the steps [[SSL Configuration In Tomcat]]. Please note the passwords that you use for each step as they will be required in the steps below.

## Creating a Certificate Authority

The following link describes the steps for creating a [Certificate Authority using OpenSSL](https://jamielinux.com/docs/openssl-certificate-authority/introduction.html). Read the introduction and follow the instructions that are detailed in [create the root pair](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html) and then [create the intermediate pair](https://jamielinux.com/docs/openssl-certificate-authority/create-the-intermediate-pair.html). For those using Windows, please note that you will need to modify the configuration files (openssl.cnf) that are referenced in the article since the paths are specific to Linux. For example, here is how the `[CA_default]` section would look for Windows:

```
  [ CA_default ]
  # Directory and file locations.
  dir               = C:\\<your directory>\\root\\ca
  certs             = $dir\\certs
  crl_dir           = $dir\\crl
  new_certs_dir     = $dir\\newcerts
  database          = $dir\\index.txt
  serial            = $dir\\serial
  RANDFILE          = $dir\\private\\.rand
```

Note the change to the ``dir`` setting to include the full path to the `root/ca` folder that is created along with the changes to the `$dir` settings to use a double backslashes `\\` instead of a single forward slash `/`.

## Signing the certificate

Using the intermediate pair created in the previous step, we can now create a server certificate using the following OpenSSL command

```
  openssl ca -config .\root\ca\intermediate\openssl.cnf -extensions server_cert -days 7300 -notext -md sha256 -in C:\path\to\csr\webapi.csr -out C:\path\to\csr\webapi.cer
```

Next we can create the .p7b file that will be used to store in the keystore:

```
  openssl crl2pkcs7 -nocrl -certfile C:\path\to\csr\webapi.cer -out C:\path\to\csr\webapi.p7b
```

## Installing into the keystore

Before we can install the certificate, we also need to install the root and intermediate certificates so that the proper trust chain is set in the keystore. This is done using the following commands:

Root:

```
  keytool -import -alias root -keystore keystore.jks -trustcacerts -file .\root\ca\certs\ca.cert.pem
```

Intermediate:

```
  keytool -import -alias intermed -keystore keystore.jks -trustcacerts -file .\root\ca\intermediate\certs\intermediate.cert.pem
```

Now we can import the certificate for the application:


```
  keytool -import -alias webapi -keystore keystore.jks -trustcacerts -file C:\path\to\csr\webapi.p7b
```