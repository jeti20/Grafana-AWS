# Grafana-AWS
A project showing how to create an example monitoring using EC2 instances from AWS and Grafana.

[Installing Grafana](#installing-grafana-on-ec2)
 <br> [Adding IP to Domain](#adding-ip-to-domain)
 <br> [Reverse Proxy with Nginx](#reverse-proxy-with-nginx)
 <br> [DataSource: MySQL](#datasource-mysql)
 
## Installing Grafana on EC2
Start with creating EC2 Instance with Ubuntu. On purpose of this project I used the free tier options.
Set up security group.

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/f3f5e9dc-e624-4ba5-bdd7-7c2b95ed84e3)

Port 22(SSH) and 3000 (Grafana) should be accessable.

Go to place where u store your private key for ssh and connect with this machine. On AWS website go into your instance, find and click "Connect" button, then switch to "SSH client"
and copy example which involve your EC2 instance public IP. Open console in place where you store privatekey for ssh (or use PuTTY) paste copied example with yourip address. If you fail to connect try with "sudo"

If you established connection with your machine type
```
apt-get update
```
go to website https://grafana.com/grafana/download?edition=oss and follow steops for Ubuntu and Debian.

Jeśli instalacja przebiegła pomyślnie włącz grafane by

Start the Grafana server using init.d
```
sudo service grafana-server start
sudo service grafana-server status
```

or 

Start the Grafana server with systemd
```
sudo systemctl daemon-reload
 sudo systemctl start grafana-server
 sudo systemctl status grafana-server
```

If Grafna is running corectly you should see 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/62a73e00-d174-4560-8271-03638a227131)

Open new tab in browser and type http://Public IPv4 address:3000
 <br> if everything is good you should see this website

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/9662f361-58ee-4580-8449-cc2192cca660)

default login and passowd is admin admin. You should change the password after the first login.

Next step is to add data source. Go into Connections --> Data sources from the left menu. Then press button "Add data source" 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/41f5c6be-ee20-4f88-9b89-03f8c38e187a)

You can search for "TestData" or scrol down to find it at the end of the list.

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/ed4bed7f-0e55-4a71-ab4c-84b4ab87b92b)


Set a name of the data source

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/5b3cb860-40a7-4168-b939-ad3ea36012d1)

And press "Save & test"

Go to "Dashboards" for mthe left menu and press "+ Add visualization". Choose data source which you created before 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/c005c673-4112-4834-90e9-777ce488b0ab)

New visualization created with random data

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/79049f84-5196-4cf2-88ed-a300483c7568)

## **Adding IP to Domain**
<br/> If u bought any domain you can connect your IP server to it. Just add new recod in DNS. Add "A" record and paste your IP. You can chek if it is already connected here https://dnschecker.org/ . You can now check you Grafana by writing domain.com:3000

## **Reverse Proxy with Nginx**
<br/>Now you have to write every time :3000 port in url, to get rid off it you have to configure webserver nginx. 
```
sudo apt install nginx
nginx -v
sudo service nginx status
cd /etc/nginx/sites-enabled
sudo nano YOUR-DOMAIN-NAME.conf
```

Paste it into this file 
```
server {
    listen 80;
    listen [::]:80;
    server_name  YOUR-DOMAIN-NAME;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass           http://localhost:3000/;
    }
}
```

Run this command to check if it was configured good
```
nginx -t
```
If everything was good you should see "nginx: configuration file /etc/nginx/nginx.conf test is successful"

Now you can type Yourdomain.com and see that grafana is working on this address without :3000

## **DataSource MySQL**

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/84d91627-98a4-43de-a6e4-e021a0abb193)


Create new instance of EC2 with open ports for 22 and 3306
<br/>![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/53566b83-9fde-4484-816e-0d6d4e966f5a)
```
sudo apt update
sudo apt install mysql-server
sudo service mysql status
```

Go to https://grafana.com/grafana/dashboards/7991-2mysql-simple-dashboard/ and downoland dashboard and github downoland this file my2_80.sql from https://github.com/meob/my2Collector . Use comand below to downoland this file.
```
wget https://raw.githubusercontent.com/meob/my2Collector/master/my2_80.sql
```
Open this file with nano and uncomment last 3 lines 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/0758d5d3-8f61-4ea7-b7f9-1ca2fe268ef1)

Run this script with this command
```
sudo mysql < my2_80.sql
```

Type 
```
sudo mysql
show databases;
show databases;
show variables where variable_name = 'event_scheduler';

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/c09f58bc-f5ca-4b90-a2c2-5494a39f99cb)

If it is OFF




```
select host, user from mysql.user;
use my2;
show tables;
select * from current;
select * from status;
quit
```

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/0a2fb53a-5fda-4c38-9cb0-c64ae1f5bd45)

