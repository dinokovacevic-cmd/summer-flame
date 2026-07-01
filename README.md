# Summer Flame — mobilna aplikacija

Ljetna loyalty putovnica za Friendly Fire igrače (1.7.–31.8.2026), po prijedlogu
projekta *Summer Flame* + Dinove izmjene.

Dvije aplikacije, jedan folder, **bez servera i baze** — sve radi u pregledniku.
Svaka je **samostalna HTML datoteka** (sva logika je unutar nje) pa radi i kad se
otvori direktno, bez ikakvog buildanja.

| Datoteka | Za koga | Što radi |
|---|---|---|
| `index.html` | **Igrači** | Putovnica (pečati, bodovi, questovi) + ljestvica. Otvaraju je na svom mobitelu. |
| `radnik.html` | **Radnici** | Odabir poslovnice + prikaz današnjeg 4-znamenkastog koda. |

---

## Kako to radi u praksi

1. **Igrač** prvi put otvori aplikaciju → upiše nadimak → odabere svoju poslovnicu
   (poredane po državama sa zastavom i oznakom CRO/BIH/AUT/SLO/NED).
2. Kad želi zabilježiti dolazak, pritisne **Zabilježi dolazak**. Otvori se polje za kod
   i poruka **„Daj radniku mobitel kako bi ti upisao današnji kod”** (Dinova izmjena —
   igrač ne zna kod, radnik ga upisuje na igračevom mobitelu).
3. **Radnik** na svom uređaju ima otvoren `radnik.html`, odabranu svoju poslovnicu i
   vidi današnji kod. Uzme igračev mobitel, upiše kod, potvrdi.
4. Pečat se aktivira **odmah**, sprema se lokalno (**nema save gumba**) i ostaje tu i
   kad igrač sljedeći put otvori aplikaciju.

### Zašto kod radi bez servera
Kod je **deterministički**: računa se iz `(poslovnica + datum + tajni ključ)`. I radnička
i igračeva aplikacija računaju istu formulu, pa se **moraju složiti** oko koda za isti dan
i istu poslovnicu — a da nikad ne komuniciraju preko mreže. Zato:
- svaka poslovnica ima svoj kod,
- kod se mijenja svaki dan u ponoć (lokalno vrijeme),
- igrač koji nije u poslovnici ne može znati kod jer ga nema odakle pročitati.

> **Ograničenje (iskreno, kako i prijedlog navodi za „kasniju fazu”):** kako nema servera,
> tajni ključ je u kodu aplikacije. Tehnički potkovan igrač može iz koda izračunati kod i
> "lažirati" pečat. Za ljetnu loyalty igru tinejdžera to je prihvatljiv rizik jer:
> (1) nagrade se ionako **preuzimaju uživo u poslovnici ponedjeljkom** (radnik vidi tko stvarno dolazi),
> (2) pečat bez servera je i tako samo prikaz. Ako ljestvica/nagrade ikad trebaju biti
> "neprobojne", kod ide na backend (radnik odobrava zahtjev kroz svoju prijavljenu app). Za sada **ne treba**.

---

## Pokretanje / hosting

Sve su statične datoteke — nije potreban build.

- **Test lokalno:** otvori folder u pregledniku (npr. dvoklik na `index.html`), ili
  `python -m http.server 8123 --directory prototip/summer-flame` pa `http://localhost:8123`.
- **Za igrače (produkcija):** postavi cijeli folder na bilo koji static hosting
  (Netlify, Vercel, GitHub Pages, Cloudflare Pages, S3…). Igračima pošalji link na
  `…/index.html`, radnicima na `…/radnik.html`.
- **Na mobitelu:** može se dodati na početni ekran („Add to Home Screen”) — ponaša se
  kao prava aplikacija (PWA, radi i offline). Za to je potreban **https** hosting.

---

## Uređivanje popisa poslovnica

