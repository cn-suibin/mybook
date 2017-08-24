
*QQ stmp发送邮件配置mailx*
```
vi /etc/mail.rc

set from=224******53@qq.com
set smtp=smtp.qq.com
set smtp-auth-user=224******53@qq.com
set smtp-auth-password=euia********chb
set smtp-auth=login
set smtp-use-starttls
set ssl-verify=ignore
set nss-config-dir=/root/.certs
```
*QQ stmp发送邮件配置mailx-证书安装*
```
mkdir -p /root/.certs/
echo -n | openssl s_client -connect smtp.qq.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/qq.crt
certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt
certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/qq.crt
certutil -L -d /root/.certs
certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu"  -d ./ -i qq.crt 
```
*QQ stmp发送邮件配置mailx-发送方法*
```
echo "1111wwww11111111111" | mail -s "hello" 157595075@qq.com
```
