# Tekniikan portfolio
Jesse Kiilamaa L4657
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

### 4.1 Järjestelmän toiminnallisuuksia tarkemmin

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

Käyttäjän muodostaessa Socket.IO-yhteys palvelimelle luodaan myös `EventEmitter`-olio, jonka avulla voidaan REST API:n kontrollerin puolelta välittää tietoa Socket.IO:lle. Saadessaan `join channel` eventin vastaan, palvelin lisää sen lähettäneen socketin parametrina annetun merkkijonon nimiseen huoneeseen. Tässä tapauksessa huoneen nimi on clientin saaman Firebase push registration tokenin arvo. Samalla huoneeseen liittyessään kutsutaan kontrollerista funktiota sendToListeners, joka lähettää huonetta kuunteleville clienteille (käytännössä siis vain yhdelle clientille) tietoja kerääjistä, jos he kuuntelevat yhtäkään niistä. Ohessa kyseinen `sendToListeners`-funktio.

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

Käydään seuraavaksi läpi keräimien tietojen reitti palvelimelle ja sieltä puhelimelle, joka on ehdottomasti projektin isotöisimpiä ja monimutkaisimpia toteutuksia. Aloitetaan keräimen ja liinan Mongoose-skeemoilla, jotka olen luonut ryhmämme tutkimuksen ja asiakkaiden kanssa käytyjen keskusteluiden pohjalta. Skeemoja on päivitelty läpi projektin vastaamaan uusia vaatimuksia tai tarpeita, esimerkit ovat viimeisimmät versiot niistä.

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

Jälleen lyhyesti tiivistettynä tämä funktio käy läpi kaikki alkuperäisessä dokumentissa olevat liinat, ja koittaa etsiä niille vastinetta kutsussa olevista liinoista. Jos sama liina löytyy kutsusta, alkuperäisessä dokumentissa olevat tiedot päivitetään ja liina poistetaan kutsun liinoista. Jos vastinetta ei löydy, on `matchFound`-muuttuja false, ja kutsusta löytymätön liina merkitään löysäksi. Tämä perustuu olettamukseen, että kaikki liinat löytyisivät aina kutsusta, jos ne olisivat kireitä, ja puuttuva liina olisi siis löystynyt tai muuten virheellinen. Kun kaikki liinat on käyty läpi, on kutsusta poistettu kaikki liinat, jotka löytyvät alkuperäisestä dokumentista. Jäljelle saattaa jäädä uusia liinoja, jotka lisätään alkuperäiseen dokumenttiin, jos niitä on. Tämän jälkeen käydään liinat vielä läpi, merkataan löysät liinat ja annetaan löystymisaika, ja merkataan kerääjän error-status joko trueksi tai falseksi. Sen jälkeen palautetaan kerääjä-dokumentti.

Katsotaan, miten data lähetetään mobiilisovelluksille, jotka kuuntelevat kerääjää.

`controller/collector.controller.js`
```javascript
sendDataToClients = (clients, data, sendPushNotification) => {
    if (app.emitter) {
        console.log('emitting event post');
        app.emitter.emit('post', clients, data);
    }

    if (sendPushNotification) {
        sendPushNotificationsToClients(clients, data);
    }
}
```

`sendData`-funktiossa kutsutaan `sendDataToClients`-funktiota, jos kerääjää kuuntelee jokin mobiilisovellus. Jos jokin socket on yhteydessä palvelimeen, on `app.emitter` olemassa ja importattu `app.js`-tiedoston puolelta. Tällä EventEmitter-oliolla lähetetään event `post`. Lisäksi kutsutaan `sendPushNotificationsToClients`-funktiota, joka arvioi, pitääkö clienteille lähettää uusista tiedoista push-ilmoitusta.

`app.js`
```javascript
eventEmitter.on('post', (clients, data) => {
    clients.forEach(client => {
      console.log('sending data to client: ' + client);
      console.log('typeof client is: ' + typeof client);
      io.in(client).emit('message', data);
    });
});
```

Ohessa pätkä koodia `app.js`:n sisältä, joka kuuntelee `post`-eventtejä ja lähettää kaikille kuunteleville clienteille uuden, käsitellyn datan eli käytännössä kerääjä-dokumentin ID:n, errorin ja liina-taulukon arvot. Tämä lähetetään Socket.IO-eventtinä `message`.

