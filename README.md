# Grafana-AWS
A project showing how to create an example monitoring using EC2 instances from AWS and Grafana.

Start with creating EC2 Instance with Ubuntu. On purpose of this project I used the free tier options.
Set up security group.

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/f3f5e9dc-e624-4ba5-bdd7-7c2b95ed84e3)

 Port 22(SSH) and 3000 (Grafana) should be accessable.

Go to place where u store your private key for ssh and connect with this machine. On AWS website go into your instance, find and click "Connect" button, then switch to "SSH client"
and copy example which involve your EC2 instance public IP. Open console in place where you store privatekey for ssh (or use PuTTY) paste copied example with yourip address. If you fail to connect try with "sudo"

If you established connection with your machine type

**apt-get update**

go to website https://grafana.com/grafana/download?edition=oss and follow steops for Ubuntu and Debian.

Jeśli instalacja przebiegła pomyślnie włącz grafane by

Start the Grafana server using init.d

**sudo service grafana-server start**
 <br> **sudo service grafana-server status**

or 

Start the Grafana server with systemd

**sudo systemctl daemon-reload
 <br>sudo systemctl start grafana-server
 <br>sudo systemctl status grafana-server**

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

Adding IP to Domain
<br/> If u bought any domain you can connect your IP server to it. Just add new recod in DNS. Add "A" record and paste your IP. You can chek if it is already connected here https://dnschecker.org/ . You can now check you Grafana by writing domain.com:3000

Reverse Proxy with Nginx
Now you have to write every time :3000 port in url, to get rid off it you have to configure webserver nginx. 

<br/>sudo apt install nginx
<br/>nginx -v
<br/>sudo service nginx status
<br/>cd /etc/nginx/sites-enabled
<br/>sudo nano YOUR-DOMAIN-NAME.conf

Paste it into this file 

server {
    listen 80;
    listen [::]:80;
    server_name  YOUR-DOMAIN-NAME;

    location / {
        proxy_set_header Host $http_host;
        proxy_pass           http://localhost:3000/;
    }
}

Run this command to check if it was configured good
<br/>nginx -t
<br/>If everything was good you should see "nginx: configuration file /etc/nginx/nginx.conf test is successful"

Now you can type Yourdomain.com and see that grafana is working on this address without :3000