Popis poslovnica je **inline** u obje datoteke — blok `const FF_COUNTRIES = [...]`
na vrhu `<script>`-a. Ako mijenjaš popis, promijeni **ISTI blok u `index.html` I u
`radnik.html`** (namjerno su samostalne pa svaka nosi svoju kopiju).

⚠️ **Ne mijenjaj `id`** poslovnice nakon što krene (jer `id` ulazi u izračun koda —
promjena `id`-a promijenila bi sve kodove). Ime, grad i redoslijed smiješ mijenjati.

### ❓ Treba tvoja potvrda (Dino)
Popis je izvučen iz `analiza/02_podaci.md` (geografski pregled arena). Nazive sam složio
kao „FF <grad>” — **provjeri službene nazive** i javi ispravke:

- **Nazivi** — jesu li svi točni? (npr. je li „FF Kvatrić”, „FF Dubrava” ok, koje je pravo ime
  za Split Joker, itd.)
- **„FF Point”** — dodan kao **Zagreb** (arena 2, `id:"point"`), po `analiza/lokacije/README.md`.
  Potvrdi grad i naziv.
- **Slovenija** — „FF Maribor” (arena 27) — potvrdi ime.
- **Izostavljeno** (mogu dodati ako trebaju): dvije neaktivne bečke arene (14, 15),
  Meksiko (arena 16 — izgleda kao anomalija).

Trenutno na popisu (20 poslovnica):
- 🇭🇷 **CRO:** Kvatrić, Dubrava, Point, Osijek, Slavonski Brod, Split, Split Joker, Zadar, Koprivnica
- 🇧🇦 **BIH:** Mepas Mall (Mostar), Zenica, Sarajevo, Banja Luka, Tuzla
- 🇦🇹 **AUT:** Millennium City (Beč), Lugner City (Beč)
- 🇸🇮 **SLO:** Ljubljana, Maribor
- 🇳🇱 **NED:** Alexandrium (Rotterdam)

---

## Što je stvarno, a što placeholder

- ✅ **Pečati, bodovi, questovi, kod, sprema­nje** — potpuno funkcionalno i po pravilima iz
  prijedloga (žuti +1, narančasti LAN 2–20, crveni niz 3–20, tjedne/mjesečne nagrade).
- ✅ **Tvoj rezultat na ljestvici** — stvaran i uživo.
- ⚠️ **Ostali igrači na ljestvici** — trenutno **demo uzorak** (nema zajedničke baze).
  Za pravu ljestvicu svih igrača po poslovnici treba backend feed. Kad ga bude, mijenja se
  samo funkcija koja puni ljestvicu u `index.html` (`seededBoard` → pravi podaci).

---

## Datoteke

```
summer-flame/
├── index.html        # igračeva aplikacija (samostalna — sva logika unutra)
├── radnik.html       # radnička mini-aplikacija (samostalna)
├── manifest.webmanifest / manifest-radnik.webmanifest   # za "Dodaj na početni ekran"
├── icons/            # ikone aplikacije (PNG + SVG)
└── README.md
```

## Napomene za razvoj

- **Samostalne datoteke:** `index.html` i `radnik.html` nemaju vanjskih skripti — sva
  logika (poslovnice, kod, pravila igre) je inline. Zato rade i kad se otvore direktno.
  Ako mijenjaš poslovnice ili pravila, uredi obje datoteke (traži `FF_COUNTRIES`,
  `FF_CAMPAIGN`, `ffWeeklyReward`…).
- Pragovi nagrada su prijedlog i lako se mijenjaju (funkcije `ffWeeklyReward` /
  `ffMonthlyReward` u `index.html`).
- Nadimci se escape-aju (nema XSS-a). Podaci igrača su lokalni na uređaju (localStorage);
  ako je preglednik u **anonimnom/private modu**, aplikacija to javi umjesto da tiho izgubi pečat.
- **iOS tipkovnica:** sheet za unos koda se automatski podigne iznad tipkovnice
  (visualViewport), pa gumb *Potvrdi kod* uvijek ostaje vidljiv.
