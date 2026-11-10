# h2 - Voileipä

## x) Lue ja tiivistä

Karvinen 2026: Sudo without password

- Käyttäjälle sudo-oikeudet ettei tarvitse salasanaa kokoajan
- Salasanaton pääsy on pakko olla automaatiolle, muuten se pysähtyisi joka kerta kun salasanaa kysytään
- sudo visudo tarkistaa ettei ole virheitä
- Onko kaikki oikeudet ilman salasanaa yhdellä käyttäjällä tietoturvariksi?

Munroe 2006: xkcd 149: Sandwitch

- Sarjakuva sudo-komennosta
- Ensin kuvassa pyydetään ilman sudo-oikeutta jolloin pyyntöä ei huomioida
- Kun leipä pyynnön eteen laitetaan sudo, niin toinen hahmo tottelee heti

Karvinen 2026: Passwordless Sudo with Ansible

- Artikkelissa kerrotaan, kuinka salasanaton sudo-oikeus rakennetaan kohdekoneille suoraan Ansiblen koodilla
- Uuden käyttäjän luonti ja lissäminen sudoless-ryhmään
- Lisätään /etc/sudoers.d/-kansioon NOPASSWD-sääntö kyseiselle ryhmälle
- Sääntö %sudoless ALL = (ALL) NOPASSWD: ALL\n antaa koko ryhmälle salasanattoman sudo-oikeuden
- Oikeuden antaminen ryhmälle on fiksumpaa kuin yksittäiselle käyttäjälle

Ansiblen sisäänrakennettu dokumentaatio ansible-doc -kommennolla

- ansible-doc copy: Kopioi tiedostoja tai hakemistoja uuteen sijaintiin kohdekoneella
- content: Teksti, joka kirjoitetaan suoraan kohdetiedoston sisään
- dest: Ehdoton polku kohdekoneella, johon tiedosto tallennetaan
- src: Polku lähdetiedostoon omalla koneellasi, joka halutaan kopioida eteenpäin
- owner: Käyttäjänimi, josta tulee luodun tiedoston omistaja
- group: Ryhmä, josta tulee luodun tiedoston omistaja
- mode: Tiedoston käyttöoikeudet

Esimerkki:

```
- name: Copy file with owner and permissions
  ansible.builtin.copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'
```
- ansible-doc apt: Hallitsee apt-paketteja eli ohjelmien asennuksia, päivityksiä ja poistoja (Debian, Ubuntnu)
- name: Paketin nimi
- state: Paketin tila esim. absent, latest, present
- update_cache: Vastaa komentoa apt-get update, päivitettyjen pakettilistojen haku ennen toimenpiteitä

Esimerkki:

```
- name: Install apache httpd
  ansible.builtin.apt:
    name: apache2
    state: present
```

- ansible-doc file: Hallitsee tiedostoja ja niiden ominaisuuksia, kuten käyttöoikeuksia ja omistajia. Poistaa hakemistoja ja linkkejä
- path: Käsiteltävän tiedoston tai hakemiston tarkka sijainti kohdekoneella
- recurse: Kun kohteena on hakemisto, sen päälle laittaminen asettaa samat oikeudet myös kaikkiin sen sisällä oleviin alikansioihin ja tiedostoihin
- src: Lähdetiedoston polku
- state: Kohteen tyypin
- owner: Tiedoston omistaja 
- group: Ryhmän omistaja
- mode: Käyttöoikeudet

Esimerkki:

```
- name: Change file ownership, group and permissions
  ansible.builtin.file:
    path: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'
```

- ansible-doc user: Käyttäjätilijen ja niide ominaisuuksien hallinta.
- name: Käyttäjän nimi
- create_home: Määrittelee, luodaanko käyttäjälle automaattisesti oma kotihakemisto
- comment: Käyttäjän kuvaus
- groups: Ryhmiä joihin käyttäjä halutaan lisätä
- shell: komentotulkki
- state: Käyttäjän olemassa olo (present, absent)
- system: Jos tämän laittaa päälle tilistä tulee systeemikäyttäjä

Esimerkki:

```
- name: Add the user 'johnd' with a bash shell, appending the group 'admins' to the user's groups
  ansible.builtin.user:
    name: johnd
    shell: /bin/bash
    groups: admins
    append: yes
```

- ansible-doc authorized_key: Hallitsee kohdekoneen käyttäjien SSH-avaimia
- user: Käyttäjätunnus kohdekoneella, jonka authorized_keys-tiedostoa muokataan.
- key: Julkinen SSH-avain

Esimerkki:

```
- name: Set authorized key taken from file
  ansible.posix.authorized_key:
    user: charlie
    state: present
    key: "{{ lookup('file', '/home/charlie/.ssh/id_rsa.pub') }}"
```

## a) Sudoless

Loin uuden käyttäjän "antero", jolle määritin automatisoidun SSH-kirjautumisen ja oikeuden käyttää sudoa ilman salasanaa. Loin myös uuden ryhmän "sudoless" ja lisäsin anteron siihen.

```
sudo adduser antero
sudo groupadd sudoless
sudo adduser antero sudoless
```
Avasin virtuaalikoneessa toisen virtuaaliruudun (TTY2) ja otin pääkäyttäjän oikeudet varmuudeksi siltä varalta, että asetuksissa menisi jotakin pieleen. Palasin takaisin alkuperäiseen ikkunaan tämän jälkeen. Macilla näppäimet uuden ikkunan avaamiseen.

```
Control + Option + Fn + F2
sudo -i
```

Annoin sudoless-ryhmälle oikeuden suorittaa komentoja ilman salasanaa. Kirjoitin avautuvaan yhden rivin.

