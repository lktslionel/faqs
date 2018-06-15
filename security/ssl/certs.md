###### FAQ • SECURITY • SSL 
# Certificates Management

Everything related to SSL Certificate management 


## Contents

  * [Generate a free SSL Certificate - LetsEncrypt]


### Generate a free SSL Certificate - LetsEncrypt

#### Objective

> **As an** ops, **I want** to Generate a new certificate for our website at domains : 
> 
> * *.company-name.com (wildcard certificate)
> * shop.dev.company-name.io
> 
> **In order** to secure the traffic on the our website. 

#### Solution

Create a working directory

```bash
mkdir -p /tmp/letsencrypt/{etc,logs}
```

Then, run the following command :

```bash
sudo docker run -it --rm --name certbot \
  -v /tmp/extras/etc:/etc/letsencrypt \
  -v $(pwd)/extras/logs:/var/lib/letsencrypt \
  certbot/certbot certonly --manual \
  --agree-tos \
  --manual-public-ip-logging-ok \
  --server https://acme-v02.api.letsencrypt.org/directory \
  -d '*.company-name.com' \
  -d 'shop.dev.company-name.io' \
```

> In the process, you will be prompt to validate the ownership of thoses domains.
> Please, follow those instructions and everything should go fine.
>
> I recommend the DNS validation because it is easy than the webserver validation challenge.


Add any number of domain at the end of the command wih the option `-d <your-domain>`.

All your generated certificates will be present in the working directory at paths `/tmp/letsencrypt/etc/live/`.


[Generate a free SSL Certificate - LetsEncrypt]: #generate-a-free-ssl-certificate--letsencrypt

