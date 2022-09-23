# Mehrere-Nightscout-Instanzen-auf-einem-V-Server-ohne-Docker-Container-LXC
ohne Docker/LXC/Container mehrere Nightscout Instanzen auf einem VPS installieren

Anleitung mit Bildern unter:
https://www.michael-schloemp.de/2022/09/20/mehrere-nightscout-instanzen-auf-einem-v-server-ohne-docker-container/

Wie kann man „unkompliziert“ mehrere Nightscout Instanzen auf einem V-Server installieren?
Lange habe ich Versucht mehrere Nightscout Instanzen auf einem V-Server zum laufen zu bekommen.
In den gängigen Communitys wurde immer auf Docker, Docker Compose und Container Visualisierung gesetzt.
Als Anfänger bzw. als Container/Docker Neuling gar nicht so einfach. Doch es geht auch ganz ohne Docker/Container.
In dieser Anleitung zeige ich Dir meinen Weg der Realisierung, wie man mehrere Instanzen auf einem V-Server ohne Docker installiert. Dazu ist nur eine kleine Änderung an Nightscout notwendig.
Selbstverständlich ist es geübten Docker Nutzern umständlich, für mich jedoch eine einfache alternative, wenn der Server ausschließlich für Nightscout Verwendet wird.

Jeder der diese Anleitung anwendet, handelt eigenverantwortlich.

Benötigtes Material:
Nightscout -> https://github.com/nightscout/cgm-remote-monitor
Putty -> https://www.putty.org/
Programmers Notepad -> https://www.pnotepad.org/
Filezilla oder gleichwertiges FTP Programm -> https://filezilla-project.org/
Beliebigen V-Server mit Ubuntu 20.04 LTS.

Für diese Tutorial habe ich einen VPS R8 von 1blu.de verwendet.

 



 

Als erstes Loggen wir uns mit Putty auf dem Server mit dem Root Benutzer ein. In der gesamten Anleitung gehe ich von einer Neuisnstallation des Servers aus.

Apache 2 deinstallieren

Den Apache2 Service stoppen:

sudo service apache2 stop

Deinstallation durchführen:

sudo apt-get purge apache2 apache2-utils

Ggf. noch vorhandene Vorkommen löschen:

sudo rm -rf /etc/apache2
sudo rm -rf /usr/sbin/apache2
sudo rm -rf /usr/lib/apache2
sudo rm -rf /etc/apache2
sudo rm -rf /usr/share/apache2
sudo rm -rf /usr/share/man/man8/apache2.8.gz

Kontrolle ob noch Dateien und Verzeichnisse von Apache auf dem Server sind

whereis apache2

Ubuntu aktualisieren und benötigte Komponenten installieren

apt update
apt upgrade

apt install ufw
apt install nginx
apt-get install nano

apt install python
apt install gcc

Mongo Datenbank installieren

apt install mongodb

Mongo DB Status überprüfen

$ systemctl status mongodb

Ausgabe sollte so aussehen:

● mongodb.service – An object/document-oriented database
Loaded: loaded (/lib/systemd/system/mongodb.service; enabled; vendor preset: enabled)
Active: active (running) since Tue 2022-09-20 18:27:57 UTC; 21s ago

In dieser Anleitung werde ich 3 Nightscout Instanzen zur Veranschaulichung installieren.
Es sind auch noch mehr möglich, jedoch sollte der V-Server auch genug Leistung haben.
Wie viele Tatsächlich stabil und Performant laufen, habe ich nicht getestet.
Theoretisch sollten bei meinem V-Server, verglichen zum Raspberry Pi1b, mindestens 10 Instanzen möglich sein.

Mongo Datenbanken erstellen

Wir erstellen direkt alle Datenbanken die benötigt werden.

Erste Datenbank:

$ mongo
> use Nightscout
> db.createUser({user: "mainuser", pwd: "passwort", roles:["readWrite"]})
> quit()

Zweite Datenbank:

$ mongo
> use Nightscout2
> db.createUser({user: "mainuser2", pwd: "passwort", roles:["readWrite"]})
> quit()

