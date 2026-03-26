# Teoriakysymykset -- Azure IaC ja Bicep

Vastaa seuraaviin kysymyksiin omin sanoin. Vastausten ei tarvitse olla pitkiä -- muutama lause riittää, kunhan osoitat ymmärryksesi.

---

## Osa 1: Infrastructure as Code

### Kysymys 1: IaC:n perusidea

Selitä omin sanoin, mitä Infrastructure as Code tarkoittaa ja miksi sitä käytetään. Anna esimerkki tilanteesta, jossa IaC on hyödyllisempi kuin resurssien luominen käsin Azure Portalissa.

`
IaC tarkottaa sitä et infraa ei enää kliksutella käsin portaalissa, vaan kaikki tehdään koodilla (skriptit, template, bicep jne).
Tätä käytetään koska se vähentää virheitä ja nopeuttaa koko luonti prosessia.
Esim jos pitää luoda sama ympäristö monta kertaa (dev, test, prod), niin IaC on paljon parempi ku portaalissa klikkailu, joka menee helposti pieleen.
`

### Kysymys 2: Deklaratiivinen vs. imperatiivinen

Selitä ero deklaratiivisen ja imperatiivisen IaC-lähestymistavan välillä. Kumpaan kategoriaan Bicep kuuluu?

Deklaratiivinen = kerrot miltä lopputuloksen pitää näyttää, ja työkalu hoitaa miten se tehdään.

Imperatiivinen = kerrot askel askeleelta mitä pitää tehdä.

Bicep kuuluu deklaratiiviseen, koska siinä vaan määritellään resurssit ja Azure hoitaa loput.


### Kysymys 3: Idempotenssi

Mitä tarkoittaa, kun sanotaan, että IaC on **idempotenttia**? Miksi tämä ominaisuus on hyödyllinen?

`Idempotentti tarkottaa et jos ajat saman IaC‑skriptin monta kertaa, lopputulos pysyy samana.
Ei tuu tuplaresursseja tai mitään sotkua.
Tää on hyödyllinen koska deploymentit on turvallisia eikä mee rikki jos ajaa vahingossa uudestaan.`

### Kysymys 4: Konfiguraation ajautuminen (drift)

Selitä, mitä "configuration drift" tarkoittaa. Miten IaC auttaa estämään sitä?

`Configuration drift = kun infra alkaa “ajautua” eri suuntaan, koska joku muuttaa asetuksia käsin portaalissa.
IaC estää tätä koska koodi on totuus, ja jos joku muuttaa jotain käsin, seuraava deployment palauttaa sen oikeeks.`

---

## Osa 2: Bicep

### Kysymys 5: Bicep vs. ARM

Miksi Bicep kehitettiin ARM JSON -templatejen tilalle? Mainitse vähintään 2 etua.

`Bicep tehtiin koska ARM JSON oli:

- liian pitkä ja vaikee lukea

- virhealtis ja tosi raskas kirjoittaa

Bicep on:

- lyhyempi ja selkeempi

- parempi IntelliSense

- helpompi debugata ja ylläpitää`

### Kysymys 6: Parametrit ja `@secure()`

Miksi tietokantasalasana merkitään `@secure()`-dekoraattorilla Bicepissä? Mitä tapahtuisi ilman sitä?

`Salasana merkitään @secure() koska:

- se ei näy lokissa

- se ei näy portaalissa selkokielisenä

- se ei vuoda mihinkään outputteihin

Ilman sitä salasana vois näkyä ihan kaikkialla, mikä ois aika huono juttu.`

### Kysymys 7: Moduulit

Miksi infrastruktuurikoodi jaettiin tässä tehtävässä kolmeen erilliseen moduuliin (`acr.bicep`, `postgresql.bicep`, `appservice.bicep`) yhden ison tiedoston sijaan? Mainitse vähintään 2 syytä.

`Koodi jaettiin moduuleihin koska:

1. Selkeämpi --> yks iso tiedosto ois sekava ku spagetti

2. Uudelleenkäytettävä --> esim ACR tai App Service voi käyttää muissakin projekteissa

Helpompi ylläpitää --> muutokset ei riko koko infraa`

### Kysymys 8: `uniqueString()`

Miksi ACR:n ja PostgreSQL-palvelimen nimissä käytetään `uniqueString(resourceGroup().id)` -funktiota? Mitä tapahtuisi ilman sitä?