```
sudo visudo /etc/sudoers.d/sudoless
%sudoless ALL=(ALL) NOPASSWD: ALL
```

Kopioin oman koneeni julkisen SSH-avaimen uudelle antero käyttäjälle. Sitten kirjauduin uudella tunnuksella sisään, joka tapahtui ilman salasanaa. Nollasin sudon välimuistin, ja kokeilin testikomentoa.

```
ssh-copy-id antero@localhost
ssh antero@localhost
sudo -k
sudo echo "See you at TeroKarvinen.com"
```

## b) Antero. Tee salasanaton, automaattisesti ssh:lla kirjautuva tunnus Ansiblella

Seuraavaksi loin automaation, joka asettaa salasanattomat sudo-oikeudet ja mahdollistaa SSH-kirjautumisen julkisella avaimella anterolle. Tein tehtävää /ansible kansiossa.

Loin antero-roolille kansion

```
mkdir -p roles/antero/tasks
```

Sitten ajoin: micro roles/antero/tasks/main.yml ja määritin seuraavat asiat Teron ohjeiden mukaisesti. Oman julkisen SSH-avaimen vaihdoin kuitenkin.

<img width="363" height="307" alt="Screenshot 2026-11-10 at 16 13 28" src="https://github.com/user-attachments/assets/1b6353cd-155a-4f63-b3b9-44fbbf9df218" />

Muokkasin pääplaybookia niin, että se ajaa uuden roolin ja pyytää oikeuksia
```
- hosts: all
  become: true
  roles:
    - hello
    - antero
```

Koska salasanatonta sudoa ei koneella vielä ollut kohdekäyttäjälle asennettuna, ensimmäinen ajo piti tehdä antamalla oman käyttäjäni salasana. Ajo teki tarvittavat muutokset onnistuneesti

`ansible-playbook site.yml -K`

<img width="591" height="449" alt="Screenshot 2026-11-10 at 15 54 09" src="https://github.com/user-attachments/assets/a915ed44-2fdc-46f6-a5f7-aecf9045292a" />


Lopuksi vielä testasin salasanattoman sudon ja SSH-kirjautumisen toimivuuden käskemällä Ansiblen yhdistämään suoraan antero-käyttäjänä. Playbook meni onnistuneesti läpi  kyselemättä salasanoja.
```
ansible-playbook site.yml -u antero
```

## c) Package. Asenna kaksi pakettia ansiblella

Asensin pari pakettia seuraavaksi: Apache2 ja curl. Tein ensin kansion paketeille ja kirjoitin koodin. Sen jälkeen lisäsin uuden roolin paketit site.yml:iin.

```
mkdir -p roles/paketit/tasks
micro roles/paketit/tasks/main.yml

- apt:
    name:
      - apache2
      - curl
    state: present
    update_cache: true

micro site.yml

- hosts: all
  become: true
  roles:
    - hello
    - antero
    - paketit
```

Ajoin `ansible-playbook site.yml -u antero`. Changed ei muuttunut ja se johtui, että minulla oli jo paketit asennettuna. Testasin poistamalla curl:in `sudo apt remove curl -y` ja sen jälkeen ajamalla uudestaan
`ansible-playbook site.yml -u antero`. Changed muuttui 1 ja paketti oli onnistuneesti asennettu ansiblella.

## d) File. Monirivisen tiedoston luominen ja käyttöoikeudet Ansiblella

Ensin loin uuden tiedosto-roolin jonka jälkeen lähdin jälleen muokkaamaan tiedostoa (main.yml)

```

mkdir -p roles/tiedosto/tasks

micro roles/tiedosto/tasks/main.yml

- copy:
    dest: "/home/antero/moi.txt"
    content: |
      Tässä moi tiedosto
      Tässä rivi 2
      Tässä rivi 3
    owner: "antero"
    group: "antero"
    mode: "0600"
```
Tämän jälkeen lisäsin tiedosto-roolin site.yml:iin. Suoritin automaation onnistuneesti `ansible-playbook site.yml -u antero`.

Mitä oikeudet "0600" ovat symbolisessa muodossa? -rw-------

- Eka merkki (-) tarkoittaa tiedostoa
- Omistaja: rw- = 6. Omistaja saa lukea ja kirjoittaa tiedostoa (r,w). Suoritus oikeutta x ei ole.
- Ryhmä: --- = 0. Ryhmän muilla jäsenillä ei ole oikeuksia tiedostoon
- Muut: --- = 0. Muilla järjestelmän käyttäjillä ei ole oikeuksia tiedostoon.

## e) Jotain muuta. Debug-moduuli

Debug-moduuli tulostaa viestejä tai muuttujien arvoja suoraan Ansible-ajon aikana komentoriville. Sitä käytetään esim skriptien vianmääritykseen.

Loin uuden roolin "muuta" ja kirjoitin koodin. 

```
mkdir -p roles/muuta/tasks

micro roles/muuta/tasks/main.yml

- debug:
    msg: "E-kohta debug viesti"
```

<img width="610" height="171" alt="Screenshot 2026-11-10 at 18 14 17" src="https://github.com/user-attachments/assets/5ff14eec-e6b4-4a1d-8530-bbcd1d067a25" />

## Lähteet

Karvinen, T. 2026. Passwordless Sudo. Luettavissa: https://terokarvinen.com/passwordless-sudo/. Luettu: 10.11.2026

Karvinen, T. 2026. Passwordless Sudo with Ansible. Luettavissa: https://terokarvinen.com/passwordless-sudo-with-ansible/. Luettu: 10.11.2026

Karvinen, T. 2026. Hello Ansible. Luettavissa: https://terokarvinen.com/hello-ansible/#test-ssh-with-ansible. Luettu: 10.11.2026