dritte Datenbank:

$ mongo
> use Nightscout3
> db.createUser({user: "mainuser3", pwd: "passwort", roles:["readWrite"]})
> quit()

Datenbank Namen und Passwörter merken/notieren!

Ersten Benutzer und erste Instanz erstellen

adduser mainuser
usermod -aG sudo mainuser
su mainuser
grep '^sudo' /etc/group

Node JS installieren und Einrichten


sudo apt install nodejs
sudo apt install build-essential checkinstall
sudo apt install libssl-dev
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

– restart konsole – login mit mainuser –


source /etc/profile
$ nvm ls-remote
$ nvm install 14.18.1
$ nvm list
$ nvm use 14.18.1
sudo apt install npm

Prüfen in welchem Verzeichnis man sich befindet:

$ pwd
> /home/mainuser

Wenn nicht im home/mainuser verzeichnis

§ cd /home/mainuser

Nightscout von Github herunterladen und auf Server laden.

Als erstes Nighscout Dateien auf GitHub als zip Datei herunterladen: https://github.com/nightscout/cgm-remote-monitor


Github Zip
Das heruntergeladene Verzeichnis auf dem PC entpacken.

FTP Programm (Filezilla) öffnen und mit Server verbinden.
Login auf Server mit Root.
Im FTP Verzeichnis /home/mainuser wechseln und den entpackten Nightscout Ordner in das /home/mainuser Verzeichnis hochladen.
Dem Nightscout Verzeichnis „cgm-remote-monitor-master“ chmod 777 geben.

Wenn alles korrekt hochgeladen wurde, zurück nach Putty wechseln.

Ins Nightscout Verzeichnis wechseln und Installation ausführen.

$ cd cgm-remote-monitor-master
$ npm install

Warten bis installation Vollständig abgeschlossen

Nightscout konfiguieren

Mongo Zugangsdaten und Api-Secret anpassen!

$ nano start.sh -> inhalt einfügen

#!/bin/bash

# environment variables
export DISPLAY_UNITS="mg/dl"
export MONGO_CONNECTION="mongodb://mainuser:passwort@localhost:27017/Nightscout"
export BASE_URL="127.0.0.1:1337"
export API_SECRET="12-stelliger-Code"
export PUMP_FIELDS="reservoir battery status"
export DEVICESTATUS_ADVANCED=true
export ENABLE="careportal loop iob cob openaps pump bwg rawbg basal cors direction timeago devicestatus ar2 profile boluscalc food sage iage cage alexa basalprofile bgi directions bage upbat googlehome errorcodes reservoir battery openapsbasal"
export TIME_FORMAT=24
export INSECURE_USE_HTTP=true
export LANGUAGE=de
export EDIT_MODE=on
export PUMP_ENABLE_ALERTS=true
export PUMP_FIELDS="reservoir battery clock status"
export PUMP_RETRO_FIELDS="reservoir battery clock"
export PUMP_WARN_CLOCK=30
export PUMP_URGENT_CLOCK=60
export PUMP_WARN_RES=50
export PUMP_URGENT_RES=10
export PUMP_WARN_BATT_P=30
export PUMP_URGENT_BATT_P=20
export PUMP_WARN_BATT_V=1.35
export PUMP_URGENT_BATT_V=1.30
export OPENAPS_ENABLE_ALERTS=false
export OPENAPS_WARN=30
export OPENAPS_URGENT=60
export OPENAPS_FIELDS="status-symbol status-label iob meal-assist rssi freq"
export OPENAPS_RETRO_FIELDS="status-symbol status-label iob meal-assist rssi"
export LOOP_ENABLE_ALERTS=false
export LOOP_WARN=30
export LOOP_URGENT=60
export SHOW_PLUGINS=careportal
export SHOW_FORECAST="ar2 openaps"

# start server
/home/mainuser/.nvm/versions/node/v14.18.1/bin/node server.js

Mit strg+c beenden und mit Y das speichern bestätigen.

Der start.sh erforderliche Rechte geben:

