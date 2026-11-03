# h1 - Hei Ansiblen maailma

## x) Lue ja tiivistä

Karvinen 2026: SSH public key - Login without password :

- SSH-avaimen luonti ja kopiointi, jotta voi kirjautua sisään ilman salasanan kirjoittamista
- ssh-keygen: Luo julkisen ja salaisen avainparin
- ssh-copy-id localhost: Kopioi julkisen avaimen kohdekoneelle
- ssh localhost: Yhteyden otto koneeseen

Karvinen 2026: Hello Ansible

- Ansiblen avulla voi automatisoida palvelinten asennukset ja määritykset
- Ansible toimii SSH kautta. Orjilla tarvitsee olla SSH ja Python
- ansible-playbook site.yml: Käskee Ansiblea lukemaan koodatut ohjeet ja toteuttamaan ne kohdekoneilla

# a) Paikallista. Asenna Debian 13 paikallisesti virtuaalikoneeseen

Raportoin tämän h0 yhteydessä: https://github.com/mickeveistila/Palvelinten-hallinta/blob/main/h0-oikeasti-netiss%C3%A4.md

# b) Sshecrets. Asenna SSH-demoni käsin ja testaa se kirjautumalla SSH:lla

Ensin päivitin pakettilistat ja asensin OpenSSH-palvelimen, jotta virtuaalikoneeseen voi ottaa etäyhteyden. Asetin palvelimen käynnistymään automaattisesti.
- sudo apt-get update - sudo apt-get -y install ssh - sudo systemctl enable --now ssh

Testasin yhteyden ottamalla SSH-yhteyden omaan koneeseen localhost ja poistuin - ssh localhost - exit

# c) Pubkey. Automatisoi ssh-kirjautuminen julkisella avaimella

Sitten loin avainparin - ssh-keygen. Painoin 3 kertaa enteriä hyväksyäkseni oletus asetukset. Sen jälkeen kopioin äsken luomani avaimen - ssh-copy-id localhost.
Testasin lopuksi toimiiko automaattinen kirjautuminen ilman salasana syöttämistä - ssh localhost

<img width="417" height="210" alt="Screenshot 2026-11-03 at 15 16 45" src="https://github.com/user-attachments/assets/a3c5e265-a0e0-4c20-8882-a1c184c59170" />

# d) Hei Ansible. Tee hei maailma ansiblella ja kokeile sitä SSH:n yli

Seuraavaksi luon Ansiblella automaation, joka kirjoittaa tekstitiedoston kohdekoneelle eli localhost. Ensin asensin Ansiblen ja pari apuohjelmaa työn helpottamiseksi - sudo apt-get update
sudo apt-get install ansible micro bash-completion tree

Asennuksen jälkeen tein kotihakemistooni Ansiblelle kansion ja siirryin kansioon - mkdir ansible/ - cd ansible/.

Seuraavaksi tein Ansiblelle hosts.ini tiedoston - micro hosts.ini. Tekstieditori aukesi ja lisäsin ensimmäiselle riville "localhost".

Testasin Ansiblen toimintaa hakemalla koneen päälläoloajan - ansible all -a 'uptime' -i hosts.ini. Tuloste oli sama kuin Teron ohjeissa ja Python ilmoituksen sain pois seuraavasti -
micro hosts.ini. Lisäsin uusille riveille [all:vars] - ansible_python_interpreter=/usr/bin/python3. Varoitus Pythonista katosi kun annoin uudestaan komennon - ansible all -a 'uptime' -i hosts.ini


<img width="329" height="66" alt="Screenshot 2026-11-03 at 18 13 21" src="https://github.com/user-attachments/assets/22d40955-dff3-4d98-bf51-c39c88b99bdd" />

Jotta ei tarvitsisi jatkossa kirjoittaa -i hosts.ini jokaisen komennon perään niin lisäsin hosts.ini ansible.cfg. - micro ansible.cfg ja uusille riveille kirjoitin "[defaults]",
"inventory = hosts.ini". Lyhyempi komento toimii nyt jatkossa.

<img width="333" height="64" alt="Screenshot 2026-11-03 at 18 22 25" src="https://github.com/user-attachments/assets/ecc2d6e6-75d4-4a35-a8bd-daab24e150e9" />

Tämän jälkeen loin pääohjelman eli playbookin site.yml, joka määrittää mitä rooleja koneelle/koneille asennetaan - micro site.yml -> 

- hosts: all
  roles:
    - hello
 
Ajetaan uusi playbook - ansible-playbook site.yml. Tuli samat virheiloitukset kuin Teron ohjeissa. 

Korjasin tilanteen luomalla hello-roolille vaaditun kansiorakenteen ja ohjeen - mkdir -p roles/hello/tasks/ - micro roles/hello/tasks/main.yml

- copy:
    dest: /tmp/hello-ansible
    content: "See you at TeroKarvinen.com!\n"

Ajetaan uudestaan - ansible-playbook site.yml ja virheilmoitus katosi.

  <img width="601" height="433" alt="Screenshot 2026-11-03 at 16 26 59" src="https://github.com/user-attachments/assets/d8c7cd72-4d47-4671-aff1-620dd191a10f" />

Varmistin vielä lopputuloksen lukemalla luodun tiedoston SSH:n yli toisella ohjelmalla - ssh localhost cat /tmp/hello-ansible

<img width="327" height="52" alt="Screenshot 2026-11-03 at 15 35 35" src="https://github.com/user-attachments/assets/e8b75d19-3a12-464e-a759-c87258262ac3" />

## Lähteet

Karvinen, T. 2026. Hello Ansible. Luettavissa: https://terokarvinen.com/hello-ansible/. Luettu: 3.11.2026

Karvinen, T. 2026. SSH public key - Login without password. Luettavissa: https://terokarvinen.com/ssh-public-key-login-without-password/. Luettu: 3.11.2026