Käydään seuraavaksi mobiilisovelluksen puolella katsomassa, miten dataa otetaan siellä vastaan.

`app/shared/services/socketio.service.ts`
```javascript
getMsg() {
    let observable = Rx.Observable.create(obs => {
        this.socket.on('message', (data) => {
            console.log('socketio.service: got message from backend');
            obs.next(data);
        });
    });
    return observable;
}
```

SocketioServicen funktio `getMsg` luo ja palauttaa RxJS-kirjaston Observable-olion, joka kuuntelee Socket.IO:n `message`-eventtejä ja vie sen eteenpäin kuunteleville observereille.

`app/shared/services/collector.service.ts`
```javascript
// Array of collectors the user has connected to
private connectedCollectors: Collector[] = [];
private collectorSubject = new Rx.Subject<Collector[]>();

constructor(
    private http: HttpClient,
    private ioServ: SocketioService
) {
    /* Subscribe to the observable created in the socket.io
        service and pass the data received to pushCollector()
        function */
    this.collectorSubscription = this.ioServ.getMsg().subscribe(
        (data) => {
            this.pushCollector(data);
        }
    );
}
```

CollectorService-palvelun konstruktorissa taasen tilataan getMsg:n palauttama olio, ja siltä saatava data annetaan argumenttina kutsuttavalle `pushCollector`-funktiolle. Huomioitavaa on myös luokan muuttuja `connectedCollectors`, mikä on taulukko `Collector`-olioita. Ne on määritelty omassa `collector.ts`-tiedostossaan sovelluksen puolella, mukaillen tietoa mitä palvelimelta lähetetään. Palveluluokassa on myös `collectorSubject`-muuttuja, jonka avulla tietoa kerääjistä voidaan lähettää ja vastaanottaa muualla sovelluksessa.

Katsotaan seuraavaksi `pushCollector`-funktiota.

`app/shared/services/collector.service.ts`
```javascript
// Pushes a collector to the array of collectors
pushCollector(data) {
    // Boolean to check if a collector exists and needs to be modified
    let matchFound = false;

    // If no collectors exist, push the collector
    if (this.connectedCollectors.length === 0) {
        console.log('collector.service: no collectors yet, adding to list');
        this.connectedCollectors.push({ _id: data.id, _error: data.error, _belts: data.belts });
        this.sendCollectors();
        return;
    }

    /* If there are existing collectors, go through the array
        and if a match is found, update the belts and error status */
    this.connectedCollectors.forEach(collector => {
        if (collector._id === data.id) {
            console.log('collector.service: match found, updating belts');
            collector._error = data.error;
            collector._belts = data.belts;
            this.sendCollectors();
            matchFound = true;
        }
    });

    // If no match is found, push the collector
    if (!matchFound) {
        console.log('collector.service: no match found, adding to list');
        this.connectedCollectors.push({ _id: data.id, _error: data.error, _belts: data.belts });
        this.sendCollectors();
    }
}
```

Jos yhtään kerääjää ei ole vielä lisätty `connectedCollectors`-taulukkoon, lisätään uusi vastaanotettu kerääjä sinne. `sendCollectors`-lähettää tietoa kerääjistä eteenpäin aiemmin mainitun `collectorSubjectin` avulla. Jos taulukossa on jo olemassa kerääjiä, käydään ne läpi vastineiden varalta. Jos vastaava kerääjä löytyy taulukosta, päivitetään sen `_error`- ja `_belts`-muuttujat, ja lähetetään uutta tietoa jälleen eteenpäin. Mikäli yhtään vastinetta ei löydetä, lisätään kerääjä uutena taulukkoon, ja lähetetään uusi tieto eteenpäin.

Katsotaan seuraavaksi `sendCollectors`-funktiota.

`app/shared/services/collector.service.ts`
```javascript
/* Return an observable of a subject for others to subscribe to,
   then return the current array of collectors to all subscribers
   whenever this function is called */
sendCollectors(): Rx.Observable<Collector[]> {
    console.log('collector.service: sendCollectors() fired');
    this.collectorSubject.next(this.connectedCollectors);
    return this.collectorSubject.asObservable();
}
```