$ chmod 775 start.sh

Danach die start.sh testen.

./start.sh

Wenn die start.sh korrekt läuft erscheint als letzte Meldung in Grüner Schrift „WS:…“

Mit strg+c beenden wir die Anzeige.

Nightscout Service einrichten

$ sudo nano /etc/systemd/system/nightscout.service

einfügen und speichern:

[Unit]
Description=Nightscout Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/mainuser/cgm-remote-monitor-master
ExecStart=/home/mainuser/cgm-remote-monitor-master/start.sh

[Install]
WantedBy=multi-user.target

Speichern und beenden mit: „strg+X“ – „Y“ – „enter“

Reload systemd:

$ sudo systemctl daemon-reload

Nigtscout Service aktivieren und starten:

$ sudo systemctl enable nightscout.service
$ sudo systemctl start nightscout.service

Nightscout Service Status anzeigen lassen:

$ sudo systemctl status nightscout.service

Ausgabe:

[…] Active: active (running)

Domain dem Host zuweisen

Hauptdomain und alle Subdomains für alle Instanzen in der Host Datei einpflegen.

$ sudo nano /etc/hosts


Etc/Host Bearbeiten
Speichern mit „strg+X“ – „Y“ – „enter“

Nginx config (Virtual Host) anlegen

Zu Erstlegen wir eine config für die Hauptdomain an und anschließend für die erste Instanz (um die anderen Instanzen kümmern wir uns später).

Hauptdomain konfiguieren (server_name eintragen)

sudo nano /etc/nginx/sites-available/domain.de.conf

Diesen Abschnitt einfügen:

server {

root /var/www/html;

server_name domain.de;
server_name www.domain.de;
location / {
try_files $uri $uri/ =404;
}
}

strg+x mit „y“ speichern

Konfiguration aktivieren:

sudo ln -s /etc/nginx/sites-available/domain.de.conf /etc/nginx/sites-enabled/

Default config löschen:

cd

cd /etc/nginx/

sudo rm sites-available/default
sudo rm sites-enabled/default

cd

Subdomain inkl. Portweiterleitung einrichten.

sudo nano /etc/nginx/sites-available/ns1.domain.de.conf

Diesen Abschnitt einfügen:

server {
listen 80;

server_name ns1.domain.de;
server_name www.ns1.domain.de;

location / {
proxy_pass http://127.0.0.1:1337;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header X-Forwarded-Proto "https";
}
}

Konfiguration aktivieren

sudo ln -s /etc/nginx/sites-available/ns1.domain.de.conf /etc/nginx/sites-enabled/

nginx neustarten

sudo service nginx restart

Prüfen ob Nginx korrekt läuft

systemctl status nginx

Logout, Login mit Root Benutzer

Zweiten Benutzer Einrichten und sudo Rechte erteilen

adduser mainuser2
usermod -aG sudo mainuser2
su mainuser2

grep '^sudo' /etc/group

Node JS Einrichten

sudo apt install nodejs
sudo apt install build-essential checkinstall
$ sudo apt install libssl-dev
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

– restart konsole – login mit mainuser2 –

source /etc/profile
$ nvm ls-remote
$ nvm install 14.18.1
$ nvm list
$ nvm use 14.18.1

 

Prüfen in welchem Verzeichnis man sich befindet:

$ pwd
> /home/mainuser2

Wenn nicht im home/mainuser2 verzeichnis

§ cd /home/mainuser2

Nightscout Instanz 2 installieren

Auf dem PC in den entpackten Nightscout Ordner gehen, ins Verzeichnis lib->server wechseln.
Hier befindet sich eine Datei namens „env.js“.


Env.Js Im Nightscout Ordner Suchen
Mit Programmers Notepad die Datei env.js öffnen und in Zeile 36 den Port ändern.
(Ports im Bereich 32768 – 65535 sollten frei zur Verfügung stehen.)
In dieser Anleitung Verwende ich für die zweite Installation den Port 36363.


Zeile 36 Port Ändern
Nach der Port Änderung die Datei speichern.

