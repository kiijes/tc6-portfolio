# Tekniikan portfolio
## 1. Johdanto

Tämän raportin tarkoitus on tuoda esille Ticorporate Demo Lab -kurssin aikana tekemäni asiat ja siitä saatu tekninen osaaminen. Kerron siitä, mitä toiminnallisuuksia tein projektiin, sekä avaan joitakin niistä hieman enemmän koodiesimerkkien kera.

TC-projektin tarkoituksena oli tuottaa prototyyppi mobiilisovellukselle, jonka avulla kuorma-auton kuljettaja voisi seurata kuormansidontaliinojensa kireyttä ja saada ilmoituksen, jos jokin niistä on löystynyt.

Teknisen tekemisen lisäksi valitsin itselleni sivutyöksi testaamisen, mutta loppujen lopuksi tämä sivutyö jäi omalta osalta tekemättä, pois lukien tietenkin teknisen työn ohessa pakosti tehtävä testaaminen. Avaan tätä hieman enemmän raportin loppupäässä.

## 2. Miksi Kuormaliina?

Halusin päästä tekemään TC:ssä jotakin web-teknologioilla tuotettavaa projektia luontaisena jatkumona syksyllä käydylle Web-sovelluskehitys -kurssille. Halusin kurssin aikana mm.

* syventää omaa osaamista web-teknologioiden kuten Angularin, JavaScriptin, ja NodeJS:n parissa,
* tehdä full stack -kehitystä ja olla kokonaisvaltaisesti mukana tekemässä laajaa kokonaisuutta,
* oppia ja harjoituttaa versionhallinnan hyödyllistä ja sujuvaa käyttöä laajemmassa, monijäsenisessä projektissa,
* hyödyntää joitakin pilvipalveluja ja/tai -alustoja, sekä
* saada vietyä isompi projekti maaliin asti, tuottaen lopputuloksena jotakin konkreettista ja toimivaa.

Päädyimme toteuttamaan Kuormaliina-asiakasprojektiin mobiilisovelluksen NativeScriptillä, backendin Node.js:llä ja Expressillä sekä tietokantaratkaisun MongoDB:llä. Backend toimii samalla sekä REST-rajapintana että Socket.IO -palvelimena: se vastaanottaa HTTP-kutsujen avulla dataa, jota se tallentaa tietokantaan ja tarvittaessa lähettää Socket.IO:n avulla eteenpäin mobiilisovellukselle. Tämä projekti tuki täten omia toiveitani ja oppimistavoitteitani hyvin.

## 3. Oppiminen projektin aikana

Projektin aikana suurin yksittäinen uusi asia, jonka opin, oli ehdottomasti NativeScript. Tämän lisäksi muita täysin uusia asioita, joita opin, olivat GitHub-branchin deployaaminen suoraan Herokuun, Socket.IO:n huoneiden käyttäminen, Google Firebase ja sen käyttöönotto, push-notifikaatiot Firebasen avulla sekä NativeScript-sovelluksen nimen, ikonin ja splash screenin vaihtaminen.

Näiden lisäksi syvensin tai vahvistin osaamistani Angularissa, RxJS:n Observablen käytössä, Socket.IO:ssa, REST-rajapintojen rakentamisessa Node.js:llä ja Expressillä, MongoDB:ssä, Mongoosessa, Herokussa ja MLabissa.

Läpi projektin oppimisprosessi minkä tahansa uuden asian opettelemiseksi ja soveltamiseksi omaan tarkoitukseen oli melko sama – ensin piti rakentaa ymmärrys lukemalla virallisia dokumentaatioita ja katsomalla tai lukemalla tutoriaaleja. Tämän jälkeen oli hyvä tutkia miten muut ovat soveltaneet tätä asiaa omissa projekteissaan, ja jos vain mahdollista, oli hyvä testata uuden asian toimintaa ja syy-seuraussuhteita kokeilemalla sitä itse jossain pienessä varta vasten luodussa demoprojektissa. Lopulta, kun asiaa alkoi oppia käyttämään, oli hyvä ryhtyä kokeilemaan sen integroimista omaan ratkaisuun.

