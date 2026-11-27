# h5 Gitar hero

## x) Lue ja tiivistä 

Chacon and Straub 2014: Pro Git, 2ed: 1.3 Getting Started - What is Git?

- Git ottaa jokaisessa tallennuksessa snapshotin kaikista tiedostoista, eikä tallenna pelkkiä yksittäisiä muutoksia
 
- Melkein kaikki toimii omalla koneella ilman nettiä, koska koko projektin historia on tallessa paikallisesti

- Tiedostoilla on kolme tilaa: muokattu (modified), valmis tallennettavaksi (staged) ja tallennettu (committed)

- Kolmen tilan idea selittää hyvin, että git add pitää tehdä ennen git commit komentoa

Komentojen selityksiä

- git add --all: Lisää kaikki muokatut ja poistetut tiedostot valmiiksi seuraavaa tallennusta varten (Git Cheat Sheet).
- &&: Ajaa seuraavan komennon, jos edellinen onnistui (Keane, Medium).
- git commit: Tallentaa muutokset uutena versiona omalle koneelle ja avaa editorin johon kirjoitetaan viesti (Git Cheat Sheet).
- ";": Ajaa seuraavan komennon joka tapauksessa, vaikka edellinen komento ei olisi onnistunut (Keane, Medium).
- git pull: Hakee uudet muutokset GitHubista omalle koneelle ja yhdistää ne (Git Cheat Sheet).
- &&: Ajaa viimeisen komennon vain, jos git pull onnistui (Keane, Medium).
- git push: Lähettää uudet tallennukset GitHubiin (Git Cheat Sheet).

## a) Online

Loin GitHubiin uuden julkisen varaston. Annoin sen nimeksi "sunshine" ja kuvaukseksi myös "sunshine". Lisäsin myös README.md ja GNU General Public License v3.0 lisenssin.

<img width="920" height="490" alt="Screenshot 2026-11-26 at 19 28 34" src="https://github.com/user-attachments/assets/fbe2ac37-5bab-493c-8a79-3c97d9dac2b1" />

## b) Dolly

Kloonasin edellisessä kohdassa luodun varaston GitHubista SSH-osoitteella omalle koneelle, siirryin uuteen kansioon ja muokkasin README.md tiedostoa.

```
git clone git@github.com:mickeveistila/sunshine.git
bash
cd sunshine
micro README.md
```

Lisäsin tiedostoon "muutokset näkyy!". Tämän jälkeen ajoin git add ja git commit. Kirjoitin editoriin viestiksi Update README.

<img width="1190" height="687" alt="Screenshot 2026-11-26 at 19 38 01" src="https://github.com/user-attachments/assets/d2187d4f-3729-4d10-8eab-9c23cad9a65d" />


Lisäsin muutoksen seurantaan ja tein tallennuksen. Kirjoitin aukeavaan editoriin commit-viestiksi englanniksi käskymuodossa "Update README".

```
git add .
git commit
```

Sitten hain ensin mahdolliset muutokset palvelimelta ja lisäsin sen jälkeen oman muutokseni GitHubiin.

```
git pull
git push
```
Päivitin sivun selaimessa ja varmistin, että lisätty teksti sekä uusin commit viesti näkyvät GitHubin weppiliittymässä.

<img width="396" height="363" alt="Screenshot 2026-11-26 at 19 50 31" src="https://github.com/user-attachments/assets/2f3a10a0-3bc4-48e3-ad3b-4fea75dc5c99" />

<img width="776" height="591" alt="Screenshot 2026-11-26 at 19 47 24" src="https://github.com/user-attachments/assets/d24e96a7-c12f-4df2-8c3f-ae6c6d2a4273" />

## c) Doh! 

Seuraavaksi varmistin, että olen sunshine kansiossa. Avasin README.md-tiedoston, tein sinne tyhmän muutoksen kirjoittamlla "APINA APINA APINA" ja tallensin sen tekemättä committia.
Tarkistin, että muutos näkyy muokattuna.

