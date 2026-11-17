# h3 - Demoni

## x) Lue ja tiivistä

Karvinen 2026: Apache installed with Ansible - quick notes

- Artikkeli näyttää package-file-service mallin. Ensin asennetaan ohjelma, sitten siirretään tai muokataan sen asetustiedostoja ja lopuksi varmistetaan palvelun käynnistyminen
- Asetustiedostoihin tehdyt muutokset tulevat voimaan vasta, kun palvelu käynnistetään uudelleen
- Ansiblessa käytetään notify-toimintoa. Kun asetustiedostoa muutetaan, notify ilmoittaa asiasta handlerille, joka hoitaa palvelun uudelleenkäynnistyksen automaattisesti
- Tämä säästää resursseja käynnistämällä Apachen uudelleen vain silloin, kun asetuksiin on oikeasti tullut jokin muutos

Ansible Community Documentation: Handlers: running operations on change

- Handlerit suorittavat tehtäviä vain silloin, kun jokin toinen tehtävä tekee muutoksen. Hyvä esimerkki on palveluiden uudelleenkäynnistys asetustiedoston muokkaamisen jälkeen
- Tehtävät kutsuvat handlereita notify avainsanalla
- Jos useampi eri tehtävä lähettäisi ilmoituksen "notify" samalle handlerille, niin Ansible suorittaa kyseisen handlerin silti vain yhden kerran koko ajon päätteeksi. Tämä estää moneen kertaan turhan käynnistelyn

ansible-doc service

- Service module hallitsee kohdekoneen daemoneita, eli sen avulla palveluita voidaan esimerkiksi käynnistää tai sammuttaa uudelleen.
- Enabled: Käynnistyykö palvelu automaattisesti aina, kun tietokone käynnistetään
- name: Kertoo palvelun nimen
- state: Määrittää tilan (started, stopped, restarted, reloaded)

- Examples

```
- name: Start service httpd, if not started
  ansible.builtin.service:
    name: httpd
    state: started
```
## y) Infra-uutiset

Valitsin artikkelin Show HN: Enroll, a tool to reverse-engineer servers into Ansible config mgmt: https://news.ycombinator.com/item?id=46449852

- Työkalu nimeltä Enroll on suunniteltu takaisinmallintamaan "reverse-engineer" olemassa olevia palvelimia Ansible-muotoon
- Se kerää tiedot koneen asennetuista paketeista, palveluista ja muokatuista asetustiedostoista
- Sitten se muuntaa tämän tilan suoraan täysin toimiviksi Ansible-rooleiksi ja playbookeiksi
- Kommenteissa todetaan tämän olevan hyvä tapa opetella Ansiblea
- Voi itse ensin säätää palvelimen  haluamakseen, sitten ajaa Enrollin ja katsoa miltä se näyttäisi Ansible koodina

## a) Apassi

Olin aiemmin asentanut jo apache2, mutta poistin kaiken siihen liittyvän, että voin aloittaa tekemää tehtävää puhtaalta pöydältä.

Päivitin ensin pakettilistat ja asensin Apache2

```
sudo apt update
sudo apt upgrade -y
sudo apt install apache2 -y
```
Sitten loin kotihakemistooni kansion verkkosivulle ja sinne uuden index.html-tiedoston. Kirjoitin tiedostoon lyhyen testitekstin  "Micken sivu".

```
mkdir publicsite
micro publicsite/index.html
```
Seuraavaksi määritin käyttöoikeuksia, jotta taustalla www-data käyttäjänä pyörivä Apache-palvelin pääsee lukemaan sivua.

```
chmod ugo+x /home/micke/
chmod ugo+x /home/micke/publicsite/
chmod ugo+r /home/micke/publicsite/index.html
```
Sitten muokkasin Apachen oletussivuston tiedostoa, jotta palvelin osaa etsiä etusivua luomastani hakemistosta järjestelmän oletuksen sijaan (/var/www/html).

```
sudo micro /etc/apache2/sites-available/000-default.conf
DocumentRoot /home/micke/publicsite/
<Directory /home/micke/publicsite/>
    Require all granted
</Directory>
```
Tarkistin asetusten syntaksin, avasin palomuurista portin 80 HTTP-liikenteelle ja käynnistin Apachen uudelleen, jotta uudet asetukset tulivat voimaan

```
sudo apache2ctl configtest
sudo ufw allow 80/tcp
sudo systemctl restart apache2
```

