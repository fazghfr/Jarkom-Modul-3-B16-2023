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
### Aura (DHCP Relay)
```
apt-get update && apt-get install isc-dhcp-relay -y

# pada /etc/default/isc-dhcp-relay
SERVERS="192.186.1.1" # ip dari dns server
INTERFACES="eth1 eth2 eth3 eth4"
OPTIONS=

service isc-dhcp-relay restart
```
  
### Heiter (DNS Server)
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
  
### Himmel (DHCP Server)
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
  
### Client (DHCP)
```
nano /etc/network/interfaces.

auto eth0
iface eth0 inet dhcp
```
  
### Eisen (Load balancer)
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
  
### PHP Worker domain Granz (Lawine Linie Lugner)
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
  
### [Extras] Helper untuk Switching Algo
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

## Soal 7
Kepala suku dari Bredt Region memberikan resource server sebagai berikut:
- Lawine, 4GB, 2vCPU, dan 80 GB SSD.
- Linie, 2GB, 2vCPU, dan 50 GB SSD.
- Lugner 1GB, 1vCPU, dan 25 GB SSD.
  
aturlah agar Eisen dapat bekerja dengan maksimal, lalu lakukan testing dengan 1000 request dan 100 request/second.

```
upstream php {
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
 }
```
  
### Weighted Round Robin

Menggunakan weighted round robin dikarenakan tiap server memiliki resource yang berbeda. Diurutkan dari yang terbesar ke terkecil, Lawine, Linie, Lugner.
Pada node eisen, Edit file /etc/nginx/sites-e*/lb-granz

```
upstream php {
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
 }

service nginx restart
```
  
**Lakukan testing menggunakan apache benchmark pada client**
```
apt-get update && apt-get install apache2-utils -y
```
  
**lakukan testing ke load balancer**
```
ab -n 1000 -c 100 http://192.186.2.3/
```
Note : ip pada link tersebut adalah ip pada loadbalancer.

**Testing Result**
  
![unnamed](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/f2edbf23-0520-44f9-9060-0baa09902e64)

  



## Soal 8
Karena diminta untuk menuliskan grimoire, buatlah analisis hasil testing dengan 200 request dan 10 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut:
- Nama Algoritma Load Balancer
- Report hasil testing pada Apache Benchmark
- Grafik request per second untuk masing masing algoritma.

Pada client lakukan 
```
ab -n 100 -c 10 -g out.data http://192.186.2.3/

mkdir /root/benchmark
cd /root/benchmark
```

  
### Weighted Round Robin

Edit file /etc/nginx/sites-e*/lb-granz
```
upstream php {
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
 }
```

  
**Testing Result**
  
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/1195be24-41cf-44c6-9ff4-3512c230a737)

  
### Least Connection
```
upstream php {
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
 }
```

**Testing Result**
  
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/c65244af-0c39-43e8-b29d-ba9d3b9a1627)
  
  
### IP Hash
```
upstream php {
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
 }
```

**Testing Result**
  
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/147bae69-32b9-4bb6-ab82-bc6d4a95955b)


### Generic Hash

```
upstream php {
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
 }
```

**Testing Result**
  
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/5c2a405c-fd14-4edd-a4ba-31e222be6e1c)

### Grafik Request per Second tiap Algo

![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/488c595a-c68d-4895-a9db-b30cabc240dc)


### Analisis
Grafik ini menunjukkan perbandingan jumlah permintaan per detik yang diatur atau dihandle oleh masing-masing metode tersebut dalam suatu sistem atau jaringan.
  
- Weighted Round Robin (WRR): Menangani sekitar 554,77 permintaan per detik. Ini mengindikasikan tingkat kinerja atau kapasitas dari metode WRR dalam menangani permintaan secara sekuensial berdasarkan bobot yang ditentukan.
  
- Least Connection: Menangani sekitar 798,66 permintaan per detik. Angka ini mencerminkan kemampuan metode ini dalam menangani permintaan dengan menugaskan permintaan baru ke server dengan jumlah koneksi terendah.

