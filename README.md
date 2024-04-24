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
sudo apt-get update
```
go to website https://grafana.com/grafana/download?edition=oss and follow steops for Ubuntu and Debian.

```
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_10.4.1_amd64.deb
sudo dpkg -i grafana_10.4.1_amd64.deb
```

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
<br/>Now that we have a MySQL server, we will install a dedicated collector for it that will periodically gather statistics about the MySQL server and store them into a table containing rows with times and values
<br/>I have MySQL 8.#.#, so I will download the file called my2_80.sql. If you have MySQL 5, then download my2.sql
Go to https://grafana.com/grafana/dashboards/7991-2mysql-simple-dashboard/ and downoland dashboard and github downoland this file my2_80.sql from https://github.com/meob/my2Collector . Use comand below to downoland this file.
```
wget https://raw.githubusercontent.com/meob/my2Collector/master/my2_80.sql
```
Open this file with nano and uncomment last 3 lines 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/0758d5d3-8f61-4ea7-b7f9-1ca2fe268ef1)

Run this script with this command. This will run a script and create a user and database
```
sudo mysql < my2_80.sql
```

Type 

```
sudo mysql
show databases;
```
Baza danych performance_schema w MySQL jest specjalnym narzędziem, które dostarcza dostęp do danych związanych z wydajnością działania serwera MySQL. Zawiera różne tabele i widoki, które oferują informacje na temat wewnętrznych operacji serwera, wykorzystania zasobów oraz metryk wydajności. W skrypcie po stworzeniu użytkownika my2 nadaliśmy mu uprawnienia do SELCT danych z bazy performance_schema. Baza my2 przechowuje dane w sposób imitujacy dane time series

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/f44a4f48-d1e2-4fe6-b2a3-5bb98a4bd1a3)


Checking if event_scheduler in turned ON so he can push the data to Grafana

```
show variables where variable_name = 'event_scheduler';
```

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/c7f88bce-1ae4-48d6-b5d5-1952cf00c2c9)

If it is OFF, add this to the end of file /etc/mysql/my.cnf
```
[mysqld]
event_scheduler = on
```
reset MySQL
```
sudo service mysql restart
```

```
select host, user from mysql.user;
use my2;
show tables;
```
sprawdź co się znajduej w tabelach

```
select * from current;
select * from status;
quit
```
Wpis bind-address = 0.0.0.0 w pliku konfiguracyjnym MySQL (zazwyczaj my.cnf lub my.ini) określa adres IP, na którym serwer MySQL będzie nasłuchiwał połączeń przychodzących. Ustawienie tego na 0.0.0.0 oznacza, że serwer będzie nasłuchiwał na wszystkich interfejsach sieciowych dostępnych na maszynie, co umożliwia przyjmowanie połączeń z dowolnego adresu IP.
```
sudo nano /etc/mysql/my.cnf
```
add this at the end of the file 
```
[mysqld]
bind-address    = 0.0.0.0
```
Go to grafana and add source MySQL.

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/0a018bfb-34fc-4a57-a0a7-a76ef892063f)

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/5d13e9ea-5e2b-4a82-8905-425b95403d1d)

Google this isssue https://stackoverflow.com/questions/66974351/connecting-grafana-to-mysql-show-query-failed-why

got to MySQL shell

```
mysql
CREATE USER 'grafana'@'YourIP' IDENTIFIED BY 'password';
GRANT SELECT ON my2.* TO 'grafana'@'184.72.213.38';
FLUSH PRIVILEGES;

```
You can check if user was added by typi
Firstly we create user grafana with password passord. Then we give him permission to read the my2 database. FLUSH PRIVILEGES is used in MySQL after updating the permission table to ensure that the MySQL server is using the latest permission configuration. Go bac kto grafana and try to add again MySQL as a source. select host, user from mysql.user; -> by typing this in mySQL shell u can check if grafana user was added.

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/8e43f4c3-32c7-44a1-9e80-2b8573874ff6)

Go to https://grafana.com/grafana/dashboards/7991-2mysql-simple-dashboard/ and copy ID 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/95bdb067-4266-432f-9c2d-ad6b5c4454a1)

Then switch to Grafana -> dashboard tab -> new -> import 

![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/5dd38122-0e96-4889-b113-378d0f19e1ca)
![image](https://github.com/jeti20/Grafana-AWS/assets/61649661/abe96b22-5908-4221-abfc-9f10ebdfd1fe)

On my MySQL server, I can run these iptables rules to restrict incoming connections to port 3306 only for my Grafana server.
```
iptables -A INPUT -p tcp -s <your Grafana servers domain name or ip address> --dport 3306 -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -j DROP
iptables -L
```

iptables -A INPUT -p tcp -s <your Grafana servers domain name or ip address> --dport 3306 -j ACCEPT: To jest pierwsza reguła. Pozwala na przyjmowanie połączeń TCP na porcie 3306 z określonego adresu IP lub nazwy domeny, którą podałeś. Oznacza to, że tylko połączenia pochodzące z tego konkretnego źródła będą dozwolone na porcie 3306.

iptables -A INPUT -p tcp --dport 3306 -j DROP: To jest druga reguła. Odrzuca (DROP) wszystkie inne połączenia TCP na porcie 3306, które nie pasują do pierwszej reguły. Oznacza to, że wszystkie połączenia na porcie 3306, które nie są zgodne z pierwszą regułą (np. połączenia z innych adresów IP), zostaną odrzucone.

iptables -L: To polecenie wyświetla listę aktualnych reguł w firewall

Its important to rememver that user my2 and grafana is READ ONLY permissions. So the cannot interrupt the actual database my2
