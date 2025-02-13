# Lucrare de laborator Nr.1: Server virtual
# Covrig Andrei, grupa I2301
# 2025

## Scopul lucrării
Familiarizarea cu virtualizarea sistemelor de operare și configurarea unui server HTTP virtual.

## Etapele realizării lucrării
### Descrierea creării imaginii discului
O imagine de disc pentru mașina virtuală este un fișier care emulează un hard disk fizic și este utilizat de software-ul de virtualizare (precum QEMU) pentru a stoca un sistem de operare, aplicații și date.
Pentru a crea o imagine de disc virtuală, folosim comanda:

```bash 
qemu-img create -f qcow2 debian.qcow2 8G
```
unde:
- `qemu-img` - utilitar pentru gestionarea imaginilor de disc pentru QEMU.
- `create` - creăm o nouă imagine de disc.
- `-f qcow2` - specificăm formatul imaginii (qcow2 este un format optimizat pentru QEMU, care suportă compresie și snapshots).
- `debian.qcow2` - specificăm numele fișierului imaginii de disc.
- `8G` - specificăm dimensiunea imaginii (8 GB).

### Descrierea instalării SO
După crearea imaginii de disc, putem porni instalarea SO Debian folosind comanda QEMU:

```bash 
qemu-system-x86_64 -hda debian.qcow2 -cdrom dvd/debian.iso -boot d -m 2G
```
unde:
- `qemu-system-x86_64` - pornim o mașină virtuală pe arhitectura x86_64.
- `-hda debian.qcow2` - specificăm discul virtual principal unde va fi instalat Debian.
- `-cdrom dvd/debian.iso` - montăm imaginea ISO a Debian aflată în directorul dvd/.
- `-boot d` - setăm ordinea de boot pentru a porni de pe unitatea CD-ROM.
- `-m 2G` - alocăm 2 GB de memorie RAM mașinii virtuale.

După instalare, putem porni sistemul folosind comanda:

```bash
qemu-system-x86_64 -hda debian.qcow2 -m 2G -smp 2 -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::1080-:80,hostfwd=tcp::1022-:22
```
unde:
- `qemu-system-x86_64` - rulăm o mașină virtuală QEMU pentru arhitectura x86_64.
- `-hda debian.qcow2` - folosim debian.qcow2 ca disc virtual.
- `-m 2G` - alocăm 2 GB RAM mașinii virtuale.
- `-smp 2` - alocăm 2 nuclee CPU pentru mașina virtuală.
- `-device e1000,netdev=net0` - utilizăm un adaptor de rețea Intel e1000.
- `user,id=net0` - creăm un dispozitiv de rețea cu ID-ul net0.
- `hostfwd=tcp::1080-:80` - redirecționăm traficul TCP de pe portul 1080 al gazdei către portul 80 al mașinii virtuale.
- `hostfwd=tcp::1022-:22` - redirecționăm traficul TCP de pe portul 1022 al gazdei către portul 22 al mașinii virtuale (util pentru acces SSH).
### Descrierea instalării LAMP
LAMP este un stack software utilizat pentru dezvoltarea și rularea aplicațiilor web. Acronimul provine de la tehnologiile pe care le include:
- __L__ - __Linux__: Sistemul de operare.
- __A__ - __Apache__: Serverul web care gestionează cererile HTTP și livrează pagini web.
- __M__ - __MySQL/MariaDB__: Sistem de gestionare a bazelor de date relaționale.
- __P__ - __PHP__: Limbaj de programare server-side pentru generarea de pagini web dinamice.

Pentru a instala LAMP în mașina virtuală executăm comenzile:
``` bash
sudo apt update -y
```
unde:
- `sudo` - rulăm comanda cu privilegii de administrator (root).
- `apt update` - actualizăm lista de pachete disponibile din depozitele oficiale Debian/Ubuntu.

``` bash
sudo apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client unzip
```
- `apache2` - instalăm serverul web Apache.
- `php` - instalăm PHP, necesar pentru a rula aplicații web dinamice.
- `libapache2-mod-php` - modul pentru Apache care permite interpretarea PHP.
- `php-mysql` - extensia PHP pentru a interacționa cu MySQL/MariaDB.
- `mariadb-server` - instalăm MariaDB, un înlocuitor open-source pentru MySQL.
- `mariadb-client` - clientul MariaDB pentru gestionarea bazelor de date din terminal.
- `unzip` - utilitar pentru extragerea fișierelor .zip.

### Descrierea instalării configurației host-urilor virtuale



### Descrierea instalării Drupal
Drupal este un sistem de management al conținutului (CMS) open-source utilizat pentru crearea și administrarea site-urilor web. Instalarea Drupal presupune mai mulți pași esențiali:
1.Descărcarea ultimei versiuni de pe site-ul oficial:
wget https://ftp.drupal.org/files/projects/drupal-11.1.1.zip
2.Dezarhivarea și mutarea în  ==> /var/www/drupal.
mkdir /var/www
unzip drupal-11.1.1.zip
mv drupal-11.1.1 /var/www/drupal
3.Crearea bazei de date și a utilizatorului pentru Drupal
mysql -u root
CREATE DATABASE drupal_db;
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON drupal_db.* TO 'user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
4.În directorul /etc/apache2/sites-available creați fișierul 02-drupal.conf
nano /etc/apache2/sites-available/02-drupal.conf
cu următorul conținut:
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/drupal"
    ServerName drupal.localhost
    ServerAlias www.drupal.localhost
    ErrorLog "/var/log/apache2/drupal.localhost-error.log"
    CustomLog "/var/log/apache2/drupal.localhost-access.log" common