- IP Hash: Menangani sekitar 823,99 permintaan per detik. Metode IP Hash menggunakan hash dari alamat IP untuk menentukan server yang akan menangani permintaan. Angka ini menunjukkan tingkat kemampuan metode ini dalam menangani permintaan dengan pendekatan tersebut.

- Generic Hash: Menangani sekitar 890,75 permintaan per detik. Metode Generic Hash juga menggunakan teknik hashing, meskipun tidak secara spesifik terkait dengan alamat IP. Angka ini mencerminkan kemampuan metode ini dalam menangani jumlah permintaan tertentu dalam waktu satu detik.

Dari data ini, tampaknya Generic Hash memiliki kinerja terbaik dalam menangani jumlah permintaan per detik dalam konteks yang disajikan. Sedangkan Weighted Round Robin memiliki kinerja terendah dalam hal jumlah permintaan per detik. Tetapi, untuk membuat keputusan tentang metode mana yang terbaik, perlu dipertimbangkan juga faktor-faktor lain seperti keandalan, skalabilitas, dan kecocokan dengan kebutuhan sistem yang spesifik.


## Soal 9
Dengan menggunakan algoritma Round Robin, lakukan testing dengan menggunakan 3 worker, 2 worker, dan 1 worker sebanyak 100 request dengan 10 request/second, kemudian tambahkan grafiknya pada grimoire.


### 1 Worker
Edit file /etc/nginx/sites-e*/lb-granz
```
upstream php {
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
 }
```
  
**Testing Result**
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/a120890d-2f69-486c-8d45-367a609006d4)


### 2 Worker
Edit file /etc/nginx/sites-e*/lb-granz
```
upstream php {
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
 }
```

**Result Testing**
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/10af69f1-e63b-4ba4-950f-8265b04da1f7)


### 3 Worker
Edit file /etc/nginx/sites-e*/lb-granz
```
upstream php {
        server 192.186.3.1;
        server 192.186.3.5;
        server 192.186.3.6l 
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
 }
```

**Testing Result**
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/5b1b820e-dec7-45e5-9a4e-50b54ae7a3c3)


### Grafik
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/3e29a213-081e-4fcf-9f9e-277a17b8c6f9)

  
## Soal 10-11 (Autentikasi, Proxy Pass)
10. Selanjutnya coba tambahkan konfigurasi autentikasi di LB dengan dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/

11. Lalu buat untuk setiap request yang mengandung /its akan di proxy passing menuju halaman https://www.its.ac.id.

Di node loadbalancer
```
apt-get update && apt-get install apache2-utils

htpasswd -c /etc/nginx/rahasiakita/.htpasswd netics
```
akan ada entry password, masukin ajkb16
edit file /etc/nginx/sites-e*/lb-granz
```
echo ‘upstream php {
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
            
 	 auth_basic "netics area";
            auth_basic_user_file /etc/nginx/.htpasswd;
        }

        location ~ /\.ht {
            deny all;
        }
        
        # proxy pass
        location /its {
        proxy_pass https://www.its.ac.id;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        auth_basic "netics area";
        auth_basic_user_file /etc/nginx/rahasisakita/.htpasswd;
       }
 }
```


## Soal 13-14 (Deploy Laravel)
### Denken (Database Server)
```
echo namerver 192.168.122.1 >> /etc/resolv.conf
apt-get install mariadb-server -y

service mysql start
```

masuk ke mysql
```
CREATE USER 'kelompokb16'@'%' IDENTIFIED BY 'passwordb16';
CREATE USER 'kelompokb16'@'localhost' IDENTIFIED BY 'passwordb16';
CREATE DATABASE dbkelompokb16;
GRANT ALL PRIVILEGES ON *.* TO 'kelompokb16'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'kelompokb16'@'localhost';
FLUSH PRIVILEGES;

echo '[mysqld]
skip-networking=0
Skip-bind-address' >> /etc/mysql/my.cnf
```