## 4. Mitä tuli tehtyä

Olin koko projektin aikana isossa vastuussa teknisestä toteutuksesta. Olin ensimmäiseen arviointitilaisuuteen eli gateen asti käytännössä ainoa ohjelmoija projektissamme. Loin mobiilisovellukselle ja backendille pohjat ja työstin niistä ensimmäistä gatea varten version, jossa oli jo toteutettuna

* sovelluksen yhteys palvelimeen Socket.IO:lla,
* keräimen lisääminen kuunteluun sovelluksen sisältä,
* keräimen tietojen vastaanottaminen ja tallentaminen tietokantaan palvelimella,
* keräimen tietojen lähettäminen eteenpäin sovellukselle, jos se kuuntelee sitä, ja
* keräimen statuksen muuttaminen sovelluksessa virhetilanteen sattuessa.

![alt text](/imgs/uidia1.png "Kuvankaappaus diasta, jossa näkyy sovelluksen käyttöliittymän näkymiä.")
Kuvio 1. Kuvankaappaus diasta, jossa näkyy sovelluksen käyttöliittymän näkymiä.

![alt text](/imgs/uidia2.png "Kuvankaappaus diasta, jossa näkyy sovelluksen käyttöliittymän näkymiä.")
Kuvio 2. Kuvankaappaus diasta, jossa näkyy sovelluksen käyttöliittymän näkymiä.

Projektin vaatimukset tarkentuivat tai muuttuivat jatkuvasti, ja samalla tekniikkakin koki muutoksia siellä täällä, mutta sovelluksen perustoimintaperiaate pysyi kuitenkin suurimmilta osin samana kuin ensimmäisen gaten versiossa: keräimen tiedot liikkuvat backendistä sovellukseen Socket.IO:n avulla ja backend ottaa vastaan keräinten lähettämän datan sekä sovelluksen lähettämän kuuntelemisilmoituksen keräimelle HTTP-kutsuina.

### 4.1 Järjestelmän toiminnallisuuksia

Käydään seuraavaksi läpi joitakin toiminnallisuuksia, joita tein tähän projektiin. Koitan havainnollistaa niitä koodinpätkillä, joissa näkyy, miten haluttu toiminnallisuus on toteutettu.

#### 4.1.1 Sovelluksen yhteys palvelimeen

Mobiilisovelluksen pääasiallinen yhteys backendin kanssa muodostetaan Socket.IO:n avulla. Tämä yhteys muodostetaan mobiilisovelluksen puolella SocketioService-nimisessä palvelussa. Katsotaan aluksi kyseisen luokan konstruktoria, jossa yhteys luodaan.

`app/shared/services/socketio.service.ts`
```javascript
constructor() {
    // Connect to backend with socket.io
    this.socket = SocketIO.connect(this.apiUrl);

    // login the user to firebase and get the push registration token
    firebase.login({
        type: firebase.LoginType.ANONYMOUS
    }).then(user => {
        firebase.getCurrentPushToken().then((token: string) => {
            console.log('Current push token: ' + token);
            // set the push registration token as the client identifier
            this.clientId = token;

            // join a channel which has the name of the client ID
            this.socket.emit('join channel', this.clientId);
        });
    }).catch(error => console.log('firebase.login error: ' + error));

}
```

Socket.IO-yhteys on hyvin yksinkertainen muodostaa apukirjaston avulla. Konstruktorissa hoidetaan myös käyttäjän kirjaaminen anonyymisti Firebaseen push registration tokenin saamiseksi. Tämä toimii käyttäjän yksilöllisenä ID:nä, ja se lähetetään `join channel` eventin mukana palvelimelle.

Katsotaan seuraavaksi yhteyden muodostaminen palvelimen puolella.

`app.js`
```javascript
io.on('connection', (socket) => {
  var eventEmitter = new events.EventEmitter();
  eventEmitter.on('post', (clients, data) => {
    clients.forEach(client => {
      console.log('sending data to client: ' + client);
      console.log('typeof client is: ' + typeof client);
      io.in(client).emit('message', data);
    });
  });

  console.log('user connected!');

  socket.on('join channel', channel => {
    console.log('client joined channel: ' + channel);
    console.log('typeof channel is: ' + typeof channel);
    socket.join(channel);
    collectorController.sendToListeners(channel);
  });

  socket.on('disconnect', () => {
    console.log('user disconnected');
  });

  exports.emitter = eventEmitter;
});
```