Funktiossa `collectorSubject`-olion tilaajille/kuuntelijoille lähetetään `connectedCollectors`-muuttuja eli taulukko kerääjistä ja niiden tiedoista, joita sovellus on ottanut vastaan. Lisäksi funktio palauttaa `collectorSubject`-olion Observablena, joka voidaan tilata kuunneltavaksi muualla sovelluksessa. Katsotaan seuraavaksi tapausta, jossa tätä oliota kuunnellaan `CollectorsList`-komponentissa, joka on käytännössä sovelluksen päänäkymä.

`app/components/collectors-list/collectors-list.component.ts`
```javascript
collectors: Collector[] = [];
collectorSubscription: Subscription;

constructor(
  private router: RouterExtensions,
  private collectorServ: CollectorService,
  private ngZone: NgZone
) {
  this.collectorSubscription = this.collectorServ.sendCollectors()
    .subscribe(
      (data: Collector[]) => {
        // Ajetaan ngZonessa, jotta UI päivittyy
        this.ngZone.run(() => {
          console.log('collectors-list.component: sendCollectors() sent data');
          console.log('collectors-list.component: ' + JSON.stringify(data));
          this.collectors = data;
        });
      }
    );
}
```

Komponentissa alustetaan tyhjä taulukko kerääjä-olioista muuttujaan `collectors`. Tätä taulukkomuuttujaa käytetään templaatin puolella tietojen näyttämiseen UI:ssa. Komponentin konstruktorissa kutsutaan `sendCollectors`-funktiota, ja sen palauttama Observable tilataan kuunneltavaksi. Vastaanotettava data laitetaan `collectors`-muuttujaan. Tätä pitää ajaa Angularin ngZone-palvelun avulla NgZonen sisällä, jotta UI päivittyy.

Katsotaan seuraavaksi vielä, miten tätä `collectors`-muuttujaa käytetään templaatin puolella datan näyttämiseen. Näytän ensimmäisen gaten version koodia, sillä se on puhtaasti itse kirjoittamani. Projektin toinen ohjelmoija teki myöhemmin UI:n puolella todella paljon töitä, tehden käytännössä visuaalisen ulkoasun toteutuksen täysin itse mockupien pohjalta. Joidenkin näkymien, kuten `CollectorsList`-komponentin templaatin, toteutus on teknisesti kuitenkin samanlainen kuin allekirjoittaneen tekemät alkuperäiset versiot.

`app/collectors-list/collectors-list.component.html`
```html
<StackLayout class="container">
    <ScrollView orientation="vertical">
        <FlexboxLayout class="sub-container">
            <FlexboxLayout class="collector" *ngFor="let collector of collectors">

                <FlexboxLayout class="ok" *ngIf="!collector._error">
                    <Label class="collector-id" text="KERÄÄJÄ {{ collector._id }}" textWrap="true"></Label>
                    <Label class="tight" text="LIINAT KIINNI: {{ beltTotal(collector._belts)[0] }}" textWrap="true"></Label>
                    <Button class="btn-delete" text="Poista keräin" (tap)="onRemoveCollectorTap(collector._id)"></Button>
                    <Button class="btn-info" text="Hallinta" (tap)="onManageTap(collector._id)"></Button>
                </FlexboxLayout>

                <FlexboxLayout class="error" *ngIf="collector._error">
                    <Label class="collector-id" text="KERÄÄJÄ {{ collector._id }}" textWrap="true"></Label>
                    <Label class="loose" text="LIINAT LÖYSÄLLÄ: {{ beltTotal(collector._belts)[1] }}" textWrap="true"></Label>
                    <Label class="tight" text="LIINAT KIINNI: {{ beltTotal(collector._belts)[0] }}" textWrap="true"></Label>
                    <Button class="btn-delete" text="Poista keräin" (tap)="onRemoveCollectorTap(collector._id)"></Button>
                    <Button class="btn-info" text="Hallinta" (tap)="onManageTap(collector._id)"></Button>
                </FlexboxLayout>

            </FlexboxLayout>

            <StackLayout class="collector">
                <Button class="btn-add" text="Yhdistä keräin" (tap)="onCollectorAddTap()"></Button>
                <Button class="btn-add" text="Send msg" (tap)="sendMessage()"></Button>
            </StackLayout>
        </FlexboxLayout>
    </ScrollView>
</StackLayout>
```

