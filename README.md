# Jarkom-Modul-3-B16-2023
**Praktikum Jaringan Komputer Modul 3 Tahun 2023**

# Kelompok B16
| Nama | NRP |Github |
|---------------------------|------------|--------|
|Ahmad Fauzan A | 5025211091 | https://github.com/fazghfr |
|Syomeron Ansell W | 5025211540 | https://github.com/Ansell10 |

# Laporan Resmi
## Topologi
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/ef991cb0-bded-4bb7-bfb6-c2d7f4165b87)

Notes : 
IP Gateway ke router 192.186.x.2, untuk memenuhi permintaan soal nomor 0. Ada domain yang harus di tujukan ke node worker dengan ip 192.186.x.1. Untuk standarisasi, semua gateway dialihkan ke 192.186.x.2
  
## Config Soal 0-6
**Aura (DHCP Relay)**
```
apt-get update && apt-get install isc-dhcp-relay -y

# pada /etc/default/isc-dhcp-relay
SERVERS="192.186.1.1" # ip dari dns server
INTERFACES="eth1 eth2 eth3 eth4"
OPTIONS=

service isc-dhcp-relay restart
```
  
**Heiter (DNS Server)**
```
apt-get update
apt-get install bind9 -y

# daftar domain
mkdir /etc/bind/prak-3

echo 'zone "riegel.canyon.b16.com" {
        type master;
        file "/etc/bind/prak-3/riegel.canyon.b16.com";
};

zone "granz.channel.b16.com" {
        type master;
        file "/etc/bind/prak-3/granz.channel.b16.com";
};' > /etc/bind/named.conf.local

touch /etc/bind/prak-3/granz.channel.b16.com
touch /etc/bind/prak-3/riegel.canyon.b16.com

echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     riegel.canyon.b16.com. root.riegel.canyon.b16.com. (
			    2023111301    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@		IN      NS      riegel.canyon.b16.com.
@		IN      A       192.186.3.1 ; 
www             	IN      CNAME   riegel.canyon.b16.com.
frieren		IN      A       192.186.4.1 ; 
flamme	IN      A       192.186.4.5 ; 
fern		IN      A       192.186.4.6 ; ' > /etc/bind/prak-3/riegel.canyon.b16.com



echo ';
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     granz.channel.b16.com. root.granz.channel.b16.com. (
			    2023111301    ; Serial
                        604800        ; Refresh
                        86400         ; Retry
                        2419200       ; Expire
                        604800 )      ; Negative Cache TTL
;
@		IN      NS      granz.channel.b16.com.
@                    IN      A       192.186.3.1 ; 
www                IN      CNAME   granz.channel.b16.com.
lawine		IN      A       192.186.3.1 ; 
linie		IN      A       192.186.3.5 ; 
lugner		IN      A       192.186.3.6 ; ' > /etc/bind/prak-3/granz.channel.b16.com
```
  
**Himmel (DHCP Server)**
```
apt-get update
apt-get install isc-dhcp-server -y

# masukin eth0 sebagai INTERFACES di /etc/default/isc-dhcp-server


# config per subnet
echo '
subnet 192.186.3.0 netmask 255.255.255.0 {
	range 192.186.3.16 192.186.3.32;
	range 192.186.3.64 192.186.3.80;
	option routers 192.186.3.2;
	option broadcast-address 192.186.3.255;
	option domain-name-servers 192.186.1.3;
	default-lease-time 180;
	max-lease-time 5760;
}

subnet 192.186.4.0 netmask 255.255.255.0 {
	range 192.186.4.12 192.186.4.20;
	range 192.186.4.160 192.186.4.168;
	option routers 192.186.4.2;
	option broadcast-address 192.186.4.255;
	option domain-name-servers 192.186.1.3;
	default-lease-time 720;
	max-lease-time 5760;
}

subnet 192.186.1.0 netmask 255.255.255.0 {
}' >> /etc/dhcp/dhcpd.conf
```
  
**Client (DHCP)**
```
nano /etc/network/interfaces.

auto eth0
iface eth0 inet dhcp
```
  
