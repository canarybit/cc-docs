# X.509 Certificate

Confidential Cloud Studio requires unique cryptographic identities for access control and implements them using cryptographic certificates. Therefore, every user must complete their account with an X.509 certificate to use the platform.

## Create a certificate

The following is an example procedure to create an _unsigned_ X.509 certificate using OpenSSL. Note that this example is exclusively meant for test and education purposes. We recommend you to follow your organisation's prescribed procedure to obtain an X.509 certificate. We also recommend you to use a modern digital signature algorithm, such as ED25519.

To create an unsigned X.509 certificate, in your CLI, create the following vars:

```
MY_COUNTRY=""
MY_REGION=""
MY_LOCATION=""
MY_COMPANY=""
MY_COMPANY_WEBSITE=""
MY_EMAIL=""
```

and run the below command:

```
> openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -subj "/C=$MY_COUNTRY/ST=$MY_REGION/L=$MY_LOCATION/O=$MY_COMPANY/OU=Org/CN=$MY_COMPANY_WEBSITE/emailAddress=$MY_EMAIL"
```

Upload the certificate in Confidential Cloud either during Sign-up or under Profile.