FTP Login auf Server mit Root.
Ins FTP Verzeichnis /home/mainuser2 wechseln und den Nightscout Ordner auf den Server laden.
Dem Nightscout Verzeichnis chmod 777 vor der Installation vergeben


Rechtsklick Auf Nightscout Ordner

777 Eingeben
Zurück zu Putty

Wechsel in das Verzeichnis von Nightscout

$ cd cgm-remote-monitor-master

Installation starten

$ npm install

Warten bis die Installation Vollständig abgeschlossen ist, anschließen die start.sh bearbeiten.

$ nano start.sh
-> inhalt einfügen

#!/bin/bash

# environment variables
export DISPLAY_UNITS="mg/dl"
export MONGO_CONNECTION="mongodb://mainuser2:passwort@localhost:27017/Nightscout2"
export BASE_URL="127.0.0.1:36363"
export API_SECRET="12-stelliger-code"
export PUMP_FIELDS="reservoir battery status"
export DEVICESTATUS_ADVANCED=true
export ENABLE="careportal loop iob cob openaps pump bwg rawbg basal cors direction timeago devicestatus ar2 profile boluscalc food sage iage cage alexa basalprofile bgi directions bage upbat googlehome errorcodes reservoir battery openapsbasal"
export TIME_FORMAT=24
export INSECURE_USE_HTTP=true
export LANGUAGE=de
export EDIT_MODE=on
export PUMP_ENABLE_ALERTS=true
export PUMP_FIELDS="reservoir battery clock status"
export PUMP_RETRO_FIELDS="reservoir battery clock"
export PUMP_WARN_CLOCK=30
export PUMP_URGENT_CLOCK=60
export PUMP_WARN_RES=50
export PUMP_URGENT_RES=10
export PUMP_WARN_BATT_P=30
export PUMP_URGENT_BATT_P=20
export PUMP_WARN_BATT_V=1.35
export PUMP_URGENT_BATT_V=1.30
export OPENAPS_ENABLE_ALERTS=false
export OPENAPS_WARN=30
export OPENAPS_URGENT=60
export OPENAPS_FIELDS="status-symbol status-label iob meal-assist rssi freq"
export OPENAPS_RETRO_FIELDS="status-symbol status-label iob meal-assist rssi"
export LOOP_ENABLE_ALERTS=false
export LOOP_WARN=30
export LOOP_URGENT=60
export SHOW_PLUGINS=careportal
export SHOW_FORECAST="ar2 openaps"

# start server
/home/mainuser2/.nvm/versions/node/v14.18.1/bin/node server.js

strg+c -> mit „Y“ speichern bestätigen

Dateiberechtigung der start.sh ändern:

$ chmod 775 start.sh

Anschließend die start.sh einmal ausführen:

./start.sh

Wenn die start.sh korrekt läuft erscheint als letzte Meldung in Grün-Schrift WS:…

Mit strg+c beenden wir die Anzeige.

Nightscout Service einrichten

Damit die zweite Installation ebenfalls läuft, wird eine eigenständige nightscout2.service eingerichtet.

$ sudo nano /etc/systemd/system/nightscout2.service

einfügen und speichern:

[Unit]
Description=Nightscout2 Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/mainuser2/cgm-remote-monitor-master
ExecStart=/home/mainuser2/cgm-remote-monitor-master/start.sh

[Install]
WantedBy=multi-user.target

Speichern und beenden mit: „strg+X“ – „Y“ – „enter“
Reload systemd:

$ sudo systemctl daemon-reload

Nigtscout2 Service aktivieren und starten:

$ sudo systemctl enable nightscout2.service
$ sudo systemctl start nightscout2.service

Nightscout Service Status anzeigen lassen:

$ sudo systemctl status nightscout2.service

Ausgabe:

● nightscout2.service – Nightscout2 Service
Loaded: loaded (/etc/systemd/system/nightscout2.service; enabled; vendor preset: enabled)
Active: active (running)
[..]

Subdomain einrichten

sudo nano /etc/nginx/sites-available/ns2.domain.de.conf