Käyttäjän muodostaessa Socket.IO-yhteys palvelimelle luodaan myös EventEmitter, jonka avulla voidaan REST API:n kontrollerin puolelta välittää tietoa Socket.IO:lle. Saadessaan `join channel` eventin vastaan, palvelin lisää sen lähettäneen socketin parametrina annetun merkkijonon nimiseen huoneeseen. Tässä tapauksessa huoneen nimi on clientin saaman Firebase push registration tokenin arvo. Samalla huoneeseen liittyessään kutsutaan kontrollerista funktiota sendToListeners, joka lähettää huonetta kuunteleville clienteille (käytännössä siis vain yhdelle clientille) tietoja kerääjistä, jos he kuuntelevat yhtäkään niistä. Ohessa kyseinen `sendToListeners`-funktio.

`controller/collector.controller.js`
```javascript
exports.sendToListeners = (client) => {
    Collector.find({ clients: client })
        .then(docs => {
            docs.forEach(doc => {
                sendDataToClients([client], { id: doc.id, error: doc.error, belts: doc.belts }, false);
            });
        }).catch(err => {
            res.status(500).send({
                message: err.message || 'Some error occurred while sending Collectors.'
            });
        });
};
```

Collector on Mongoose-malli, jonka avulla voidaan suorittaa tietokantatoimenpiteitä, tässä tapauksessa etsiä kaikki kerääjät, joiden clients-kentästä löytyy etsitty client. Haku palauttaa taulukon kaikista Collector-dokumenteista, joka täyttää haun ehdot. Nämä dokumentit käydään yksi kerrallaan läpi ja niiden olennaiset tiedot lähetetään dokumentti kerrallaan clientille. Näin sovelluksen auetessa saadaan haettua ne keräimet, joiden kuuntelua sovellus ei ole lopettanut.

#### 4.1.2 Keräimen tietojen vastaanotto ja lähetys

Käydään läpi seuraavaksi keräimien tietojen reitti palvelimelle ja sieltä puhelimelle. Aloitetaan keräimen ja liinan Mongoose-skeemoilla, jotka olen luonut ryhmämme tutkimuksen ja asiakkaiden kanssa käytyjen keskusteluiden pohjalta. Skeemoja on päivitelty läpi projektin vastaamaan uusia vaatimuksia tai tarpeita, esimerkit ovat viimeisimmät versiot niistä.

`models/Collector.js`
```javascript
const Collectors = new Schema({
    id: {
        type: String,
        required: true
    },
    belts: {
        type: [BeltSchema],
        required: true
    },
    clients: {
        type: [String]
    },
    error: {
        type: Boolean,
        required: true
    }
});
```

`models/Belt.js`
```javascript
module.exports = new Schema({
    id: {
        type: String,
        required: true
    },
    force: {
        type: Number,
        required: true
    },
    loose: {
        type: Boolean,
        required: true
    },
    fastened: {
        type: Date,
        required: true,
        default: Date.now
    },
    loosened: {
        type: Date
    }
});
```

Pakollisia tietoja tietokantaan tallennettavaksi ovat kaikki paitsi kuuntelevat clientit ja löystymispäivämäärä. Katsotaan seuraavaksi, miten palvelin käsittelee sille tulevan kutsun.