**Eisen (Load balancer)**
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update
apt-get install nginx -y

 # Default menggunakan Round Robin granz
echo 'upstream php {
	server 192.186.3.1;
 	server 192.186.3.5; 
 	server 192.186.3.6; 
 }

 server {
 	listen 80;
 	server_name _;

 	location / {
 	proxy_pass http://php;
proxy_set_header    X-Real-IP $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    Host $http_host;
 	}
 }' > /etc/nginx/sites-available/lb-granz


 ln -s /etc/nginx/sites-available/lb-granz /etc/nginx/sites-enabled
unlink /etc/nginx/sites-enabled/default

service nginx restart
```
  
PHP Worker domain Granz (Lawine Linie Lugner)
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
apt-get update && apt-get install unzip nginx php php-fpm -y

cd /var/www/ && curl -LOJ "https://drive.google.com/u/0/uc?id=1ViSkRq7SmwZgdK64eRbr5Fm1EGCTPrU1&export=download" && unzip *.zip && mv modul* granz && rm *.zip
 
 echo 'server {

 	listen 80;

 	root /var/www/granz;

 	index index.php index.html index.htm;
 	server_name _;

 	location / {
 			try_files $uri $uri/ /index.php?$query_string;
 	}

 	# pass PHP scripts to FastCGI server
 	location ~ \.php$ {
 	include snippets/fastcgi-php.conf;
 	fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
 	}

    location ~ /\.ht {
 			deny all;
 	}

 	error_log /var/log/nginx/granz_error.log;
 	access_log /var/log/nginx/granz_access.log;
 }' > /etc/nginx/sites-available/granz

 ln -s /etc/nginx/sites-available/granz /etc/nginx/sites-enabled
unlink /etc/nginx/sites-enabled/default

service nginx restart
service php7.3-fpm start
```
  
**[Extras] Helper untuk Switching Algo**
Untuk memudahkan pengerjaan nomor 7 dan 8 Di node load balancer
```
mkdir /root/algo-lb-granz
mkdir /root/algo-lb-granz/change-script

echo 'cp ~/algo-lb-granz/gen* /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-genhash.sh

echo 'cp ~/algo-lb-g*/ip* /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-iphash.sh

echo 'cp ~/algo-lb-g*/least* /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-leastcon.sh

echo 'cp ~/algo-lb-g*/*1worker /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-rr1worker.sh

echo 'cp ~/algo-lb-g*/*2worker /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-rr2worker.sh

echo 'cp ~/algo-lb-g*/*3worker /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-rr3worker.sh

echo 'cp ~/algo-lb-g*/weigh* /etc/nginx/sites-e*/lb-granz
service nginx restart' > /root/algo-lb-granz/change-script/to-weightedrr.sh

echo 'upstream php {
        hash $request_uri consistent;
        server 192.186.3.1;
        server 192.186.3.5;
        server 192.186.3.6;
 }

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 }' > /root/algo-lb-granz/genhash

echo 'upstream php {
        ip_hash;
        server 192.186.3.1;
        server 192.186.3.5;
        server 192.186.3.6;
 }

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 }' > /root/algo-lb-granz/iphash

echo 'upstream php {
        least_conn;
        server 192.186.3.1;
        server 192.186.3.5;
        server 192.186.3.6;
 }

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 }' > /root/algo-lb-granz/leastconn

echo 'upstream php {
        server 192.186.3.1;
}

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 }' > /root/algo-lb-granz/1worker

echo 'upstream php {
        server 192.186.3.1;
        server 192.186.3.5;
 }

        server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 }' > /root/algo-lb-granz/2worker

echo 'upstream php {
        server 192.186.3.1;
        server 192.186.3.5;
        server 192.186.3.6;
 }

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 } ' > /root/algo-lb-granz/3worker

echo 'upstream php {
        server 192.186.3.1 weight=3; # tambahkan weight
        server 192.186.3.5 weight=2;
        server 192.186.3.6 weight=1;
 }

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://php;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    Host $http_host;

        }
 }' > /root/algo-lb-granz/weightedrr
```