kopieren und einfügen:

server {
listen 80;

server_name ns2.domain.de;
server_name www.ns2.domain.de;

location / {
proxy_pass http://127.0.0.1:36363;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header X-Forwarded-Proto "https";
}
}

Konfiguration aktivieren und nginx neustarten

sudo ln -s /etc/nginx/sites-available/ns2.domain.de.conf /etc/nginx/sites-enabled/
sudo service nginx restart

Prüfen ob Nginx korrekt läuft

systemctl status nginx

Logout, Login mit Root Benutzer

Dritten Benutzer und Instanz anlegen

adduser mainuser3
usermod -aG sudo mainuser3
su mainuser3
grep ‚^sudo‘ /etc/group

Node JS Installieren

sudo apt install nodejs
sudo apt install build-essential checkinstall
$ sudo apt install libssl-dev
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

– restart konsole – login mit mainuser3 –

source /etc/profile
$ nvm ls-remote
$ nvm install 14.18.1
$ nvm list
$ nvm use 14.18.1

Prüfen in welchem Verzeichnis man sich befindet:

$ pwd
> /home/mainuser3

Wenn nicht im home/mainuser3 verzeichnis

§ cd /home/mainuser3

Git Kopieren/Clonen

Auf dem PC wieder in den Nightscout Ordner wechseln und die env.js wie bei der zweiten Instanz den Port ändern.

Für die dritte Installation habe ich den Port 36364 verwendet.

FTP Login auf Server mit Root.
Ins FTP Verzeichnis /home/mainuser3 wechseln und Nightscout Ordner auf den Server hochladen.
Nightscout Verzeichnis chmod 777 vor der Installation vergeben (wie auch bei der zweiten Instanz)

$ cd cgm-remote-monitor-master
$ npm install

Warten bis Installation Vollständig abgeschlossen (2-3 Minuten)

$ nano start.sh -> inhalt einfügen

#!/bin/bash

# environment variables
export DISPLAY_UNITS="mg/dl"
export MONGO_CONNECTION="mongodb://mainuser3:passwort@localhost:27017/Nightscout3"
export BASE_URL="127.0.0.1:36364"
export API_SECRET="12-stelliger-code"
export PUMP_FIELDS="reservoir battery status"
export DEVICESTATUS_ADVANCED=true
export ENABLE="careportal loop iob cob openaps pump bwg rawbg basal cors direction timeago devicestatus ar2 profile boluscalc food sage iage cage alexa basalprofile bgi directions bage upbat googlehome errorcodes reservoir battery openapsbasal"
export TIME_FORMAT=24
export INSECURE_USE_HTTP=true
export LANGUAGE=de
export EDIT_MODE=on
export PUMP_ENABLE_ALERTS=true
export PUMP_FIELDS="reservoir battery clock status"
export PUMP_RETRO_FIELDS="reservoir battery clock"
export PUMP_WARN_CLOCK=30
export PUMP_URGENT_CLOCK=60
export PUMP_WARN_RES=50
export PUMP_URGENT_RES=10
export PUMP_WARN_BATT_P=30
export PUMP_URGENT_BATT_P=20
export PUMP_WARN_BATT_V=1.35
export PUMP_URGENT_BATT_V=1.30
export OPENAPS_ENABLE_ALERTS=false
export OPENAPS_WARN=30
export OPENAPS_URGENT=60
export OPENAPS_FIELDS="status-symbol status-label iob meal-assist rssi freq"
export OPENAPS_RETRO_FIELDS="status-symbol status-label iob meal-assist rssi"
export LOOP_ENABLE_ALERTS=false
export LOOP_WARN=30
export LOOP_URGENT=60
export SHOW_PLUGINS=careportal
export SHOW_FORECAST="ar2 openaps"

# start server
/home/mainuser3/.nvm/versions/node/v14.18.1/bin/node server.js

Mit strg+c schließen und mit „Y“ das speichern bestätigen.

Anschließend der start.sh wieder entsprechende Zugriffsrechte erteilen.

$ chmod 775 start.sh