</VirtualHost>
5.Pentru a activa configurațiile, executați comanda:
/usr/sbin/a2ensite 02-drupal

##Descrierea instalării PhpMyAdmin
phpMyAdmin este o interfață web pentru gestionarea bazelor de date MySQL/MariaDB. Instalarea phpMyAdmin presupune mai mulți pași esențiali:
1.Descărcarea ultimei versiuni de pe site-ul oficial:
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.2/phpMyAdmin-5.2.2-all-languages.zip
2.Dezarhivarea și mutarea în  ==> /var/www/phpmyadmin.
mkdir /var/www
unzip phpMyAdmin-5.2.2-all-languages.zip
mv phpMyAdmin-5.2.2-all-languages /var/www/phpmyadmin
3.În directorul /etc/apache2/sites-available creați fișierul 01-phpmyadmin.conf
nano /etc/apache2/sites-available/01-phpmyadmin.conf
cu următorul conținut:
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/phpmyadmin"
    ServerName phpmyadmin.localhost
    ServerAlias www.phpmyadmin.localhost
    ErrorLog "/var/log/apache2/phpmyadmin.localhost-error.log"
    CustomLog "/var/log/apache2/phpmyadmin.localhost-access.log" common
</VirtualHost>
4.Pentru a activa configurațiile, executați comanda:
/usr/sbin/a2ensite 01-phpmyadmin


## Răspunsurile la întrebările propuse
### Cum se poate descărca un fișier în consolă cu ajutorul utilitarului wget?
`wget` este un utilitar de linie de comandă folosit pentru a descărca fișiere de pe internet. 
Pentru a descărca un fișier folosind utilitarul `wget`, în consolă folosim comanda:

``` bash 
wget [URL]
```
unde:
- `[URL] `- adresa completă a fișierului pe care dorim să-l descărcăm.

### De ce este necesar să creați pentru fiecare site baza de date și utilizatorul său?
Dacă fiecare site are propria sa bază de date și utilizator, atunci accesul la datele unui site este restricționat doar la acel utilizator. Astfel, chiar dacă un site este compromis, atacatorul nu va putea accesa sau modifica datele altor site-uri care rulează pe aceeași mașină sau server.

### Cum se poate schimba accesul la sistemul de gestionare a bazei de date pe portul 1234?
##############################################
### Care sunt avantajele, din punctul dvs. de vedere, ale virtualizării?
Virtualizarea se evidențiază prin următoarele avantaje:
- __Implementare rapidă__. Crearea și implementarea de noi mașini virtuale este mult mai rapidă decât achiziționarea și configurarea unui server fizic nou.
- __Securitate îmbunătățită__. Virtualizarea poate oferi un nivel suplimentar de securitate prin izolarea mașinilor virtuale.
- __Recuperare în caz de dezastru__. Virtualizarea facilitează recuperarea rapidă a sistemelor în caz de dezastru, deoarece mașinile virtuale pot fi restaurate pe un alt server fizic.

### Pentru ce este necesar să se stabilească ora/zona de timp pe server?
Setarea corectă a orei și a zonei de timp pe un server este esențială pentru automatizarea proceselor, acuratețea înregistrărilor și a jurnalelor, funcționarea corectă a aplicațiilor și a serviciilor.

### Cât spațiu ocupă instalarea OS (disc virtual) pe mașina gazdă?
Spațiul ocupat de discul virtual pe mașina virtuală este de 8 GB.

### Care sunt recomandările privind partiționarea discului pentru servere? De ce se recomandă să se partiționeze discul în acest fel?
Recomandările generale de partiționare includ:
- __partiția rădăcină (/)__.
- __partiția swap__.
- __partiția /home__.
- __partiția /var__.
- __partiția /opt__.
- __partiția /data__.

__Partiția rădăcină (/)__ este partiția principală unde este instalat sistemul de operare și este recomandat să aibă o dimensiune suficientă pentru a găzdui sistemul de operare, fișierele de configurare și aplicațiile esențiale. 

__Partiția swap__ este o partiție dedicată memoriei virtuale (swap) și este utilizată de sistemul de operare pentru a stoca temporar datele care nu încap în memoria RAM. 

__Partiția /home__ este partiția unde sunt stocate fișierele utilizatorilor (documente, imagini etc.). Separarea acestei partiții de cea rădăcină permite o gestionare mai ușoară a datelor utilizatorilor și protecția acestora în cazul unui eșec al sistemului de operare. 

__Partiția /var__ este partiția unde sunt stocate fișierele variabile ale sistemului, cum ar fi jurnalele (log-urile) și fișierele temporare.

__Partiția /opt__ este partiția unde sunt instalate de obicei aplicațiile software terțe.

__Partiția /data__ este partiția unde sunt stocate datele utilizatorilor sau ale aplicațiilor. 

Partiționarea discului aduce numeroase avantaje, cum ar fi o mai bună organizare a fișierelor, securitate sporită prin separarea fișierelor sistemului de operare, a aplicațiilor și a datelor utilizatorilor, întreținere mai ușoară a sistemului de operare și recuperare în caz de dezastru a datelor.

## Concluzii.


## Bibliografie.