`FlexboxLayout`-elementit, joiden luokka on joko `ok` tai `error`, toimivat ns. kortteina, joiden sisällä näkyy tiedot keräimestä sekä painike keräimen poistamiseen tai hallintapaneeliin siirtymiseen. Myöhemmissä versioissa keräimen poistamis-toiminto on siirretty hallintapaneelin alle. `collector`-luokan elementin sisälle luodaan jokaista komponentin `collectors`-muuttujassa olevaa kerääjää varten oma `FlexboxLayout`-elementti, jonka luokka määräytyy sen mukaan, onko kerääjän `error` true vai false. Alimmaisena näkymässä on nappi, josta pääsee lisäämään uuden kerääjän. `sendMessagea` kutsuva nappi on kehitysvaiheessa testaamista varten tehty toiminto.

Näitä `ok`- tai `error`-kortteja ylemmät elementit ovat lähinnä ryhmittämistä ja näytöllä kohdistamista varten. Toteutus voisi kyllä varmasti olla hienompi ja yksinkertaisempi näiden wrapperien osalta, mutta olen suht tyytyväinen tähänkin toteutukseen.

### 4.2 Muita mainitsemisen arvoisia asioita

Firebasen saaminen käyttöön Herokussa olevassa backendissa ympäristömuuttujia käyttämällä tarvitsi hieman erikoisemman ratkaisun.

`app.js`
```javascript
var admin = require('firebase-admin');

if (!fs.existsSync('./config/serviceAccount.json')) {
  fs.writeFileSync('./config/serviceAccount.json', process.env.GOOGLE_CREDENTIALS, (err) => {
    if (err) {
      console.log(err);
    }
  });
}

admin.initializeApp({
  credential: admin.credential.cert('./config/serviceAccount.json')
});
```

NPM-paketti `firebase-admin` vaatii `serviceAccount.json`-tiedoston tietääkseen, mikä projekti on kyseessä. En halunnut lisätä tätä tiedostoa repoon, sillä se sisälsi arkaluontoista tietoa. Tämän JSON-sisällön tunkeminen Herokun ympäristömuuttujiin ei kuitenkaan riittänyt, joten minun oli luotava nopea if-lause tarkistamaan, onko tätä tiedostoa olemassa config-kansiossa, ja jos sitä ei ole olemassa, luodaan se, ja sisällöksi kirjoitetaan `GOOGLE_CREDENTIALS`-ympäristömuuttujan arvo, joka on `serviceAccount`-tiedoston sisältö. Tämän jälkeen pystyin käyttämään tätä tiedostoa Firebasen tunnistautumiseen ilman, että minun piti lisätä arkaluontoista tietoa repoon.

Katsotaan vielä, miten Firebasen avulla voidaan lähettää push-ilmoituksia palvelimelta.

`controller/collector.controller.js`
```javascript
sendPushNotificationsToClients = (clients, data) => {
    if (data.error) {
        clients.forEach(client => {
            app.admin.messaging().send({
                android: {
                    priority: 10,
                    notification: {
                        title: 'Liina on löystynyt!',
                        body: `Keräimestä ${data.id} on löystynyt yksi tai useampi liina.`,
                        sound: 'default'
                    }
                },
                token: client
            }).then((response) => {
                console.log('Succesfully sent message:', response);
            }).catch((error) => {
                console.log('Error sending message:', error);
            });
        });
    }
}
```

`Data` on käytännössä objekti, missä on keräimen ID, error ja liinat. Jos `error` on true, pitää lähettää löysistä liinoista push-ilmoitus käyttäjälle. Kerääjää kuuntelevat clientit käydään yksi kerrallaan läpi forEach-loopissa, jonka sisällä käytetään `app.js`:n puolelta importattua `firebase-admin`-moduulin `messaging`-funktiota push-ilmoituksen lähettämiseen. Android on ainoa alusta, jolle teimme käytännössä työtä ja testausta, joten käytimme Androidille spesifejä ilmoitusasetuksia. `Token` määrää kenelle viesti lähetetään, ja koska jokaisen clientin yksilöllinen ID sovelluksen puolella on Firebasen push registration token, voidaan tätä käyttää suoraan `token`-kentän arvona.

