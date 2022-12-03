# X.509 Certificate

Confidential Cloud Studio is architected using strong security mechanisms and requires an x.509 certificate to secure the entire collaboration process.

## Create a certificate

Fill in the following vars:

```
MY_COUNTRY = ""
MY_REGION = ""
MY_LOCATION = ""
MY_COMPANY = ""
MY_COMPANY_WEBSITE = ""
```

and run the below command:

```
> openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -subj "/C=$MY_COUNTRY/ST=$MY_REGION/L=$MY_LOCATION/O=$MY_COMPANY/OU=Org/CN=$MY_COMPANY_WEBSITE"
```

Once done, upload the certificate in Confindetial Cloud either during sign-up or under Profile.
