- IppSec lähti videossa murtautumaan kohteeseen aloittaen komennolla

```sudo nmap -sC -sV -oA nmap/shoppy <ip>```

- Selvitin tähän väliin mitä komennon eri flagit tarkoittavat, avasin manuaalin ( ```man nmap``` ) ja hain sieltä komennon eri osia ( ```/<hakusana>``` ).
Käytin apuna myös ```nmap -help``` komentoa.

- Komennon osat selkeytettynä:
  - ```nmap```: skannaa kohdeverkon
  - ```-sC```: suorittaa scripti skannauksen käyttäen oletuslistaa
  - ```-sV```: selvittää palveluiden versiot
  - ```-oA nmap/shoppy```: vissiinkin tallentaa skannauksen tulokset kaikissa formaateissa nmap/shoppy hakemistoon.

- Skannauksesta selvisi, että kohdekoneessa on vain 2 porttia auki, joista ensimmäisessä on SSH ja toisessa HTTP, josta näkee, että palvelu ohjaa URL osoitteeseen.
- URL osoitteessa IppSec yrittää selvittää sen käyttämän frameworkin, joka parin kokeilun jälkeen löytyy aiheuttamalla 404 virheilmoitus, jonka tyylistä tunnistaa, että taustalla pyörii Node.js.
- Tämän jälkeen hän avaa kirjautumissivun, joka löytyy yksinkertaisesti lisäämällä osoitteen perään ```/login```. Kirjautumissivulla IppSec pitää auki välimiesproxya ja lähtee yrittämään sisäänkirjautumista sieppaamalla HTTP pyyntöjä ja muokkaamalla niiden sisältämää dataa.
- IppSec arvuuttelee, että palvelu käyttää NoSQL MongoDB tietokantaa ja rakentaa injektion siihen perustuen.
- Syöttämällä käyttäjätunnukseksi ```admin'||'1'='1``` hän pääsee kirjautumaan admin sivulle.
- Sivulla on käyttäjähaku, josta IppSec etsii adminin-käyttäjän ja ottaa talteen sen salasanan hashin varmuuden vuoksi.
- Tämän jälkeen hän selvittää loputkin käyttäjät, jotka saa esille syöttämällä saman ```admin'||'1'='1``` injektion hakukenttään.
- Hän tunnistaa hashin olevan md5 muotoa ja etsii netistä salasana kräkkerin, joka kääntää salasanan normaaliin muotoon.
- Sivulta ei kuitenkaan löydy tapaa päästä käsiksi muunlaiseen tietoon, joten IppSec lähtee etsimään muita mahdollisia URL osoitteita. Hän käyttää ffuf nimistä fuzzeria (lukaisin tähän väliin fuzzaamisesta, [OWASP Fuzzing](https://owasp.org/www-community/Fuzzing)), jolla hän väsytyshyökkäyksen tyylisesti etsii palvelun aliverkkotunnuksia käyttäen apunaan sanalistaa.
  - Komento testaa syöttää alkuperäisen osoitteen eteen monia eri sanoja sanalistasta ja palauttaa vastauksena ne joihin saa yhteyden, esim. ```x.esimerkki-osoite.com```, jossa x tilalle kokeillaan sanoja.
  - ```-fw 5```, eli "filter word" poistaa tulostuksesta sivut joiden sisällä sanamäärä on 5 merkkiä, eli todennäköisesti virheilmoitus.

```ffuf -u http://shoppy.htb/ -H "Host: FUZZ.shoppy.htb" -w /opt/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -fw 5```

- Tämä tuottaa tulosta ja IppSec löytää hyökkäyskohteen käyttämän kommunikaatiopalvelun, johon pääseekin kirjautumaan jo löytyneillä tunnuksilla. Yhdeltä keskutelukanavista joku viestittää toiselle jotkin muut tunnukset. Osoittautuu, että näillä pääsee kirjatumaan SSH kautta.
- Kun sisään on päästy, IppSec selvittää mitä kyseinen käyttäjä voi tehdä. Tämä onnistuu komennolla ```sudo -l```.
- Käyttäjällä on oikeus password-manageriin, mutta osoittautuu, että se vaatii kuitenkin master salasanan, jota lähdetään seuraavaksi yrittämään. IppSec yrittää "buffer overflow" menetelmää ([OWASP Buffer Overflow](https://owasp.org/www-community/vulnerabilities/Buffer_Overflow)), se ei toimi.
- Seuraavaksi kokeillaan komentoa ```strings```, joka näyttää tiedostosta kaikki tulostettavat merkkijonot. IppSec tarkentaa sitä lisäämällä perään ````-e l```, joka vaihtaa enkoodausta, jolloin kaikki "turhuudet" jää pois tulostuksesta. Tästä saa vastauksena "Sample", joka osoittautuu toimivaksi salasanaksi password-manageriin, josta samme taas tietoomme uuden salasanan, jolla pääsee kirjautumaan käyttäjälle, jolla on enemmän oikeuksia.
- Käyttäjällä on oikeuden dockeriin ja IppSec avaa kontin: ```docker run --rm -it -v /:/mnt```
  - ```--rm```: poistaa kontin jälkeenpäin
  - ```-it```: interaktiivinen terminaali
  - ```-v /:/mnt```: mount point
  - ```alpine```: docker image
  - ```/bin/sh```: kohde?
- Root saavutettu
- Lopuksi IppSac vielä näytti toisen tavan saada salasana password-manageriin, käyttämällä [Hihdraa](https://ghidra-sre.org/).
