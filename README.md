# v2ray_cloudflare
## 1、**vps购买**
## 2、**nameServer注册域名**
## 3、**cloudflare获取APIKEY**
## 4、**安装依赖文件**
```markdown
 apt-get update && apt-get install nginx curl ufw socat
 ```
## 5、**校准时间**

```markdown
date -R
sudo timedatectl set-local-rtc 1
sudo timedatectl set-timezone Asia/Shanghai
```

## 6、**开放端口**
```markdown
sudo ufw allow 80
sudo ufw allow 443
```
## 7、**安装v2ray**
```markdown
~~bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)~~
sudo curl  https://get.acme.sh | sh 
acme.sh  --register-account  -m EMAIL_ADDRESS --server zerossl

```
## 8、**生成证书**
```markdown
sudo curl  https://raw.githubusercontent.com/acmesh-official/acme.sh/master/acme.sh | sh
source ~/.bashrc
export CF_Key="API_KEY"
export CF_Email="EMAIL_ADDRESS"

没有生效则直接写入~/.acme.sh/account.conf当中，参考https://github.com/acmesh-official/acme.sh/wiki/dnsapi#1-use-cloudflare-domain-api-to-automatically-issue-cert

sudo ~/.acme.sh/acme.sh --issue --dns dns_cf -d domainname -d *.domainname -k ec-256
sudo ~/.acme.sh/acme.sh --installcert -d domainname -d *.domainname --fullchainpath /usr/local/etc/v2ray/domainname.crt --keypath /usr/local/etc/v2ray/domainname.key --ecc
```
## 9、**修改v2ray文件**
```markdown
nano /usr/local/etc/v2ray/config.json
```
```markdown
{
  "inbounds": [
    {
      "port": 12345,
      "listen": "127.0.0.1",
      "tag": "vmess-in",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "生成的userid",
            "alterId": 0
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/wsapp"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "vmess-in"
        ],
        "outboundTag": "direct"
      }
    ]
  }
}
```

## 10、**更新nginx**
```markdown
sudo mkdir /var/www/mysite
nano /etc/nginx/conf.d/default.conf
```
```markdown

server {
    listen 443 ssl;
    ssl on;
    ssl_certificate /usr/local/etc/v2ray/domainname.crt;
    ssl_certificate_key /usr/local/etc/v2ray/domainname.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    server_name domainname;
    index index.html index.htm;
    root /var/www/mysite;
    location /wsapp
    {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:12345;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
    }
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

## 11、**开始V2ray和nginx**
```markdown
sudo fuser -k 80/tcp
sudo fuser -k 443/tcp

systemctl enable v2ray
systemctl enable nginx
systemctl restart nginx
systemctl status nginx
systemctl restart v2ray
systemctl status v2ray
```

## 11、**配置客户端**

![avatar](picture\配置.png)
