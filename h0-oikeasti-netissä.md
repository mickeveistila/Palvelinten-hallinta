# h0 - Oikeasti Netissä

## x) Lue ja tiivistä 

Niemi 2025: H4 Tehtävät: 
- Pilvipalvelimen vuokraus: VPS-palvelimen (UpCloud) vuokraaminen. Operating system Debian GNU/linux 13 (trixie). SSH avaimet luominen - ssh-keygen terminaalissa.   
- Virtuaalipalvelimen alkutoimet: Etäyhteyden virtuaalipalvelimeen. sh root@195.12.12.12 - mkdir -p /home/aatu/.ssh - chmod 700 /home/aatu/.ssh. Lisätään public key nano /home/aatu/.ssh/authorized_keys. 
- Palvelimen suojaus: Palomuurin asennus ja portin avaaminen. apt-get install ufw - ufw allow 22/tcp. Palomuurin tilan tarkistus ufw status.  
- Web-palvelin asennus: Asennetaan Apache2 - sudo apt-get install apache2. Sivun muokkaaminen - sudo nano /var/www/html/index.html. Portin 80 läpipääsy - sudo ufw allow 80/tcp ja sudo ufw enable

Partanen 2025: Harjoitus 5: Nimekäs. 
- Domain nimen hankinta : Namecheap palvelusta halvin mahdollinen domain annetulla nimellä. Kassalla domain privacy suojaus päälle ilmaiseksi.  
- Osoite osoittamaan omaan koneesee : Valitaan haluttu nimi ja Advanced DNS. CNAME tietueen poisto ja oman A record kohdan lisäys osoittamaan virtuaalipalvelimelle.

## a) Paikallista. Asenna Debian 13 paikallisesti virtuaalikoneeseen

Latasin Debian 13 (Trixie) ARM64 -levykuvan, koskan käytän Macbookia Apple m4 chipillä. Loin VirtualBoxiin uuden virtuaalikoneen. Laitoin virtuaalikoneen verkkokortin asetuksista Bridged Adapter tilaan, jotta virtuaalikone näkyisi lähiverkossa.  
Suoritin käyttöjärjestelmän asennuksen ilman graafista työpöytää. Annoin sille RAM 4096 MB, prosessoreita 2 ja disc size 20 GB.

<img width="853" height="767" alt="Screenshot 2026-10-23 at 14 51 59" src="https://github.com/user-attachments/assets/3b326be3-3049-42c8-9bff-61afc0c31c84" />


Seuraavaksi Käytin Boot Menussa Pummalan ohjeita seuravaasti:

Kieli: English.

Location: other.

Continent or region: Europe.

Country: Finland.

Configure locales: en_US.UTF-8.

Configure the keyboard: Finnish.

Edellä mainittujen jälkeen tulee hieman asennusta, jonka jälkeen jatketaan Configuratioo järjestyksessä.

Hostname: debian.

Domain name: - Jätin tämän tyhjäksi, koska kyseessä on paikallinen virtuaalikone omassa kotiverkossasi.

Root password: jätetään tyhjäksi.

Full name for the new user: micke veistila.

Username for your account: micke.

Password: valitaan vahva salasana, jota ei mainita tässä ohjeessa.

Partition disks: Guided - use entire disk.

Partition disks: valitaan ainut vaihtoehto - SCSI1 (0,0,0) (sda) - 21.5 GB VBOX HARDDISK.

Partition disks: All files in one partition.

Painetaan valintaa: Finish partitioning and write changes to disk. 

Lisäksi minulla tuli software selection johon laitoin vain SSH server ja standard system utilities.  

Seuraavaksi tuli asennus ja sen jälkeen pääsin kirjautumaan sisään luomillani tunnuksilla. 

## f) Paikallinen weppipalvelin ja nimipalvelun simulointi

Selvitin virtuaalikoneeni IP-osoitteen komennolla ip a osoite oli 192.168.1.203. Simuloin nimipalvelua macbookillani muokkaamalla sen hosts-tiedostoa, jotta selain osaa yhdistää nimen virtuaalikoneeseeni - sudo nano /etc/hosts.  
Lisäsin tiedoston loppuun rivin 192.168.1.203 mickensivu.test

Päivitin pakettilistat ja asensin Apache-weppipalvelimen - sudo apt-get update - sudo apt-get install apache2

Debian-asennukseni oli ilmeisesti hieman suppea, joten asensin palomuurin erikseen ja sallin weppiliikenteen (portti 80) sekä SSH-yhteydet (portti 22) - sudo apt-get install ufw - sudo ufw allow 80/tcp - sudo ufw allow ssh - sudo ufw enable.  

Teron ohjeiden mukaisesti käytin userdir, jotta voin julkaista sivuja omasta kotihakemistostani ilman pääkäyttäjän oikeuksia - sudo a2enmod userdir - sudo systemctl restart apache2.  

Loin kotihakemistooni weppikansion ja sinne testisivun. Varmistin, että olen kotihakemistossa pwd. mkdir public_html - nano public_html/index.html. Tänne kirjoitin "Toimii!". Testasin selaimeen IP-osoitetta, http://mickensivu.test/ ja http://mickensivu.test/~micke/. Sain lopulta kaiken toimimaan.

Haasteita oli yhteyden muodostamisessa, kun yritin avata sivua selaimessa http://mickensivu.test/~micke/. Sain Forbidden virheen. Debianin tiukkojen oletusoikeuksien vuoksi Apachella ei ollut pääsyä kotikansiooni. Ratkaisin tämän antamalla weppipalvelimelle kulkuoikeuden kotihakemistooni ja lukuoikeuden weppikansioon - chmod a+x /home/micke - chmod a+rx /home/micke/public_html.  
Yhteys katkeili monesti myös kun yritin avata sivua. Virtuaalikoneen uudelleen käynnistys sai sivun latautumaan onnistuneesti. Tämä jäi epäselväksi miksi näin tapahtuu.

<img width="848" height="210" alt="Screenshot 2026-10-26 at 19 25 00" src="https://github.com/user-attachments/assets/1ebd87cd-0414-43b9-adb3-3518befea92a" />
<img width="845" height="113" alt="Screenshot 2026-10-26 at 19 40 03" src="https://github.com/user-attachments/assets/e559effe-41f7-4308-8ed2-ef0d8fbf7570" />



## Lähteet

Karvinen, T. 2006. Raportin kirjoittaminen. Luettavissa: https://terokarvinen.com/2006/raportin-kirjoittaminen-4/. Luettu: 26.10.2026

Karvinen, T. 2021. install debian on virtual box. Luettavissa: https://terokarvinen.com/2021/install-debian-on-virtualbox/. Luettu: 26.10.2026

Karvinen, T. 2008. Install Apache Web Server on Ubuntu. Luettavissa: https://terokarvinen.com/2008/05/02/install-apache-web-server-on-ubuntu-4/index.html. Luettu: 26.10.2026

Karvinen, T. 2021. Install Debian on Virtualbox - Updated 2024. Luettavissa: https://terokarvinen.com/2021/install-debian-on-virtualbox/#. Luettu: 26.10.2026

Punnala, T. 2025. h1-installing-linux.md. GitHub. Luettavissa: https://github.com/punnalathomas/linux-course/blob/main/h1-installing-linux.md. Luettu: 26.10.2026

