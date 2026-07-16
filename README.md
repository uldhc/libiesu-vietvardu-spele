# Lībiešu vietvārdu spēle 🔮

Pārlūkā spēlējama spēle, kas iepazīstina ar **lībiešu (līvõ kīel) un latviešu vietvārdu pāriem** gar Kurzemes lībiešu krastu. Spēlētājs savieno abu valodu nosaukumus; par katru pareizu pāri īstā vieta iemirdzas **reālā kartē**, un kamera eleganti tuvinās tai, tad atkal attālinās līdz pārskatam.

Karte veidota ar [Leaflet.js](https://leafletjs.com/) un CARTO tumšo karšu flīzēm (tā pati tehnoloģija/stils, kas izmantota pievienotajā atsauces projektā) — īsta kartogrāfija (krasta līnija, reljefs, ceļi, vietvārdi), nevis pašzīmēta forma.

Viena statiska lapa — gatava GitHub Pages hostēšanai. Leaflet bibliotēka iekļauta lokāli (`vendor/`), tāpēc nav atkarības no CDN; vienīgie tīkla pieprasījumi spēles laikā ir pašas kartes flīzes (CARTO serveri) un Google Fonts.

## Struktūra

```
index.html   – lapas iezīme un ekrāni (sākums / spēle / kārtas beigas / fināls); logotips un favicon iekļauti tieši HTML kā base64 attēli
style.css    – dizaina tokeni, tumšais/gaišais režīms, izkārtojums, animācijas, Leaflet kontroles un uznirstošo uzrakstu restilizācija
data.js      – 51 vietvārdu pāris ar reālām platuma/garuma koordinātām (bez pašzīmētas kartes ģeometrijas — to dara Leaflet)
app.js       – spēles loģika: kārtas, sakritību pārbaude, punkti, kartes tuvināšanas/attālināšanas ritms, uznirstošie nosaukumi
vendor/      – lokāli iekļauta Leaflet.js bibliotēka (leaflet.js + leaflet.css)
assets/      – oriģinālie logotipa PNG faili (avota references; pašā lapā tie nav vajadzīgi, jo iekļauti kā base64)
```

## Palaišana lokāli

Nepieciešams vietējais serveris (ne tikai fails pārlūkā), jo lapa ielādē vairākus resursus:

```bash
python3 -m http.server 8000
# tad atveriet http://localhost:8000
```

## Publicēšana GitHub Pages

1. Izveidojiet jaunu GitHub repozitoriju (piem. `libiesu-vietvardi`).
2. Ielādējiet **visu** šo mapju saturu repozitorija saknē, ieskaitot `vendor/` (Leaflet bibliotēka) — bez tās karte nestrādās. `assets/` nav obligāti jāielādē.
3. Repozitorija **Settings → Pages** izvēlieties **Deploy from a branch**, zaru `main` un mapi `/ (root)`.
4. Pēc pāris minūtēm spēle būs pieejama adresē `https://<lietotājvārds>.github.io/libiesu-vietvardi/`.

Nav nepieciešams build solis — viss ir tīrs HTML/CSS/JS + viena bibliotēka.

## Spēles mehānika

- **5 kārtas**, 51 vietvārdu pāris kopā, sakārtoti no **vislīdzīgākajiem** vārdu pāriem uz **vismazāk līdzīgajiem**. Līdzība aprēķināta ar `difflib.SequenceMatcher` (diakritiskās zīmes ignorētas).
- Abas vārdu kolonnas (lībiski / latviski) sajauktas **neatkarīgi**. Izvēlies pa vienam vārdam katrā kolonnā (starp tiem uzzīmējas savienojuma līnija), tad spied **"Pārbaudīt"**.
- Punkti: **+10** par pareizu pāri, **−4** par nepareizu minējumu (kopējais rezultāts nekrīt zem 0).
- Karte ir **īsta, tuvināma/attālināma un vilkāma** (Leaflet + CARTO `dark_all` flīzes), ar visiem 51 punktiem redzamiem no sākuma (blāvi zili, vēl neatklāti). Kad pāris pareizs:
  1. punkts iemirdzas (zeltains, lielāks, ar pulsāciju),
  2. **kamera eleganti tuvinās** tieši šai vietai (~1.1s lidojums),
  3. pēc īsa mirkļa **atkal eleganti attālinās** atpakaļ līdz visas kartes pārskatam (~1.3s) — skaists tuvināšanas/attālināšanās ritms ar katru atrasto pāri.
- **Nosaukumu pārblīvēšanas novēršana**: spēles laikā redzams tikai **pēdējā atrastā** punkta uzraksts (lai karte nepārblīvējas); pārējos atrastos punktus var apskatīt, uzbraucot ar peli. Kārtas/fināla kopsavilkuma kartēs (kur jāredz visi uzreiz) uzraksti rādīti pastāvīgi, un karte ir tuvināma, ja tie kādā vietā sakrīt blīvi.
- Uzrakstos vispirms rādīts **lībiskais** nosaukums, aiz tā iekavās — latviskais.
- **Laika bonuss**: kopējais spēles laiks (visas 5 kārtas) tiek pieskaitīts/atskaitīts no gala rezultāta finālā — jo ātrāk pabeidz, jo lielāks bonuss. Konstantes viegli maināmas `app.js` sākumā:
  - `TIME_TARGET_SECONDS` (mērķa laiks visai spēlei, noklusējums 330s)
  - `TIME_BONUS_PER_SECOND` (punkti par katru sekundi ātrāk/lēnāk, noklusējums 1.5)
  - `TIME_BONUS_MAX` / `TIME_BONUS_MIN` (bonusa griesti, noklusējums +180 / −120)
- **Iziešanas apstiprinājums**: uzklikšķinot uz logotipa vai virsraksta spēles/kārtas/fināla ekrānā, parādās jautājums, vai tiešām vēlies pārtraukt un atgriezties sākumā (progress tiek zaudēts). Sākuma ekrānā klikšķis nekādu jautājumu nerada.
- Katras kārtas beigās mainīgs "Vai zināji?" fakts par lībiešu valodu un kultūru; pēdējais fakts satur saiti uz LU Lībiešu institūtu (livonian.lv).

## Datu labošana / paplašināšana

Vietvārdu pāri atrodas `data.js` — katram ierakstam vienkārši `liv`, `lv`, reāls `lat`/`lon`, `sim` (līdzības procents) un `round` (1–5). Lai pievienotu jaunu vietvārdu, nav nekādas projekcijas vai SVG ceļa jārēķina — pietiek ar īstajām koordinātām (piemēram, no Wikipedia vai OpenStreetMap), un pēc izvēles pārrēķināt `round`/`sim`, ja vēlaties, lai tas iekļaujas grūtības secībā.

## Kartes avots

Kartes flīzes: `https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png` (© OpenStreetMap © CARTO, bezmaksas lietošanai ar atsauci, tāpat kā pievienotajā atsauces projektā). Ja vēlaties citu stilu (piemēram, gaišu vai topogrāfisku karti), nomainiet `TILE_URL` konstanti `app.js` sākumā — Leaflet atbalsta jebkuru XYZ flīžu servisu.

## Pieejamība un veiktspēja

- Nav build rīku; viena bibliotēka (Leaflet) iekļauta lokāli.
- Atbalsta `prefers-reduced-motion`.
- Pielāgots mobilajām ierīcēm (karte augšā, vārdu saraksts apakšā kā izvelkama josla).
- Tumšais/gaišais režīms tiek saglabāts `localStorage`; kartes flīzes abos režīmos ir vienas un tās pašas (tumšās), ar smalku spilgtuma korekciju gaišajā režīmā.

---

Veidots izglītības un lībiešu valodas un kultūras mantojuma popularizēšanas nolūkos.