## 5. Sivutyö tai sen puute

Mainitsin raportin alussa siitä, miten sivutyöni eli testaaminen jäi hieman välistä. Ainoa selitys, jonka voin antaa, on se, että työaikani meni täysin ohjelman kehittämiseen, joten en kyennyt keskittymään testaamiseen niin paljon kuin olisin kenties halunnut. Olin kuitenkin melko pitkään mukana testaajapalavereissa ja luin opettajan vaatiman testaamista käsittelevän kirjan. Tämä ei kuitenkaan taida täyttää testaamisen sivutyön kriteerejä, mutta uskon tehneeni tekniikkaa niin paljon ja kokonaisvaltaisesti, että pelkkä tekniikan tekeminen riittää omalta osaltani. 

Lisäksi, tein loppupeleissä silti todella paljon testaamista järjestelmän parissa, sillä ohjelman kehittäminenhän vaatii aina jatkuvasti testaamista - kääntyykö koodi, ajaako se virheittä, toimivatko toiminnallisuudet niin kuin pitää, raja-arvot pitää testata, toimiiko yhteys järjestelmän eri osien välillä jne. Joten täysin toimetta en testaamisen saralla jäänyt, en vain tehnyt sitä kovin järjestelmällisesti, automatisoidusti tahi raportoinut sitä.

## 6. Parannettavaa?

Kaikesta löytyy aina parannettavaa, mutta esimerkiksi mobiilisovelluksen puolella olisin voinut kunnostautua paljon enemmän visuaalisen ilmeen toteuttamisen parissa, jota en ensimmäisen gaten jälkeen tainnut tehdä miltei lainkaan. Olin enempi keskittynyt toiminnallisuuksien toteuttamiseen ja niiden toimimiseen, kuin miltä sovellus näyttää.

Dokumentaatiota en ole tämän raportin kirjoittamisajankohtana kerinnyt kirjoittaa, jos tässä raportissa selitettyjä toiminnallisuuksia ei oteta huomioon. Kommentointia olen koittanut tehdä parhaani mukaan, mutta sitä vielä puuttuu ja kenties joidenkin kommenttien muotoja tai selityksiä voisi hieman viilailla.

## 7. Pohdinta

Ticorporate oli todella opettavainen opintojakso. Se antoi mielestäni melko hyvän käsityksen siitä, millaista projekteissa työskentely käytännössä on, ja uskon tämän antavan hyviä valmiuksia työelämää varten. Lisäksi, koska teimme asiakasprojektia, saimme konkreettista kokemusta asiakkaan kanssa työskentelystä ja kommunikaation tärkeydestä, jossa onnistuimme mielestämme hyvin. Scrumin käytänteet ja ketterä kehittäminen tulivat myös tutuiksi projektin aikana.

Oman tekemisen osalta olen tyytyväinen siihen, mitä olen saanut aikaan ja miten olen kehittynyt osaajana. Sain otettua haltuun täysin uuden teknologian, NativeScriptin, ja yhdistettyä aiempaa web-sovelluskehitysosaamistani sen kanssa työskentelemiseen onnistuneesti. Muistan hyvin sen onnistumisen tunteen, kun sain sovelluksen luomaan yhteyden palvelimeen Socket.IO:n avulla, ja kun ensimmäisen kerran näin sovelluksen etusivun päivittyvän automaattisesti lähetettyäni HTTP-kutsulla uutta tietoa palvelimelle. Se osoitti minulle, että osaan juurikin sitä, mitä halusin - kehitystyötä järjestelmän joka osa-alueella aina tietokanta- ja palvelinpuolelta client-sovellukseen.

Kiitos kaikille opettajille, labramestari Pölkille, mentori Hanhelalle, kaikille muille Ticorporatelaisille sekä tietenkin omille ryhmäläisilleni tästä puolesta vuodesta. Hauskaa oli!