`
uniqueString(resourceGroup().id) varmistaa et resurssin nimi on uniikki, koska Azure ei salli kahta samaa nimeä esim PostgreSQL:lle.
Ilman sitä deployment vois failata jos nimi on jo varattu.
`

### Kysymys 9: `targetScope`

Mitä tarkoittaa `targetScope = 'subscription'` main.bicep-tiedostossa? Miksi emme käytä oletusarvoa `resourceGroup`?

`targetScope = 'subscription' tarkottaa et skripti luo resource groupin subscription‑tasolla.
Jos käytettäis resourceGroup, ei voitais luoda resource groupia, koska se pitäis olla jo olemassa.`

---

## Osa 3: Azure-resurssit

### Kysymys 10: Resource Group

Mikä on Azure Resource Groupin tarkoitus? Miksi kaikki sovelluksen resurssit kannattaa sijoittaa samaan resource groupiin?

`Resource Group on paikka mihin kaikki sovelluksen resurssit kerätään yhteen.
Sama RG helpottaa:

- hallintaa

- kustannusten seurantaa

- poistamista

- tagien käyttöä`

### Kysymys 11: Ympäristömuuttujat ja Connection String

Selitä, miten sovelluksen tietokantayhteys konfiguroidaan eri ympäristöissä:
- Miten connection string asetetaan **Docker Composessa** (lokaalissa kehityksessä)?
`environment:
  - ConnectionStrings__Default=Host=localhost;Port=5432;...

- Miten **sama** connection string asetetaan **Azure App Servicessä**?
Host=psql-todoapp-dev-n7kfspyj54mqw.postgres.da....
- Miksi sovelluksen koodi ei muutu, vaikka ympäristö vaihtuu?
Koodi ei muutu koska se lukee connection stringin konfiguraatiosta, ei kovakoodattuna.
`
### Kysymys 13: PostgreSQL Flexible Server -- Firewall

Miksi PostgreSQL-palvelimeen luodaan firewall-sääntö `AllowAzureServices` (IP-alue `0.0.0.0 - 0.0.0.0`)? Mitä tapahtuisi ilman sitä?

`AllowAzureServices sallii Azure‑sisäiset palvelut (kuten App Service) yhdistää tietokantaan.
Ilman tätä App Service ei pääsis PostgreSQL:ään ollenkaan → yhteysvirhe.`

---

## Osa 4: Deployment ja turvallisuus

### Kysymys 15: What-if

Miksi `what-if` on tärkeä vaihe ennen deploymenttia? Anna esimerkki tilanteesta, jossa what-if estäisi ongelman.

`what-if näyttää etukäteen mitä Azure aikoo muuttaa.
Esim jos vahingossa poistais koko tietokannan, what-if näyttäis sen ja voisit perua ennen ku mitään pahaa tapahtuu.`

### Kysymys 16: Tagit

`Tagit auttaa:

- kustannusten seurannassa

- omistajuuden selvittämisessä

- ympäristöjen erottelussa (dev/test/prod)

- automaatiossa

Ne on vähän niinku “tarralaput” resursseille.`

Miksi kaikkiin Azure-resursseihin lisättiin tagit (`Application`, `Environment`, `ManagedBy`)? Miten ne hyödyttävät käytännössä?

Tagit (Application, Environment, ManagedBy) lisättiin sen takia, että Azure‑ympäristö pysyy siistinä ja hallittavana, eikä mee semmoseks “mikä resu tää oli ja kuka tän loi” -kaaokseks.

Hyödyt ovat:

- Kustannusten seuranta helpompi  
Näkee heti paljonko dev, test tai prod maksaa ilman et pitää arvailla.

- Ympäristöjen erottelu  
Tagilla Environment=dev ei vahingossa poista tuotantoa, heh.

- Omistajuus selkee  
ManagedBy=Bicep tai tiimin nimi kertoo kuka vastaa resurssista.

- Automaatio toimii paremmin  
Voi esim sammuttaa kaikki Environment=test yöllä tai poistaa dev‑resut automaattisesti.

- Hallinta ja siivous nopeempaa  

### Kysymys 17: Siivous ja kustannukset

Miksi on tärkeää poistaa kehitysresurssit Azuresta kun niitä ei enää tarvita? Mikä on helpoin tapa poistaa kaikki tämän tehtävän resurssit kerralla?

`
Azure maksaa koko ajan, vaikka et käyttäis resursseja.
Siksi dev‑resurssit kannattaa poistaa kun niitä ei tarvita.

Helpoin tapa poistaa kaikki:

az group delete -n <resourceGroupName>

`

---

