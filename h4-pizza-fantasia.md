# h4 - Pizza Fantasia

## x) Lue ja tiivistä

Karvinen 2023: Configuration Management of Distributed Systems over Unreliable and Hostile Networks

4.12.1 Size and Complexity of Some DSLs

- Hallintatyökalujen kielet (DSL) ovat todella isoja ja monimutkaisia.
- Saltissa on 510 funktiota ja Puppetissa 113.
- Ohjeet ovat väitöskirjan pituiset. Opettelu menee tuollaisessa helposti sekaisin.

4.12.2 Use of DSL Functions in Case Configuration

- Vaikka toimintoja on satoja niin oikeissa ympäristöissä (Mozilla ja USGCB) pärjätään pienellä määrällä perustoimintoja.
- Yleisimmät olivat file, package, service ja exec. Jos käyttää exec-komentoa, idempotenssi pitää varmistaa itse.
- Käy järkeen package–file–service-malli, mitä kurssilla käytetty.

4.12.3.1 Dependencies Between Main Functions

- Perusasiat hoituvat samoilla työkaluilla package, file, service, user, group ja ajon pitää olla idempotentti.
- Lopulta kaikki palautuu vain kahteen asiaan, tiedostojen käsittelyyn "file" ja komentojen ajamiseen "exec".
- Automaatio tekee taustalla lopulta vain samoja perus asioita kuin itse käsin terminaalissa.

## a) Räpylä

Valitsin demoniksi PostgreSQL, jonka olin asentanut käsin ja johon olin tehnyt testitaulun Teron ohjeen mukaan https://terokarvinen.com/2016/postgresql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu/.

Tarkistin palvelun tilan, klusterin version ja asetustiedoston sijainnin. Palvelu oli käynnissä `active`, `enabled`, versio `17` oli tilassa `online` portissa `5432` ja asetustiedosto löytyi polusta `/etc/postgresql/17/main/postgresql.conf`.

```
systemctl status postgresql
pg_lsclusters
ls -la /etc/postgresql/*/main/postgresql.conf
```
Tässä vielä linkki h10 tehtävään, mikä todistaa tehtävän tehdyksi https://github.com/mickeveistila/Palvelinten-hallinta/blob/main/h10.md 


<img width="1042" height="886" alt="Screenshot 2026-11-24 at 16 42 29" src="https://github.com/user-attachments/assets/33769379-3c0c-463a-98ec-98f44134f524" />

## b) Automaatti

Automatisoin PostgreSQL asennuksen ja hallinnan tekemällä sille oman roolin Ansibleen. Loin ensin roolille kansion ja kopioin Debianin valmiin asetustiedoston roolin haltuun.

```
cd ~/ansible
mkdir -p roles/postgresql/tasks roles/postgresql/files roles/postgresql/handlers
cp /etc/postgresql/17/main/postgresql.conf roles/postgresql/files/
```

Seuraavaksi tein roolin tehtävät

```
micro roles/postgresql/tasks/main.yml

- apt:
    name: postgresql
    state: present

- copy:
    src: postgresql.conf
    dest: /etc/postgresql/17/main/postgresql.conf
    owner: postgres
    group: postgres
    mode: "0644"
  notify: restart postgresql

- systemd:
    name: postgresql
    state: started
    enabled: yes
```

<img width="367" height="358" alt="Screenshot 2026-11-24 at 17 50 42" src="https://github.com/user-attachments/assets/94a18997-237c-4ae9-9871-07c91199ce84" />

Sitten luodaan handleri, joka käynnistää PostgreSQL:n uudelleen silloin, kun postgresql.conf muuttuu.

```
micro roles/postgresql/handlers/main.yml

- name: restart postgresql
  systemd:
    name: postgresql
    state: restarted
```

<img width="346" height="124" alt="Screenshot 2026-11-24 at 17 57 52" src="https://github.com/user-attachments/assets/f1b5c254-4fd8-431f-9286-543daf534018" />

Kytketään postgresql rooli mukaan playbookiin ja ajetaan Ansible. Postgresql lisätään listan jatkoksi loppuun. `micro site.yml`. Lopuksi ajoin playbookin. Kaikki uuden roolin kolme tehtävää menivät läpi (ok).

<img width="204" height="229" alt="Screenshot 2026-11-24 at 18 03 19" src="https://github.com/user-attachments/assets/f57c4022-800b-4216-96a6-66e278689599" />

<img width="850" height="970" alt="Screenshot 2026-11-24 at 18 18 05" src="https://github.com/user-attachments/assets/5ff47759-9a01-4322-8193-f339851a3cf8" />

## c) Asetus

Muokkasin herralla olevaa PostgreSQL:n asetustiedostoa vaihtamalla `max_connections` oletusarvosta 100 arvoon 123. Asetus vaatii palvelun uudelleenkäynnistyksen, joten samalla tuli testattua myös rooliin tehty handleri.
Etsin control + f max_connections tekstin seasta.