```
micro README.md
git status
git diff
```

<img width="684" height="594" alt="Screenshot 2026-11-26 at 19 58 35" src="https://github.com/user-attachments/assets/c3c2c949-b6d9-49cb-868b-23b54fd091a0" />


Poistin tyhmän muutoksen ja varmistin, että tiedosto palautui ennalleen ja se onnistui.

```
git reset --hard
git status
cat README.md
```

<img width="440" height="267" alt="Screenshot 2026-11-26 at 20 00 54" src="https://github.com/user-attachments/assets/1cc5a253-8603-4628-bdb3-d1880ce2ade3" />

## d) Tukki

Tarkastelin varaston lokia ja tiedostoihin tehtyjä muutoksia.

```
git log --patch
```

<img width="900" height="620" alt="Screenshot 2026-11-26 at 20 06 03" src="https://github.com/user-attachments/assets/815e19c9-cf71-4577-bc05-430004b0ff7e" />

- Pitkä merkkijono ylimmällä rivillä on tallennuksen tunniste. Tekstit HEAD, main ja origin/main kertovat, että oman koneen ja GitHubin versiot ovat samassa kohdassa

- Author näyttää muutoksen tekijän ja sähköpostiosoitteen, ja Date kertoo tarkan päivämäärän sekä kellonajan
 
- Uusin tallennus on "Update README" (26.11.2026 klo 19:45). Muutosnäkymä näyttää, että README.md tiedostoon lisättiin uusi vihreä rivi "+muutokset näkyy!"
 
- Alempi tallennus on GitHubin luoma "Initial commit" (26.11.2026 klo 19:28), jossa varastoon luotiin aloitustiedostot, kuten uusi LICENSE-tiedosto
 
- Tarkistin lopuksi nimen ja sähköpostin. Haluan pitää kyseiset nimet nyt, mutta niiden muuttaminen onnistuu `git config --global user.name`ja `git config --global user.email`komennoilla.

## e) Gitanbile

- Siirryin ansible-kansioon ja laitoin sen paikalliseen Git-versionhallintaan.

```
cd ~/ansible
git init
```

- Tallensin nykyiset Ansible-tiedostot pohjaversioksi.

```
git add .
git commit
```

Avasin hello-roolin tehtävätiedoston ja muokkasin tiedostoon kirjoitettavaa tekstiä lisäämällä "Git test". Sitten ajoin muutoksen Ansiblella. Ajo onnistui ilman virheitä.
Tarkistin vielä tiedostosta, että uusi muokattu teksti meni läpi.
```
micro roles/hello/tasks/main.yml

ansible-playbook site.yml -K

cat roles/hello/tasks/main.yml
```

<img width="538" height="187" alt="Screenshot 2026-11-26 at 20 39 52" src="https://github.com/user-attachments/assets/d14b16eb-fc59-44f2-9f5e-b2e4329c3e24" />

Tallensin muutoksen uutena versiona paikalliseen Gitiin.

```
git add .
git commit
```

- Tarkistin lopuksi lokista uusimman tallennuksen ja tiedoston muutoksen ja varmistin, ettei tallentamattomia muutoksia jäänyt.

```
git log --patch 
git status
```

<img width="587" height="333" alt="Screenshot 2026-11-26 at 20 44 43" src="https://github.com/user-attachments/assets/da7897de-be97-472e-a307-6e03f7648e0a" />

## Lähteet

Chacon, S. & Straub, B. 2014. Pro Git. 2. painos. 1.3 Getting Started - What is Git? Apress. Luettavissa: https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F. Luettu: 26.11.2026.

Git s.a. Git Cheat Sheet. Luettavissa: https://git-scm.com/cheat-sheet. Luettu: 26.11.2026.

Keane, J. 2020. Meaning of Linux command operators. Medium-blogi. Luettavissa: https://jankeane.medium.com/meaning-of-linux-command-operators-and-573a83a593be. Luettu: 26.11.2026.