### Worker Laravel (Database Client)
```
echo nameserver 192.168.122.1 >> /etc/resolv.conf
apt-get update && apt-get install mariadb-client -y

mariadb --host=192.186.2.1 --port=3306 --user=kelompokb16 --password
```

### Worker Laravel
```
apt-get update
apt-get install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2 -y

curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'

apt-get update && apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y

apt-get update && apt-get install nginx -y
```

instalasi composer
```
wget https://getcomposer.org/download/2.0.13/composer.phar
chmod +x composer.phar
mv composer.phar /usr/bin/composer

apt-get install git -y

cd /var/www/ && git clone https://github.com/martuafernando/laravel-praktikum-jarkom.git
cd /var/www/laravel* && composer update && composer install

cp .env* .env
```
edit .env di bagian dbnya, ipnya dll  
migrate db cd ke /var/www/laravel DULU
```
php artisan migrate:fresh
php artisan db:seed --class=AiringsTableSeeder
php artisan key:generate
```
deploy

## Soal 15 (Post Request Register)
isi regbody.json
```
{
  "username": "kelompokb16",
  "password": "passwordb16"
}

ab -n 100 -c 10 -p regbody.json -T application/json http://192.186.4.1:8001/api/auth/register
```

![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/56d672fd-533f-4155-8289-fccfa143c45b)

Terlihat bahwa dari 100 request, terdapat 99 request yang failed. Ini dikarenakan karena kita mengirim data body yang sama ke 100 request tersebut. maka melanggar sebuah constraint yaitu harus unique usernamenya. Oleh karenanya, hanya 1 yang sukses.

![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/4f0867ec-ac2f-4ce1-8c1a-bbd15a0d1663)

## Soal 16 (Post Auth Login)
isi loginbody.json
```
{
  "username": "kelompokb16",
  "password": "passwordb16"
}
ab -n 100 -c 10 -p regbody.json -T application/json http://192.186.4.1:8001/api/auth/register
```

![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/302fc037-b489-44c2-8c03-5d9166682b98)

Terdapat 39 yang failed. Ini dikarenakan kami melakukan testing ke satu worker saja. Berdasarkan data di atas, dapat disimpulkan satu worker ini tidak dapat menghandle semua 100 request, hanya 61 request saja.


## Soal 17 (GET /me)
```
curl -X POST -H "Content-Type: application/json" -d @logbody.json
http://192.186.4.1:8001/api/auth/login > login_output.txt
```
Sehingga didapatkan token suatu user pada login_output.txt
```
token=$(cat login_output.txt | jq -r '.token')
ab -n 100 -c 10 -H "Authorization: Bearer $token" http://192.186.4.1:8001/api/me
```
  
![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/114125933/b4c53b74-34d2-459a-9551-13bb60672208)

Terlihat dari data di atas, terdapat 40 request yang failed. Ini dikarenakan testing atau bencmarking dilakukan pada satu worker, dimana worker ini tidak dapat menghandle semua request yang ada.

## Soal 18 (Load Balancer Domain Laravel)

buat /etc/nginx/sites-a*/lb-riegel
```
echo 'upstream laravel {
        server 192.186.4.1:8001;
        server 192.186.4.5:8002;
        server 192.186.4.6:8003;
 }

 server {
        listen 80;
        server_name _;

        location / {
        proxy_pass http://laravel;
        }
 }' > /etc/nginx/sites-a*/lb-riegel

ln -s /etc/nginx/sites-available/lb-riegel /etc/nginx/sites-enabled/

service nginx restart
```

![image](https://github.com/fazghfr/Jarkom-Modul-3-B16-2023/assets/96367502/c5a075a9-db9d-4eab-994c-f064ba6be2a1)


Terlihat bahwa semua request berhasil, jika dibandingkan dengan benchmark yang menggunakan 1 worker, ini menunjukkan bahwa request di handle menggunakan semua worker yang ada.