Die start.sh einmal ausführen

./start.sh

Wenn die start.sh korrekt läuft erscheint als letzte Meldung in Grün-Schrift WS:…

Mit strg+c beenden wir die Anzeige.

Nightscout Service für dritte Instanz einrichten

Als nächstes muss ebenfalls eine nightscout.service Datei für die dritte Instanz erstellt werden.

$ sudo nano /etc/systemd/system/nightscout3.service

einfügen und speichern:

[Unit]
Description=Nightscout3 Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/mainuser3/cgm-remote-monitor-master
ExecStart=/home/mainuser3/cgm-remote-monitor-master/start.sh

[Install]
WantedBy=multi-user.target

Speichern und beenden mit: „strg+X“ – „Y“ – „enter“
Reload systemd:

$ sudo systemctl daemon-reload

Nigtscout3 Service aktivieren und starten:

$ sudo systemctl enable nightscout3.service
$ sudo systemctl start nightscout3.service

Nightscout Service Status anzeigen lassen:

$ sudo systemctl status nightscout3.service

Ausgabe:

● nightscout3.service – Nightscout3 Service
Loaded: loaded (/etc/systemd/system/nightscout3.service; enabled; vendor preset: enabled)
Active: active (running)
[..]

Subdomain für dritte Instanz

sudo nano /etc/nginx/sites-available/ns3.domain.de.conf

kopieren und einfügen:

server {
listen 80;

server_name ns3.domain.de;
server_name www.ns3.domain.de;

location / {
proxy_pass http://127.0.0.1:36364;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header X-Forwarded-Proto "https";
}
}

Konfiguration aktivieren und nginx neustarten.

sudo ln -s /etc/nginx/sites-available/ns3.domain.de.conf /etc/nginx/sites-enabled/
sudo service nginx restart

Prüfen ob Nginx korrekt läuft

systemctl status nginx

Login mit Root Account

Certbot nginx installieren

Falls noch nicht installiert, certbot-nginx installieren:

sudo apt install certbot python3-certbot-nginx

SSL-Zertifikate anfordern

Wir beginnen mit der Hauptdomain und im weiteren Verlauf werden die SSL_Zertifikate für die Subdomains angefordert.

sudo certbot --nginx -d domain.de -d www.domain.de

E-Mail Adresse eingeben
Mit „A“ Bedingungen akzeptieren
Werbung mit N für Nein oder J für Ja
2 für automatische weiterleitung

sudo certbot --nginx -d ns1.domain.de -d www.ns1.domain.de
mit 2 weiterleitung bestätigen

sudo certbot --nginx -d ns2.domain.de -d www.ns2.domain.de
mit 2 weiterleitung bestätigen

sudo certbot --nginx -d ns3.domain.de -d www.ns3.domain.de
mit 2 weiterleitung bestätigen

Certbot Timer prüfen:

sudo systemctl status certbot.timer

Erneuerungsprozess testen

sudo certbot renew --dry-run

Firewall konfigurieren und aktivieren

Im ersten Schritt geben wir der Firewall (UFW) die entsprechenden Berechtigungen/Portfreigaben.

ufw allow 1337
ufw allow 36363
ufw allow 36364
ufw allow 'Nginx Full'
ufw allow 27017
ufw allow OpenSSH

Firewall aktivieren

ufw enable

Anschließend Kontrolle ob alles korrekt ist wie oben angeben.

$ sudo ufw status

(Status und freigegebene Ports einsehen)

Fertig. 3 laufende Nightscout Instanzen auf einem Server ohne zusätzliche andere Komponenten wie Docker, LXC oder ähnliche Container Dienste.

Update Hinweis:
Sollte eine neue Nightscout Version herauskommen, so muss jede Instanz einzeln Upgedated werden.
Vor einem Update muss zwingend die start.sh und die geänderte env.js gesichert werden!
Die Update Anleitung  Nightscout Updates installieren muss enstprechend auf die einzelnen Instanzen bzgl der nightscout.service und dem Nightscout Verzeichnis geänder werden.