Testasin lopuksi vielä toimivuutta. Hain virtuaalikoneen IP-osoitteen komennolla hostname -I. Varmistin toimivuuden ensin paikallisesti komennolla curl http://localhost ja sen jälkeen isäntäkoneen Macin selaimella. Sivu ei auennut ensi yrityksellä, mutta arvelin sen korjaantuvan samalla kikalla kuin olen aiemmin tehnyt. Eli sammutan vain virtuaalikoneen ja laitan uudelleen päälle. Näin sivu toimi onnistuneesti.

## b) Moottorix

Seuraavaksi lähdin asentamaan Nginx. Kaksi palvelinta ei voi kuunnella samaa aikaan samaa porttia, joten sammutin Apachen. Tämän jälkeen asensin Nginxin.

```
sudo systemctl stop apache2
sudo apt install nginx -y
```
Määritin Nginxin näyttämään a) tehtävässä luomaani sivua muokkaamalla Nginxin oletusasetustiedostoa. Etsin tiedostosta polkua root /var/www/html; ja muutin sen kotihakemistooni. Tallensin ja poistuin.

```
sudo micro /etc/nginx/sites-available/default
root /home/micke/publicsite;
```
Tarkistin ettei tiedostoon tullut virheitä ja käynnistin Nginxin uudelleen.

```
sudo nginx -t
sudo systemctl restart nginx
```
Lopuksi testasin toimiiko se macin selaimessa. Tarkistin IP-osoitteen ja lisäsin sen hakukenttään. Sivu antoi Apache2 It works sivun. Korjasin tilanteen tyhjentämmällä välimuistin (cmd+shift+r) ja annoin osoitteen uudestaan hakukenttään. Nyt sivu aukesi onnistuneesti "Micken sivu".

## c) Automoottorix

Nyt automatisoin edellisessä tehtävässä käsin tehdyn Nginx asennuksen Ansiblella. 
Loin Nginxille oman roolin Ansible hakemistooni ja tarvittavat kansiot.

```
cd ansible
mkdir -p roles/nginx/tasks roles/nginx/files roles/nginx/handlers
```
Koska olin edellisessä tehtäväsä jo konfiguroinut Nginxin ohjaamaan liikenteen omaan publicsite kansiooni ja testannut toimivuuden, niin käytin tätä asetustiedostoa Ansiblen pohjana. Kopioin default-tiedoston suoraan Nginxin hakemistosta Ansiblen files-kansioon.
```
cp /etc/nginx/sites-available/default roles/nginx/files/
```
Loin uuden tiedoston microlla ja kirjoitin sinne ohjeet Teron ohjeista mallia ottaen.

```
micro roles/nginx/tasks/main.yml
- apt:
    name: nginx
    state: present
    update_cache: yes

- copy:
    dest: "/etc/nginx/sites-available/default"
    src: "default"
    owner: "root"
    group: "root"
    mode: "0644"
  notify: restart nginx
```

Jotta Nginx osaa ottaa uudet asetukset käyttöön kopioinnin jälkeen, loin tiedoston kuuntelemaan äsken tehtyä notifyta.

```
micro roles/nginx/handlers/main.yml
- name: restart nginx
  systemd:
    name: nginx
    state: restarted
```
Lopuksi lisäsin uuden nginx roolin Ansiblen site.yml:iin listan perään ja ajoin Ansiblen. Annoin kun sitä kysyttiin.

```
micro site.yml
ansible-playbook -K site.yml
```

Playbook ajo suoriutui onnistuneesti. Ajoin vielä varmuudeksi "curl localhost", joka palautti oikean HTML-sivun. Tämä vahvisti web-palvelimen pyörivän onnistuneesti automaation jälkeen.
Tässä viel kuvat playbook ajosta ja puusta.



<img width="307" height="544" alt="Screenshot 2026-11-17 at 17 13 57" src="https://github.com/user-attachments/assets/42a8c6c6-7f9b-4eb7-aab4-29eaa13d405a" />



<img width="824" height="863" alt="Screenshot 2026-11-17 at 17 06 18" src="https://github.com/user-attachments/assets/0b420d2a-7b70-4036-8507-36bd12d1c299" />

## Lähteet

Show HN: Enroll, a tool to reverse-engineer servers into Ansible config mgmt: https://news.ycombinator.com/item?id=46449852. Luettu: 17.11.2026

Karvinen, T. 2026. Apache installed with Ansible - quick notes. Luettavissa: https://terokarvinen.com/apache-ansible/. Luettu: 17.11.2026

Ansible Community Documentation: Handlers: running operations on change. Luettavissa: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html. Luettu 17.11.2026
