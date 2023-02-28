# h11_prod


## Ympäristö

Hyper-V kotikoneella (Host):

- CPU: i5-9600K
- RAM: 16Gb
- HDD: 120Gb
- Win 11 Pro x64

Virtuaalikoneen speksit:

- 2 x CPU
- 4 Gb RAM
- 50 Gb HDD
- Generation 2 (Hyper-V pyytää määrittelemään)


## Aloitus 
Aloitin työt käynnistämällä Debianin Hyper-V hostillani 28.2.2023 klo 19:37. Tehtävän tarkoituksena oli tutustua Djangon tuotantopalvelimen konfigurointiin ja jatkaa täten edellisviikon Django harjoitteita.

## a) Tee Djangon tuotantoasennus.
Käytin tehtävää varten opettajamme Tero Karvisen ohjeistusta Djangon tuotantoon asennuksesta. Ihan ensimmäisenä päivitin repositoriot komennolla ```sudo apt-get update```, jonka jälkeen asensin vielä Microon bash-completionin ```sudo apt-get -y install micro bash-completion```, jotta saan hyödynnettyä tabulatoria komentojen täydennykseen. Samassa vielä asetin oletus tekstieditoriksi micron komennolla ```export EDITOR=micro```. Ohjeessa puhuttiin seuraavaksi Apache2:n asennuksesta, mutta jätin sen väliin, koska virtuaalikoneella oli jo Apache2 asennettuna & konfiguroituna. Siirryin siis suoraan Djangon asennukseen VirtualEnv-ympäristössä.