```
micro roles/postgresql/files/postgresql.conf
```
Tämän jälkeen ajoin playbookin. Tulosteessa näkyi muutos kopioinnissa ,minkä jälkeen käynnistyi restart postgresql handleri, aivan kuten halusinkin.

<img width="849" height="963" alt="Screenshot 2026-11-24 at 18 55 41" src="https://github.com/user-attachments/assets/5c66c4af-2c18-4469-8285-482581d0dd6e" />

Lopuksi varmistin tietokannalta, että uusi asetus tuli käyttöön. Palvelin palautti uuden arvon 123 aiemman 100 sijaan.

<img width="301" height="141" alt="Screenshot 2026-11-24 at 19 00 04" src="https://github.com/user-attachments/assets/d07d0d14-5bd7-4ae6-9974-1c2069b9a8c3" />

## d) Paikka remonttiin

Testasin postgresql-roolin toimintaa rikkomalla asennuksen ja ajamalla Ansiblen uudestaan. Pysäytin ensin palvelun, poistin asetustiedoston ja poistin postgresql-paketin.

```
sudo systemctl stop postgresql
sudo rm /etc/postgresql/17/main/postgresql.conf
sudo apt-get purge -y postgresql

```
Tämän jälkeen tarkistin, että palvelu on oikeasti rikki.

```
ls -la /etc/postgresql/17/main/postgresql.conf
pg_lsclusters
psql -c "SHOW max_connections;"
```
<img width="613" height="151" alt="Screenshot 2026-11-24 at 20 17 11" src="https://github.com/user-attachments/assets/fef5940b-8c6d-42e1-ac75-ea29ea21dbdb" />

Seuraavaksi ajoin playbookin uudestaan `ansible-playbook -K site.yml`

Ajo meni läpi ilman virheitä, mutta tarkistuksessa pg_lsclusters näytti klusterin jääneen punaiselle tilaan down.

<img width="622" height="198" alt="Screenshot 2026-11-24 at 20 27 41" src="https://github.com/user-attachments/assets/45e64dfd-eeb4-4c6a-8c85-f2d31b3012ac" />

Syyksi selvisi, että roolissa oli käytetty yleistä palvelunimeä postgresql, vaikka Debianissa varsinainen tietokantapalvelu on postgresql@17-main. Lisäksi systemd tarvitsi daemon_reload: yes asetuksen huomatakseen palautetun asetustiedoston.
Muokkasin tiedostot

```
micro roles/postgresql/tasks/main.yml

- apt:
    name: postgresql
    state: present

- copy:
    src: postgresql.conf
    dest: /etc/postgresql/17/main/postgresql.conf
    owner: postgres
    group: postgres
    mode: "0644"
  notify: restart postgresql

- systemd:
    name: postgresql@17-main
    state: started
    daemon_reload: yes

micro roles/postgresql/handlers/main.yml

- name: restart postgresql
  systemd:
    name: postgresql@17-main
    state: restarted
    daemon_reload: yes
```

Korjauksen jälkeen rikoin asennuksen vielä kerran uudestaan ja ajoin playbookin.

ällä kertaa kaikki neljä postgresql kohtaa apt, copy, systemd ja handler menivät läpi tilassa changed.

Lopuksi tarkistin klusterin tilan, muutetun asetuksen ja aiemmin luodun taulun.

<img width="849" height="965" alt="Screenshot 2026-11-24 at 20 55 05" src="https://github.com/user-attachments/assets/43a7ff44-fd84-4c18-9d9e-a96a1534e949" />

## e) Idempotentti

Osoitin tilan idempotentiksi ajamalla playbookin vielä kerran uudestaan tekemättä mitään muutoksia.

```
ansible-playbook -K site.yml
```

Kaikki tehtävät menivät läpi vihreänä tilassa ok, handleri ei käynnistynyt ja lopputuloksena oli changed=0 sekä failed=0.

<img width="847" height="968" alt="Screenshot 2026-11-24 at 21 17 36" src="https://github.com/user-attachments/assets/62372648-03bf-49f9-85bb-34499fdd41db" />


## Lähteet

Ansible Documentation s.a. ansible.builtin.systemd module – Manage systemd units. Luettavissa: https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html. Luettu: 24.11.2026.

Karvinen, T. 2016. PostgreSQL Install and One Table Database – SQL CRUD Tutorial for Ubuntu. Luettavissa: https://terokarvinen.com/2016/postgresql-install-and-one-table-database-sql-crud-tutorial-for-ubuntu/. Luettu: 24.11.2026.

Karvinen, T. 2023. Configuration Management of Distributed Systems over Unreliable and Hostile Networks. Väitöskirja. University of Westminster. Luettavissa: https://westminsterresearch.westminster.ac.uk/item/w7vvz/configuration-management-of-distributed-systems-over-unreliable-and-hostile-networks. Luettu: 24.11.2026.