`controller/collector.controller.js`
```javascript
// This function should save data conforming to the Collector model to MongoDB
exports.sendData = (req, res) => {
    console.log("Incoming Request:", req.body);

    // Validation that ID and a belt array is included, belt array can be empty
    if (!req.body.id || !req.body.belts || req.body.error === null) {
        return res.status(400).send({
            message: 'Request is missing ID, belt or error info'
        });
    }

    // Try to find a collector with an ID specified in request
    Collector.findOne({ id: req.body.id }, (err, collector) => {
        if (err) {
            return res.status(500).send({
                message: err
            });
        }

        // If the specified collector is not found in the database, create it
        if (!collector) {
            console.log('Collector not found with ID ' + req.body.id + ', adding new one');
            const collector = new Collector({
                id: req.body.id,
                error: req.body.belts.length > 0 ? 
                    req.body.belts.some(belt => belt.loose) : false,
                belts: req.body.belts
            });

            collector.belts.forEach(belt => {
                if (belt.loose) {
                    belt.loosened = new Date(Date.now());
                }
            });

            collector.save()
                .then(data => {
                    return res.status(200).send(data);
                }).catch(error => {
                    return res.status(500).send({
                        message: 'Some error occurred while saving Collector data: ' + error
                    });
                });
        }

        // If the specified collector is found, update its belts
        if (collector) {

            // Update the belt array of the Collector document object
            collector = updateBelts(collector, req);

            if (collector.clients.length > 0) {
                sendDataToClients(collector.clients, { id: req.body.id, error: collector.error, belts: collector.belts }, true);
            }

            collector.save()
                .then(data => {
                    return res.status(200).send(data);
                }).catch(error => {
                    return res.status(500).send({
                        message: 'Some error occurred while updating Collector data: ' + error
                    });
                });
        }

    });

};
```

Lyhyesti tiivistettynä funktio etsii tietokannasta, löytyykö kutsussa olevan ID:n omaavaa keräintä kannasta. Jos ei, se luo uuden Kerääjä-dokumentin, käyden samalla läpi onko mikään liina merkitty löysäksi ja antamalla löysälle liinalle löystymisajan. Tämän jälkeen dokumentti tallennetaan tietokantaan. Jos taasen tietokannasta löytyy olemassa oleva dokumentti kutsussa olevalle ID:lle, käydään läpi keräimen liinat `updateBelts`-funktiossa, lähetetään uudet tiedot kaikille kuunteleville clienteille, jos niitä on, ja sen jälkeen tallennetaan päivitetty dokumentti tietokantaan. Käydään seuraavaksi katsomassa, miten olemassa olevat liinat päivitetään.

`controller/collector.controller.js`
```javascript
updateBelts = (collector, req) => {
    
    // Initializing iterators for loops and a boolean to know if a match is found
    let i = 0;
    let j = 0;
    let matchFound = false;

    // We need to go through all the original belts to see if any is missing
    // from the new request
    while (i < collector.belts.length) {

        // Inner loop to go through the belts in the new request
        while (j < req.body.belts.length) {

            // If a belt is matched, we update the values,
            // remove it from the request and mark a match found in the
            // boolean variable matchFound
            if (collector.belts[i].id === req.body.belts[j].id) {
                collector.belts[i].force = req.body.belts[j].force;
                collector.belts[i].loose = req.body.belts[j].loose;
                req.body.belts.splice(j, 1);
                matchFound = true;

                // Break from the loop since a match was found for this belt
                break;
            } else {
                // Otherwise we increment our iterator by one
                ++j;
            }
        }

        // If we've gone through the whole request belts array and
        // no match was found, mark the unmatched belt in the original
        // Collector document object as loose and give it a force of 0
        if (!matchFound) {
            collector.belts[i].loose = true;
            collector.belts[i].force = 0;
            ++i;
        } else {
            // If a match is found, just change matchFound back to false and
            // increment the iterator
            ++i;
            matchFound = false;
        }
    }

    // After going through the request array, all the matched elements
    // are spliced, and only any new belts are left. Here we add the
    // remaining belts, if there are any.
    if (req.body.belts.length > 0) {
        req.body.belts.forEach(belt => collector.belts.push(belt));
    }

    // Check for any loose belts
    let looseBelts = false;
    collector.belts.forEach(belt => {
        if (belt.loose) {
            looseBelts = true;
            belt.loosened = new Date(Date.now());
            return;
        }
    });

    // If any belt is loose, update collector's error status
    if (looseBelts) {
        collector.error = true;
    } else {
        // If no belt is loose, there is no error
        collector.error = false;
    }

    // Return the Collector document object
    return collector;
}
```