### Uuden VirtualHostin luonti ja konfigurointi tuotantoympäristöä varten
Klo 19:42 </br>
Loin ensiksi kansion käyttäjän juureen ```mkdir -p publicwsgi/yritysoy/static/``` ja myös vähän kansion täytettä ```echo "ehkä näet, ehkä et"|tee publicwsgi/yritysoy/static/index.html```. Juuri tekemäni asia loi siis staattisen html sivun kansioon ```publicwsgi/yritysoy/static/```, tätä käytettiin myöhemmin osana harjoitusta. </br>
![Kuva1](https://user-images.githubusercontent.com/122887740/221936350-19f44dcb-0a42-4648-86cb-d8d663c38d94.png)</br>

Kansion ja sivuston luonnin jälkeen siirryin luomaan Apache2:n konfiguraatiotiedoston osoitteessa: ```/etc/apache2/sites-available/``` komennolla ```sudoedit /etc/apache2/sites-available/yritysoy.conf``` ja syötin sinne seuraavanlaiset määritteet: </br>
![Kuva2](https://user-images.githubusercontent.com/122887740/221938389-cae9508d-1697-4f57-be93-e0c39916c620.png)</br>

Seuraavana oli luvassa asettaa juuri luotu .conf tiedosto Apachen oletussivuksi ja samalla ottaa vanha oletussivu pois käytöstä komennoilla: 
```
$ sudo a2ensite yritysoy.conf
$ sudo a2dissite frontpage.conf 
$ /sbin/apache2ctl configtest
$ sudo systemctl restart apache2
$ curl http://localhost/static/
```


![Kuva3](https://user-images.githubusercontent.com/122887740/221940151-39e6f5bb-5e93-42a1-95d9-6ae30eee10a7.png) </br>

Kuten kuvassa näkyy, sain konfiguroitua ja testattua sivuston muutoksen ongelmitta. Testi toki ilmoitti, että se ei voi määritellä palvelimen FQDN:ää, mutta se johtuu täysin olemattomista DNS asetuksista, joita ei tässä tehtävässä tarvinnut ottaa huomioon.


### VirtualEnv ympäristön asennus ja konfigurointi tuotantoa varten
Klo 20:03</br>
Sivusto konfigurointien jälkeen siirryin luomaan uutta virtuaalista ympäristö tulevalle Djangon käyttöönotolle.
Aloitin ajamalla ```sudo apt-get -y install virtualenv``` tarkistaakseni, löytyisikö edellisellä kerralla asennetulle virtualenville päivityksiä. </br>
![Kuva4](https://user-images.githubusercontent.com/122887740/221940954-f29c7157-e1cc-4bba-b84d-32369b6a654e.png)</br>
Päivityksiä ei näyttänyt olevan, joten siirryin seuraavaan osioon.


Navigoin uuteen kansioon ```publicwsgi/``` määritelläkseni uuden virtuaaliympäristön Pythonille ja ajoin komennon ```virtualenv -p python3 --system-site-packages env```</br>
![Kuva5](https://user-images.githubusercontent.com/122887740/221941329-9a929efb-a7e0-4af5-a2a9-c2054a3cb3df.png)</br>

Virtuaaliympäristön luonnista toipuneena siirryin itse virtuaaliympäristöön asentamaan Djangon siihen. Aktivoin virtuaaliympäristön ```source env/bin/activate``` ja heti perään varmistin, että olen oikeassa paikassa ```which pip```:</br>
![Kuva6](https://user-images.githubusercontent.com/122887740/221941945-20d1f85b-6810-4023-92d1-7b5c9b586080.png)</br>
Oikea ympäristö oli aktivoituna! </br>


Nyt kun alustukset oli tehty ja tarkistettu, oli aika siirtyä luomaan määritystiedostoa asennukselle komennolla ```micro requirements.txt``` ja syöttämällä sinne yksi rivi ```django```. Määritystiedoston luonnin jälkeen oli vuorossa itse asennus komennolla ```pip install -r requirements.txt```. Lopulta oli vielä asennetun Djangon version tarkistus ```django-admin --version```: </br>
![Kuva7](https://user-images.githubusercontent.com/122887740/221942700-ecb1eb89-b7a7-4025-9cac-01d5ea0b0f3e.png)</br>

Django oli siis asennettu oikein. 


### Djangon määrittely
Hyppäsin ensiksi Django projektin luonnin yli, koska se oli jo tehty aiemmin. Siirryin siis suoraan Apache2:n määritystiedoston jatkokonfigurointiin ```sudoedit /etc/apache2/sites-available/yritysoy.conf``` ja syöttämällä siihen seuraavanlaiset määritteet: </br>
![Kuva8](https://user-images.githubusercontent.com/122887740/221943831-51172aee-4097-4d0d-aa83-a63b2f8caf9d.png)</br>

Määritteiden jälkeen oli aika siirtyä asentamaan WSGI moduuli Apachea varten ```sudo apt-get -y install libapache2-mod-wsgi-py3```: </br>
![Kuva9](https://user-images.githubusercontent.com/122887740/221944185-4ee72b78-43fa-4d8d-a4af-600ab15ac913.png)</br>

ja tarkistamalla syntaksi: </br>
![Kuva10](https://user-images.githubusercontent.com/122887740/221944485-8f458afb-c8de-44a9-a5db-fa8e5a9d20a9.png)
Hyvälle näyttää ja eikun uudelleenkäynnistämään apache2 komennolla ```sudo systemctl restart apache2```!

Uudelleenkäynnistyksen jälkeen testit kehiin: </br>
![Kuva11](https://user-images.githubusercontent.com/122887740/221949789-3834afe7-a114-4b69-ba10-5410940bef0d.png)</br>
![Kuva12](https://user-images.githubusercontent.com/122887740/221949798-210c2a6c-557c-4082-9bde-29eaed5d8a61.png)</br>

Ai että on se ihanaa, kun hommat pelittävät!

### Disable DEBUG
Klo 20:48</br>
Viimeisenä osiona oli vielä kytkeä pois päältä Debug - mode.


## Lopetus


## Lähteet:
Karvinen, Tero, 2022 - Deploy Django 4 - Production Install (https://terokarvinen.com/2022/deploy-django/)
