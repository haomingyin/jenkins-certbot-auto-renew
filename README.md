# jenkins-certbot

This repository contains a Jenkinsfile which instructs Jenkins to periodically run [scripts](https://github.com/haomingyin/certbot-namecheap-hook) to renew SSL certs.

## Job Description

### Domain

haomingyin.com

### IP

The IP of the service which runs this Jenkinsfile

### Frequency

Once a month at a random time which is managed by Jenkins

### Cert To Be Renewed

- `*.haomingyin.com`
