## How to automate certbot with aws wild card certificates


### Import entrys for route 53
```
$ORIGIN dudewhereismy.mx
$TTL 3600   
old IN A 13.58.151.163
deepspeech IN A 3.22.95.69
```
### Create Policy in AWS

* Go to AWS console log with root user 
* Click in the right top and click My security credentials
* Expand access keys
* Click create New access Key and download (Open the file and get the access key and access secret
* Create a file in the location: /home/$USER/.aws/config and /root/.aws/config using the following formart
```
[default]
aws_access_key_id=XXXXXXXXXXXXXXXX
aws_secret_access_key=XXXXXXXXXXX
```
* In the left pannel click and create a policy with the following:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:GetChange",
                "route53:ListHostedZones",
                "route53:ChangeResourceRecordSets"
            ],
            "Resource": "*"
        }
    ]
}
```


### Stop services in the server
```
sudo service apache2 stop
sudo service haproxy stop
sudo service rabbitmq-server stop
```

### Install certbot route
```
sudo apt-get update -y
sudo apt-get install -y python3-certbot-dns-route53
```
### Optional install

```
https://websiteforstudents.com/how-to-install-python-on-ubuntu-linux/

wget https://files.pythonhosted.org/packages/da/f6/c83229dcc3635cdeb51874184241a9508ada15d8baa337a41093fab58011/pip-21.3.1.tar.gz
tar -xzvf pip-21.3.1.tar.gz 
cd pip-21.3.1
sudo python3 setup.py install

$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository ppa:certbot/certbot
$ sudo apt update
$ sudo apt install certbot

$ sudo apt install python3-pip
$ sudo pip3 install certbot-dns-route53

cat /etc/letsencrypt/live/dwim.mx/fullchain.pem /etc/letsencrypt/live/dwim.mx/privkey.pem > /etc/haproxy/certs/dwim.mx.pem
```

### Create certificate

```
certbot certonly -d *.dudewhereismy.mx --dns-route53 --logs-dir ~/letsencrypt/log/ --config-dir ~/letsencrypt/config/ --work-dir /home/wsgi/letsencrypt/work/ -m jorge@dwim.mx --agree-tos --non-interactive --server https://acme-v02.api.letsencrypt.org/directory
```
### Create PEM
```
sudo DOMAIN='dudewhereismy.mx' sudo -E bash -c 'cat /home/wsgi/letsencrypt/config/live/$DOMAIN/fullchain.pem /home/wsgi/letsencrypt/config/live/$DOMAIN/privkey.pem > /etc/haproxy/certs/dudewhereismy.mx.pem'
```
### Update Rabbit 
```
cp /home/wsgi/Python-3.5.2/lib/python3.5/site-packages/pip/_vendor/certifi/cacert.pem  /home/wsgi/certs/.
cd /home/wsgi/letsencrypt/config/live/dudewhereismy.mx
cp cert.pem /home/wsgi/certs/.
cp privkey.pem /home/wsgi/certs/.
sudo chmod -R go-rwx /etc/haproxy/certs
```
### Restart Services
```
sudo service rabbitmq-server start
sudo service apache2 start
sudo service haproxy start
```
### Create crontab for auto renew

```
sudo -i
crontab -e
```
### Add at the bottom Check every month:
```
0 0 1 * 0 /home/wsgi/backups/renew-certs.sh
```

### Create file renew-certs.sh at /home/wsgi/backups
```
/usr/bin/certbot renew --dns-route53 --dns-route53-propagation-seconds 29 && bash -c "cat /etc/letsencrypt/live/dudewhereismy.com.mx/fullchain.pem /etc/letsencrypt/live/dudewhereismy.com.mx/privkey.pem > /etc/haproxy/certs/dudewhereismy.com.mx.pem " && bash -c "cat /etc/letsencrypt/live/elisasoftware.com.mx/fullchain.pem /etc/letsencrypt/live/elisasoftware.com.mx/privkey.pem > /etc/haproxy/certs/elisasoftware.com.mx.pem" && service haproxy reload
```
or 
```
/usr/bin/certbot renew --dns-route53 --dns-route53-propagation-seconds 29 && bash -c "cat /etc/letsencrypt/live/dudewhereismy.mx/fullchain.pem /etc/letsencrypt/live/dudewhereismy.mx/privkey.pem > /etc/haproxy/certs/dudewhereismy.mx.pem " && bash -c "cat /etc/letsencrypt/live/elisasoftware.mx/fullchain.pem /etc/letsencrypt/live/elisasoftware.mx/privkey.pem > /etc/haproxy/certs/elisasoftware.mx.pem" && service haproxy reload
```
or 
```
/usr/bin/certbot renew --dns-route53 --dns-route53-propagation-seconds 29 && bash -c "cat /home/wsgi/letsencrypt/config/live/dudewhereismy.mx/fullchain.pem /home/wsgi/letsencrypt/config/live/dudewhereismy.mx/privkey.pem > /etc/haproxy/certs/dudewhereismy.mx.pem" && chmod 644 /etc/haproxy/certs/dudewhereismy.mx.pem && service haproxy reload
```
### Grant permit
```
chmod +x renew-certs.sh 
```
