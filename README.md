## SMTP Postfix server in docker-compose

### start
* `cd config/certs/ && openssl req -new -nodes -keyout smtp.key -out smtp.csr`
* `cd ../../ && `opendkim-genkey -D /etc/opendkim/domainkeys -d example.com -s mail`
  * opendkim-genkey -D ./config/opendkim/ -d $(hostname -d) -s $(hostname)` 
* `docker-compose up -d`
* if don't need DKIM:
  * `docker-compose exec postfix bash`
  * comment following strings in /etc/postfix/main.cf
   ```
   milter_protocol = 2
   milter_default_action = accept
   smtpd_milters = inet:localhost:12301
   non_smtpd_milters = inet:localhost:12301
   ```
  * `docker-compose restart postfix`

### DNS config
* add A record:
  * mail A <ip>
* add MX record:
  * @ MX 10 mail
* add CNAME records:
  * relay CNAME mail
  * smtp CMANE mail
* SPF config:
  * (SPF config in panel `example.com. IN TXT "v=spf1 +a +mx ?all"`)
  * add TXT SPF record
    *  `example.com. IN TXT "v=spf1 +a +mx ?all"`
* DKIM config:
  * (activate in panel)
  * add TXT record:
    * mail._domainkey TXT <from file ./config/opendkim/mail.txt>
  * if needed - add ADSP (for deny other servers receive mails without sign, with my domain): #OLD
    * `_adsp._domainkey IN TXT "dkim=all"`
* check:
   * `dig A mail.example.com`
   * `dig MX mail.example.com`
   * `dig TXT mail.example.com`
   * `dig CNAME mail.example.com`
* verify domain in Google:
   * https://support.google.com/a/answer/183895
* spam check:
   * https://spamcheck.postmarkapp.com/
   * https://spamassassin.apache.org/
  
### check mail
* genegate auth for telnet:
  * `perl -MMIME::Base64 -e 'print encode_base64("user\@example.com");'
    * dXNlckBzZXJ2ZXIucnU=
  * `perl -MMIME::Base64 -e 'print encode_base64("PASSWORD");'
    * UEFTU1dPUkQ=
    
* `telnet postfix 25`
  ```
  Trying 172.18.0.9...
  Connected to postfix.
  Escape character is '^]'.
  220 example.com ESMTP Postfix (Ubuntu)
  ```
  * `helo test`
  ```
  250 example.com
  ```
  * `auth login`
  ```
  334 VXNlcm5hbWU6
  ```
  * `dXNlckBzZXJ2ZXIucnU=`
  ```
  334 UGFzc3dvcmQ6
  ```
  * `UEFTU1dPUkQ=`
  ```
  235 2.7.0 Authentication successful
  ```
  * `mail from: test@example.com`
  ```
  250 2.1.0 Ok
  ```
  * `rcpt to: user@mail.com`
  ```
  250 2.1.5 Ok
  ```
  * `data`
  ```
  354 End data with <CR><LF>.<CR><LF>
  ```
  * `From: test@example.com`
  * `To: user@mail.com`
  * `Subject: testmail`
  * `some text`
  * .
  ```
  250 2.0.0 Ok: queued as B209CA20E1E
  ```
  * `quit`

