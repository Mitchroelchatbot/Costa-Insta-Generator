# Costa Weekplanning Generator — overdracht (t/m v7.15, 6 augustus 2026)

## ⚠️ Voor een nieuwe sessie — lees dit eerst
- **Werkwijze-afspraak met de eigenaar (Mitch): eerst een render/voorbeeld in de chat laten zien, pas na zijn akkoord pushen en deployen.** Uitzondering alleen als hij expliciet "push direct" zegt. Reden: elke Netlify-deploy via de MCP draait een build en kost buildtegoed; GitHub-pushes zijn gratis. Wijzigingen bundelen, één deploy per akkoord.
- **Code**: github.com/Mitchroelchatbot/Costa-Insta-Generator (`index.html` = de hele app, `docs/` = deze overdracht + collega-instructie). Pushen: HTTPS met een fine-grained token van Mitch (Contents: read/write; hij moet het token opnieuw plakken in een nieuwe sessie — remote-URL: `https://x-access-token:TOKEN@github.com/...`). SSH is geblokkeerd in de cloudomgeving; push met `git -c http.proxy= -c https.proxy=` om de sandboxproxy te omzeilen.
- **Live**: costainstacreator.netlify.app. Deployen: map met alleen `index.html`, dan Netlify-MCP `deploy-site` (siteId `ab0ceaba-eef3-49fc-bb7e-4283f8e51840`) → geeft een npx-commando → uitvoeren in die map.
- **Mappen van Mitch** (via de bestanden-brug, desktop-app moet online zijn): `C:\Users\31641\Desktop\Costa AI\06 Costa generator\` en `G:\Mijn Drive\CAFE COSTA\Ontwerpen\Benodigdheden\06 Costa generator\` — na elke goedgekeurde versie beide bijwerken. **Let op: beide mappen staan nog op v7.6** (device was offline bij v7.7 t/m v7.9.1) — bijwerken zodra de desktop-app weer online is.
- **Testen**: Playwright headless, invoer via échte input-events, canvas via toDataURL exporteren, renders visueel beoordelen (aanpak staat verderop).
- Nog open bij eigenaar: map "Ontwerpen" op G: hernoemen naar "Ontwerpen Instagram"; TV-video als loop testen op het scherm in de zaak; map "10 Fotos" vullen voor extra cutouts.


## Wat het is
Eén zelfstandig HTML-bestand (~14,7 MB) dat wekelijkse Instagram-posts (4:5), stories (9:16), TV-schermen (16:9) en video's (9 sec animatie, MP4 via Safari/iPhone) genereert voor Café Costa. **Status: live op costainstacreator.netlify.app** (deploy via de Netlify-MCP, zie bovenaan). Invoerformaat en parsing ongewijzigd sinds v2 — zie `costa-generator-instructie.md`.

## Nieuw in v4 (feedbackronde eigenaar)
1. **Lettertype: Anton → Oswald.** De originele designer-flyers bleken geen Anton te gebruiken; naast elkaar gelegd (WEEK-23/29-INSTA) is het Oswald 700 — smaller, rondere hoeken, minder zakelijk. Ingebedd als `FONT_TITLE` (Oswald 700, familie **CostaTitle**) voor titels/datums/dagletters/OPEN/chips, en `FONT_SUB` (Oswald 500, familie **CostaSub**, geladen met weight 600) voor subregels — de `BARLOW`-const wijst nu eerst naar CostaSub, Barlow blijft als fallback ingebed. Alle `ANTON`-verwijzingen heten nu `TITLEF`.
2. **Game Night-logo teruggedraaid** naar de v2-versie (pixel-letters met roze/blauwe gloed, 283×150) — de eigenaar vond het "vlakke" origineel uit map 01 minder mooi. Bewuste uitzondering op "originelen zijn beter".
3. **Hoekpalmen (flair vol) gefixt** — idee van de eigenaar: beide gespiegeld, zodat de rechte snijranden van de bronbestanden buiten beeld vallen en de toekan het beeld ín kijkt. Iets kleiner (`W*0.30`) tegen de drukte.
4. **Fotokiezer + "zonder foto".** Nieuwe dropdown **Foto**: `geen` (default) of foto 1–4 uit een ingebedde galerij (EMBED-keys `foto1`–`foto4`, gehaald uit de bestaande templates; alleen de 4 schone zonder ingebakken actie-badges). Upload blijft werken; een galerijkeuze reset een eerdere upload.
   - **`drawEmptyTop()`**: er bestaan géén originele templates zonder foto (ook de "Lege templates"-map heeft ingebakken foto's), dus de fotozone wordt afgedekt met een verloop in kleuren die uit het template zelf worden gesampled (op 0,555H), plus accentgloed en donkere palm-silhouetten. Afdekking tot **0,58H** met **alpha-uitvloei** (destination-out, band 0,09H) — géén kleurband, anders ontstaat een zichtbare naad (leergeld). Cache per template+formaat in `EMPTY_CACHE`.
   - De afdekking is de **basislaag die er altijd ligt** (bij ingebouwde templates); een gekozen/geüploade foto komt er bovenop, met een extra schaduwband (0,07H) onder de foto voor een zachte overgang. Zonder die basislaag schemert de ingebakken templatefoto onder de gekozen foto door.
5. **Animatie-effecten** (alleen tijdens animatie/video; statische PNG blijft schoon, alles achter `tSec < 900`):
   - "Deze week"-zone: **neon-opstartflikker** (stotterpatroon t<1,7 s) en daarna rustige ademhaling, als 'lighter'-radial op ~0,795H (post) / 0,838H (story).
   - **Finale**: vanaf t=7 dimt het beeld (max 0,55), vanaf t=7,9 licht het COSTA-logo op (warme gloed op ~0,908H/0,923H); video eindigt helder op het logo.

## v7.15 — PSV-blok: leesbaarheid en plaatsing
**Wedstrijdregel leesbaarder.** Was `#E30613` op een witte balk = 4,7:1 contrast, op B*0,15. Nu `#B3040F` op B*0,20 = 6,9:1 (op gekleurde balken wit). Gekozen uit drie varianten (rode pil met witte tekst / groter donkerrood / tijd als losse pil); de eigenaar koos de middelste.

**Embleem zo ver mogelijk naar rechts.** `psvEmbRight = xRt + 16 + SL*0.29 - 4`: de linkerrand van het OPEN-blok op de hoogte waar het embleem het breedst is. Schaalt mee met B, dus klopt ook bij 5–6 rijen. Het embleem wordt **ná het OPEN-blok** getekend (via `psvDraw`), zodat een klein overlapje er bovenop ligt in plaats van dat de gouden ring wordt afgesneden — daardoor kan het verder naar rechts dan wanneer het eronder zou liggen.
*Leergeld: eerst een vaste clamp op +18px gebruikt om overlap bij 5 rijen te voorkomen; die trok de door de eigenaar gekozen positie stilzwijgend terug naar links. Een grens die niet meeschaalt met B is hier altijd fout.*

**Wedstrijdregel volgt de schuinte**: rechts uitgelijnd op `xRt + SL*0.84 - 10`, baseline `y + B*0.90`. Niet verder naar rechts of lager: de afgeronde onderhoek (radius 0,16B) snijdt de balk daar al naar binnen, en dan valt de laatste letter buiten het wit.

**`badgeW` volgt de werkelijke uiterst-linkse punt** van embleem óf regel, i.p.v. het maximum van beide breedtes. Scheelt ~58px titelbreedte.

**Gemeten grens voor lange titels** (bij B=114, gevraagde titelmaat 59px): "ZWOELE ZOMER NACHTEN" (20 tekens) krijgt 31px mét PSV-blok en 43px zónder; "ZWOELE ZOMER" (12 tekens) krijgt 53px mét. De titel is dus vooral door zijn eigen lengte begrensd, niet door het PSV-blok — verdere lay-outwinst is marginaal. Advies aan de eigenaar: kortere titel of `|`-splitsing.

## v7.14 — story-achtergrond volgt het gekozen template
Klacht: bij story kleurde de achtergrond niet mee met het template (balken wel). Oorzaak: er is **één** designer-story (roze, 1080×1920) tegenover 18 posters; `IMGS.story` werd altijd gebruikt. De designer maakt geen story-versies, dus die route was dood.

Oplossing: `buildStoryBg(key, img, W, H)` bouwt de story-achtergrond op uit het **post-template van de gekozen kleur**. Bovenkant (0–58%) en onderblok (70–100%: PROGRAMMA, Deze week, logo, toekans, palmen) worden 1-op-1 overgenomen; alleen de egale gloed daartussen (58–70%) wordt uitgerekt tot de resterende hoogte. Die zone is vlak, dus het rekken is onzichtbaar, en de snijpunten zijn continu in de bron (geen sprong in de inhoud, alleen in de verticale schaal). Gecachet per template+formaat.

Gevolg: elk nieuw template heeft automatisch een bijpassende story. `IMGS.story` blijft alleen nog fallback. De PROGRAMMA-band voor story wordt vanaf de ónderkant teruggerekend, omdat het onderblok 1-op-1 is overgenomen.

Eerder geprobeerd en verworpen: de roze story omkleuren met een 'color'-blend. Werkte voor de middenzone, maar liet het onderblok (toekans, logo, Deze week) roze — of maakte de toekans oranje/groen als je het wél meenam.

Getest: 18 templates × 3 formaten, nul console-errors.

## v7.13 — PSV-invoer volgordevrij
De eigenaar schrijft `met DJ Riva Soul PSV 20:00 vs excelsior`. De oude regex zocht "vs" pál achter `psv`, dus met de tijd ertussen bleef de tegenstander in de DJ-naam staan. Nu geldt: **alles vanaf het woord `psv` tot het eind van de met-tekst is de wedstrijd**; daaruit worden tijd en tegenstander los gevist (tijd eerst weggehaald, anders leest "vs Excelsior 20:00" de tijd als deel van de clubnaam). Een tijd die vóór `psv` staat (`+ 20:00 PSV vs Fortuna`) wordt uit het staartje van het voorstuk gehaald. Clubnamen tot twee woorden.
Getest: `PSV 20:00 vs excelsior`, `PSV vs Excelsior 20:00`, `+ 20:00 PSV vs Fortuna Sittard`, `PSV vs AZ`, `PSV` — allemaal correct, DJ-naam blijft schoon.

## v7.12 — uitlijning, gestapelde DJ-naam, PSV-blok

**Dagletters echt gecentreerd.** Canvas' `textBaseline='middle'` rekent met de em-box (inclusief staartruimte), waardoor hoofdletters optisch te laag stonden. Nu `alphabetic` met baseline op `y + B/2 + cap/2`, cap = 0,72×fontgrootte.

**`//` in de DJ-naam = zelfgekozen regelovergang** (`met Mister // Costa`), naast de bestaande `**headliner**` die automatisch breekt. Beide regels krijgen dezelfde maat: de smalste bepaalt.

**Tekstbreedte volgt de schuine rechterrand.** `wAt(f) = xRt + SL*f - 20 - textX` — de balk loopt schuin, dus lager in de balk is er tot ~70px meer ruimte. Werd eerder met de smalste bovenrand gerekend. Effect: "MISTER" van 23 naar 29px zónder het logo te verkleinen (dat bleef op 285px; een eerdere poging om het logo terug te schalen naar 0,46 werd door de eigenaar afgewezen).

**PSV-blok**: embleem van 0,55B naar 0,90B en bewust over de bovenrand (`by = y - B*0.16`); wedstrijdregel als één tekst ("20:00 VS EXCELSIOR", font B*0,15) onderaan de balk op `y + B*0.93`, dus los van de hoogte van de DJ-naam. `badgeW` wordt gemeten (max van embleembreedte en regelbreedte) i.p.v. vast gereserveerd — een korte club laat de titel meer ruimte. Aftraptijd komt uit een los tijdstip in de 'met'-tekst.

**PROGRAMMA-detectie vervangen door een opgemeten tabel** (`PROG_BAND`). De runtime-detectie zocht groene tekst, maar dat woord heeft per template een andere kleur (paars = geel, groen = oranje); kleur-onafhankelijk maken maakte het juist slechter, want de gloed matcht dan ook. Alle 18 templates + story zijn één keer offline opgemeten via randdichtheid. Onbekend template = geen afdekking (veilige default).

**Teruggedraaid: titels op alle rijen even groot.** Was gebouwd (kleinste rij bepaalt de maat) maar de eigenaar vond het slechter: één lange titel maakte de hele week kleiner. Korte titels benutten hun ruimte weer.

## v7.11 — leesbaarheidsronde naar de designerposter (WEEK 5-8, 5 rijen)
Aanleiding: de eigenaar leverde een nieuwe designerflyer met 5 rijen. Alles opgemeten.

**Balken groter.** Gemeten op de referentie: bannerhoogte 91, rijafstand 120, band 31,3%–73,6%. Onze band stond op 36–70%. Nu 31–74% → bij 4 rijen B=110, bij 5 rijen B≈88. `B/pitch` blijft 0,76 (klopte al exact).

**Bolle hoeken.** `roundPoly(ctx, pts, r)` — polygoon met afgeronde hoeken via arcTo, radius per hoek geklemd op de helft van de kortste aanliggende zijde (zo blijven pijlpunt en inkeping herkenbaar). Toegepast op dagblok, banner, flair-clip, OPEN-blok en het gele label. Radius 0,16×B, gemeten op de referentie.

**PROGRAMMA afdekken.** De band kon niet voorbij 70% omdat "PROGRAMMA" in de template-afbeeldingen zit (nt_roze vanaf 71,0%). `findProgBand()` zoekt de groene tekst per template op een verkleinde kopie (63–80% hoogte, alleen als de band < 7,5% hoog is, dus geen palmblad) en cachet dat. `coverStrip()` dekt af door de bronrij bóven en ónder de strook over de hele strook uit te rekken en in elkaar over te vloeien (plus uitvloeiende randen) — een vlakke kleurband zou zichtbaar zijn op de gloed. Alleen als de onderste balk er daadwerkelijk op valt; bij 2–3 rijen blijft PROGRAMMA staan.

**Dagblok compacter** (330→268, punt 358→296): levert 62px extra titelbreedte. Nodig omdat lange titels breedte-begrensd zijn — het lettertype opschroeven had geen effect (gemeten: vraagt 57px, krijgt 33px).

**DJ-naam.** `**naam**` = headliner: groter en desnoods over twee regels (maat geklemd op B*0,34, want dit script-font heeft stokken en staarten die anders op de regel eronder vallen). Gewone DJ-namen: vaste maat `min(B*0,23, titel*0,80)` — dus op élke rij even groot. Stond eerst op titelgrootte×factor, waardoor een rij mét badge (PSV) een kleinere titel én dus een kleinere DJ-naam kreeg; vrijdag en zaterdag verschilden daardoor zichtbaar.

**OPEN-blok.** Tekst zat tegen de randen: tijd B*0,45→0,40, label B*0,165→0,15, en elke regel wordt gecentreerd op de breedte op ZIJN eigen hoogte (`cxAt(f)`) — het blok is schuin, dus de onderste regel schoof anders tegen de linkerrand. Blokbreedte volgt nu de breedste van beide regels.

**Logo's.** Game Nights opnieuw uitgesneden zónder donkere contour (die was voor de witte balk; de balk is nu paars #662384 zoals op de designerposter) en op `logoScale:1.15`. Stelz'n op `logoScale:1.30` — mag bewust over de balk heen vallen.

**PSV.** Herkenning kijkt nu ook in de 'met'-tekst (zo schrijft de eigenaar het). Een los tijdstip daar wordt de aftraptijd: embleem + "VS CLUB" + "AFTRAP 20:00" klein eronder, alles binnen de balk.

**Actie-label** wijkt naar onder de balk zodra de rij een headliner over twee regels heeft.

Getest headless op 4, 5 en 6 rijen, met/zonder logo, PSV met en zonder aftraptijd. Nul console-errors.

## v7.9.1 — twee correcties op v7.9
- **TV + volledige foto: OPEN-blok liep 143px de foto in.** `xRtBase = Math.min(850, W - 40 - SL - oWShared - 16)` leverde op TV (W=1920) altijd 850 op, terwijl het fotopaneel al bij `W - round(W*0.48)` = 998 begint en het blok inclusief schuinte tot 1141 doorliep. (Ook vóór v7.9 was er al 38px overlap; die formule maakte het zichtbaar erger.) Nu bepaalt een `rightLimit` de rechtergrens: bij een rechthoekige foto op TV `W - round(W*0.48) - 12`, anders `W - 40`. Cutouts zijn transparant en staan gecentreerd op 0,73W, dus die houden de brede balken (xRt 850); alleen bij een volledige foto krimpen ze naar ~695.
- **Titelgrootte-schuif deed niets boven ~105% op rijen mét eventlogo**: `Math.min(B*0.38*tScale, B*0.40)` kapte de schaal af. De cap is weg; `fitFont` begrenst al op breedte.

## v7.9 — leesbaarheid: dagblok, titelfont, tekst- en labelregelaars
Aanleiding: de eigenaar legde een gegenereerde week naast een originele designerflyer (WEEK31) en
vond het origineel duidelijk beter leesbaar. Alles hieronder is opgemeten, niet op gevoel bepaald.

**Dagblok (grootste winst).** De dagletters en de datum stonden los op de achtergrond: een 6px
contour in `T.accent` met `rgba(0,0,0,0.25)` als vulling. Daardoor hing het contrast volledig af van
het gekozen template. Gemeten over alle 18 templates lag de slechtste op 3,41:1 en de default
`nt_roze` op 3,90:1. Nu staat er een massief accentblok (x 60→330 met pijlpunt naar 358, 14px
slash-gap voor de inkeping van de banner) met datum en dagletters in `T.openInk` — dezelfde inkt die
al op het OPEN-blok stond. Slechtste template nu 4,52:1, en de letters zijn massief in plaats van
contour, dus de waargenomen streekdikte gaat ~4x omhoog. Dagletters van `B*1.5` outline naar
`B*1.30` massief (bij 1,45 steken ze boven het blok uit).

**`openInk` van `stelzp` en `moon`** van `#FFFFFF` naar `#14082A`: die twee stonden wit op een
lichtpaars accent en werden na bovenstaande wijziging de slechtste van het stel (4,23 en 3,61) —
nu 4,52 en 5,30.

**Staffeling uit.** `stagger` op nullen. In WEEK31 lijnen alle balken rechts uit binnen 8px
(gemeten 815/811/807); de staffel liet ook de OPEN-blokken meeschuiven omdat die aan `xRt` hangen.
Oude waarden staan als comment in de code.

**Flair-alpha omgedraaid.** `base` was 0,5 op gekleurde balken en 1 op witte. Precies verkeerd om:
de witte balken dragen zwarte tekst en kregen de meeste ruis. Nu 1 op gekleurd, 0,45 op wit.

**Titelfont terug naar Oswald 700.** `TITLEF` wijst naar familie `CostaDay` (al ingebed voor
dagletters/OPEN). De v4-conclusie klopte: de designerflyers zijn Oswald 700. Lilita One uit v7 was
ronder en breder en las kleiner. Titel `B*0.37 → B*0.46`, subregel `B*0.24 → B*0.30`. Opgemeten
kapitaalhoogte designer 0,31×B tegen 0,245×B in de generator — dat verschil is hiermee weg.

**Balkbreedte afgeleid i.p.v. hardgecodeerd.** `oW` wordt nu één keer voor alle rijen bepaald
(alle OPEN-blokken exact even breed, dus uitlijning aan beide kanten), en
`xRtBase = Math.min(850, W - 40 - SL - oWShared - 16)` houdt een vaste rechtermarge van 40px.
Effect: 2 rijen 790 (ongewijzigd), 4 rijen 823, 5 rijen 836 — titelbreedte van 375 naar 408/421px.
Bewust niet hardgecodeerd op 850: bij 1–3 rijen is `B` groot, dus `SL` ook, en dan schiet het
OPEN-blok van het doek.

**Verticale uitlijning van het tekstblok.** De titel stond vast op `y + B*0.44`; bij een grote titel
raakte hij de bovenrand (gemeten 0px marge bij 120%). `drawMixedTitle` heeft een `measureOnly`-stand
gekregen die `{s, cap}` teruggeeft; de rij meet titel + subregel, centreert het blok en houdt
minimaal 11% van `B` vrij boven en onder. Past het niet, dan schalen beide regels evenredig terug.
Bovenmarge nu 8–9px over het hele schuifbereik 100–130%.

**Tekstschuifjes** (`#titleSize`, `#subSize`, `#textX`, `#textY`): titel- en DJ-naamgrootte
70–130%, tekst −40..+80px horizontaal en ±18px verticaal. Let op: bij een lange titel is de
*breedte* de grens, niet de basisgrootte — `drawMixedTitle` krimpt tot het past, dus omhoog schuiven
doet daar niets. Bij de normale korte titels ("Bangers Only", "Neon Party") werkt het hele bereik.

**Actie-label altijd volledig zichtbaar.** Het gele label werd midden in de rijenlus getekend en kon
dus onder een volgend dagblok of eventlogo verdwijnen. Het gaat nu via `chipQueue` naar een laatste
tekenlaag na de rijenloop, met een harde clamp op de halve breedte inclusief schuinte en rotatie, en
eigen schuifjes `#dealZoom` (60–180%), `#dealX` (−260..+120) en `#dealY` (−70..+40). De maximale
labelbreedte volgt nu de balk (`Math.min(430, xRt - 400)`) in plaats van een vaste 260, dus lange
acties krimpen niet meer naar 13px.

**Teruggedraaid: `bandBottom` van 70 naar 76.** Leek logisch (hogere balken), maar de rijen liepen
dan over de "PROGRAMMA" die in de template-afbeeldingen is ingebakken. Naar boven zit de
Stelz-actie in de weg. De 36/70-defaults zijn op het artwork afgesteld — laat staan. Rijen groter
maken kan alleen als de band per template uit het artwork wordt afgeleid; dat is een apart project.

Getest: Playwright headless op 2, 4 en 5 rijen, met en zonder eventlogo, met PSV-bijvermelding, met
de langste actietekst, en op de uitersten van alle schuifjes. Nul console-errors.

## v7.7 — PSV-bijvermelding compact
- Syntax voor collega's: `psv vs Ajax` (of `psv-Ajax`, of alleen `psv`) ergens in de titel → klein embleem (B*0,55) rechts met "VS AJAX" (rood, Oswald B*0,185) eronder; `psv` verdwijnt uit de titel zodat die de volle breedte houdt. Echte wedstrijdtitels (beginnen met PSV, of "vs"/"wedstrijd") houden het grote embleem + standaardactie. Velden: `ev.psvSide`, `ev.psvVs`; breedtereservering via `badgeScaleF`.

## v7.6 — zes verbeteringen (goedgekeurd via voorbeeld-eerst-werkwijze)
**Werkwijze-afspraak eigenaar: eerst renders in de chat laten zien, pas na akkoord pushen/deployen (Netlify-builds kosten buildtegoed; GitHub-pushes zijn gratis).**
- **Game Nights-logo scherp**: nieuwe uitsnede uit `game nights logo met text en kleur (1).png` (1080px) mét volledige "CAFE COSTA'S"-kopregel (crop 70,246–1010,712 — kopregel meenemen loste het afsnijprobleem definitief op), feather 16px, donkere contour (GaussianBlur 6, offset 3,3) voor contrast op witte balken. Per-logo schaal: `logoScale:0.85` op de game-entry (`lh *= st.logoScale||1`).
- **PSV overal**: `\bpsv\b` in elke titel geeft embleem + rode highlight; `ev.noDefaultDeal` onderdrukt de standaard pitcher-actie tenzij de titel met PSV begint of "vs"/"wedstrijd" bevat.
- **Sticker-zoom** tot 400%.
- **TV-achtergrond kleurt mee met het template**: basisgradient + gloed + palmsilhouetten allemaal afgeleid van T.accent/T.accent2 (donkere tinten via `dk(f)`), derde gloed rechtsonder.
- **Costa-details in 5 standen**: uit/subtiel/medium/vol/extra (lvl 0–4); grootte en dekking schalen lineair, lvl≥2 = extra blad + bloem, lvl≥3 = sterren + hoekpalmen, lvl≥4 = nóg meer + hoekpalmen 15% groter.
- **Sticky preview** robuuster (align-self:start) — headless gemeten en correct; klachten kwamen vermoedelijk uit browsercache.

## v7.4 — templatelijst opgeschoond
- **Dropdown plat en op kleur gesorteerd** (Roze → Magenta → Oranje → Paars → Blauw → Groen), namen minimaal: kleur, of kleur + actie/maan/WK. De oude optgroups ("met ingebakken foto") klopten niet meer: sinds drawEmptyTop tonen de klassieke templates geen foto meer.
- **'roze' (klassiek) volledig verwijderd** (option + EMBED + THEMES; −0,4 MB): eigenaar vond hem lelijk — het waren twee gecombineerde designs. `nt_roze` blijft de default.

## v7.3 — TV-verfijning (rustige loop-beweging, PROGRAMMA, OPEN-breedte)
- **OPEN-blok groeit naar rechts** als de tijd niet past (oW = max(136, tijdbreedte+30)); lettertype wordt per eis van de eigenaar nooit verkleind. WK-badge schuift automatisch mee (hangt aan oX+oW+SL).
- **TV**: "PROGRAMMA" (Lilita, #90EA3C, zwarte schaduw) gecentreerd in het lege vak ónder de balken (linkerkolom, x=570): positie/formaat dynamisch tussen rowsEnd en MORE INFO (font ≤ H*0,085 en ≤ 62% van de vrije hoogte); verschijnt alleen als er ≥60px vrij is.
- **Rustige continue beweging op TV** (alleen fmt==='tv', alle frequenties zijn hele golven per 9s-loop dus naadloos): rijen deinen ±3px (2 golven, fase i*0,9), foto 'ademt' ±2,2% (1 golf, cutout onderkant-verankerd; paneel schaalt om z'n middelpunt), sticker pulseert ±5% (3 golven). Post/story bewegen niet extra.

## v7.2 — pijl verwijderd (keuze eigenaar)
- Na twee pijl-iteraties (gebogen open vorm) besloot de eigenaar: **helemaal geen pijl** — de pijlvormige **inkeping in de balk blijft** als vormaccent (variant A). De banner- en flairclip-paden zijn ongewijzigd.
- **Dagletters raken de balk nooit meer**: breedte wordt gemeten en het font schaalt in stappen van 4px terug tot de rechterrand ≤ 338 blijft (relevant bij ≤3 rijen, waar B en dus de letters het grootst zijn).

## v7.1 — dagletters terug, OPEN-blok E, gebogen pijl (keuzes eigenaar)
- **Dagletters + datums terug naar Oswald 700** als aparte familie `CostaDay` (const `DAYF`); Lilita bleef voor titels/chips/badge/MORE INFO. Les: fontwissels nooit generiek doorvoeren — de eigenaar wilde alleen de títels anders.
- **OPEN-blok, indeling E**: tijd groot (B*0,45) met "OPEN VANAF" klein (B*0,165) eronder, beide in Oswald. Gekozen uit 5 mock-indelingen.
- **Pijl**: massieve driehoek → open contour met gebogen achterkant (quadratische curve, buik B*0,40, lijndikte B*0,062, lineJoin round) die meebuigt met de dagletter. Gekozen uit 5 pijlvarianten + 2 interpretatierondes.

## v7 — fonts (keuze eigenaar, 5 rondes) + schuifjes
- **Titelfont: Oswald → Lilita One** (`FONT_TITLE`/familie CostaTitle vervangen; alle TITLEF-plekken schakelen automatisch mee: titels, dagletters, datums, OPEN, chips, MORE INFO). Keuzeproces: 5 vergelijkingsrondes met de eigenaar; Oswald bleef het dichtst bij de originele flyers maar hij wilde ronder/vriendelijker.
- **DJ-naam: dik en donker** — Shrikhand (`CostaDJ1`) en Bangers (`CostaDJ2`), dropdown `#djfont`: afwisselend per rij (default) of vast één. Host wordt in parseSchedule-uitvoer als `*naam*` gemarkeerd; `drawMixedTitle` kreeg een `scriptFont`-parameter (script-delen 1,35× formaat). Kleur `djInk`: #2B2B2B op witte balken, #FFF8EE op gekleurde. Titel-sierletters (vrijdag/zaterdag) blijven Yellowtail.
- **Schuifjes**: foto links/rechts + omhoog/omlaag + zoom (`#photoX/#photoY/#photoZoom`), zelfde drietal voor de sticker. `coverDraw` kreeg oy- en zoom-parameters (zoom geklemd op ≥1 voor cover-foto's; cutouts zoomen vrij 50–200%). TV-fotopaneel-cache-key bevat alle drie de waarden.

## v6.1/v6.2 — Happy Thursday + sticker-dropdown
- **Happy Thursday** herkend (`ev_happy`, oranje balk `#FF9E1B→#C85A00`, hideLeftover, geen defaultDeal — donderdagactie wisselt, dus via haakjes). Ook typo "thruday" matcht.
- **Sticker-dropdown** (`#sticker`): 8 kant-en-klare actie-stickers (EMBED `st_*`, uit map "03 Acties kant-en-klaar" + de Acties_transparant-zip) én alle eventlogo's als los element. Getekend NÁ de rijen (ligt er dus overheen, zoals in het WEEK31-voorbeeld); post/story linksboven (185, 0,155H, h 0,165H), TV op (0,505W, 0,44H, h 0,30H) over de rij-uiteinden. Pop-in bij 0,9s. Sticker heeft voorrang op de tekstbadge.
- `hideLeftover` ook op XXL Saturday (logo bevat de volledige naam).
- Netlify-deploy kan nu rechtstreeks via de Netlify-MCP (deploy-site op site-id ab0ceaba-…): map met alleen index.html klaarzetten, npx-commando uit de tool-respons draaien.

## Nieuw in v6 — TV-formaat + beweegbare foto (voorbeeld: WEEK31TV_SCREEN.mp4)
1. **TV-formaat (1920×1080)**, opgemeten uit het aangeleverde voorbeeld: rijen links in een vaste zone (0,07H–0,90H, pitch-cap 180; de uitlijn-schuiven gelden alleen voor post/story), foto rechts als paneel, "Deze week" (Yellowtail 150px, neongloed in accentkleur) + wit getint CAFÉ COSTA-wordmark rechtsonder op de foto, "MORE INFO: CAFECOSTA.NL" linksonder met typemachine-effect. Badge staat op TV rechts naast de rijen (W*0,55, H*0,42).
   - **Achtergrond**: `drawTvBg()` — donkere tropische basis + gloed in themakleuren + palm-silhouetten + full-colour hoekpalmen; gecachet per thema. Eigen template-upload (liggend) vervangt de achtergrond.
   - **Volledige foto's** op TV: rechterpaneel (48% breed) met zachte linkerrand (destination-out, 180px), single-entry cache (`PHOTO_TV`) — bewust géén map-cache, anders loopt het geheugen vol bij slepen aan de positie-schuif.
   - **Loop-vriendelijk**: op TV geen zoom op de achtergrond en géén dim/oplicht-finale; de video eindigt in de eindstand zodat hij naadloos loopt.
   - **Wordmark-valkuil**: `cafe_costa_logo_transparant.png` (Huisstijl kit) is nep-transparant (ingebakken checkerboard). Gebruikt is `cafe costa logo zwart.png` uit map 01 (écht transparant), wit getint via drawAsset. EMBED-key: `costa_logo`.
2. **Foto-positieschuif** (`#photoX`, −100..100): verschuift cutouts (±0,25W post/story, ±0,12W TV) en de uitsnede van volledige foto's (via nieuwe `ox`-parameter op `coverDraw`).
3. **Foto-entree in de animatie**: foto's faden + schuiven rustig binnen (start 0,35s, duur 0,9s) in plaats van hard aanwezig op frame 1. Conform het TV-voorbeeld: beweging zit in entrees (rijen, foto, teksten, typemachine), daarna is het beeld rustig.

## Nieuw in v5 (samenvatting)
### v5 — aangeleverde fotoloze templates + uitgeknipte mensen
De eigenaar leverde twee zips aan (`Cafe_Costa_templates 1.zip`, `Cafe_Costa_cutouts_batch1.zip`):

1. **7 échte fotoloze templates** (1122×1402 → geschaald naar 1080×1350, jpeg q86): EMBED-keys `nt_roze`, `nt_oranje`, `nt_stelz`, `nt_stelzb`, `nt_wk`, `nt_maan`, `nt_or10`. In THEMES gemarkeerd met **`clean:true`**: geen `drawEmptyTop`-afdekking en geen extra hoekpalmen (staan er al ingebakken in, incl. toekan). Bovenaan in de template-dropdown (optgroup "Nieuw — zonder foto"); **`nt_roze` is de nieuwe default**. Let op: story-formaat gebruikt nog altijd het oude story-template, dus daar blijft de afdekking actief ongeacht het gekozen thema.
2. **11 uitgeknipte mensen** (transparant, zacht uitlopende randen): EMBED-keys `cut01`–`cut11` (WebP q86, alpha-bbox-crop met drempel 36, hoogte ≤880). Eigen tekenpad in render(): géén rechthoek/fade, maar los bovenop de gloed, gecentreerd, **onderkant verankerd op 0,40H**, hoogte 0,335H (breedte geklemd op 0,86W). Optgroep "Uitgeknipte mensen (aanbevolen)" in de fotokiezer; de 4 rechthoekige foto's uit v4 blijven bestaan als "Volledige foto's".
3. De zip bevat ook 4 transparante actie-overlays (`Acties_transparant/`) — nog niet ingebouwd, zie openstaand.

Bestandsgrootte 8,4 → 14 MB (bewuste keuze: kwaliteit boven laadtijd, privégebruik; na eerste keer laden gecachet).

## Technisch (kern)
Canvas 1080×1350/1920, geometrie uit originele flyers (banner 91/rij 120, pijl-inkeping, schuinte +75, staffel). `drawAsset(ctx,key,cx,cy,h,{tint,alpha,rot,flip})` + `TINT_CACHE`; eventlogo's 300 px hoog ingebed; palm1–4 (700 px) voor flair, hoeken en leeg-template-silhouetten. Video: MediaRecorder → MP4 (Safari/iPhone) / WebM (Chrome).

## Testaanpak (herhaald deze sessie)
Playwright headless: invoer via echte `input`-events (NaN-regressieroute v2), exports voor post leeg/foto, story leeg, en losse animatieframes via `render(1.25)` / `render(7.55)` / `render(8.9)`; daarna visueel beoordelen. Fontcheck via `document.fonts.check('76px CostaTitle')`. Nul console-errors.

## Openstaand (op volgorde van impact)
1. **Nieuwe events** met origineel logo in map 01/02: Neon Party, Student Night, Matchday, Photobooth, Pitcher, Pasen/Pinksteren (Happy Thursday zit er al in sinds v6.1).
2. **Transparante actie-overlays inbouwen** (map `Acties_transparant` in de templates-zip): als kiesbare badge-laag (10 shots, Stelz met/zonder badge, WK 2026).
3. **Map "10 Fotos" / meer cutouts**: nieuwe uitgeknipte mensen aanleveren als `cutouts_batch2`; zelfde pipeline (alpha-bbox, WebP).
4. **Rijenband per template uit het artwork afleiden** — dan pas kunnen de balken groter (zie de teruggedraaide `bandBottom` hieronder bij v7.9).
5. Fontlicentie designer checken (map 05); huidige Oswald is OFL (gratis, geen risico).

## Bestanden in de repo
- `index.html` — de volledige app (dit is wat gedeployed wordt).
- `docs/costa-overdracht.md` — dit document.
- `docs/costa-generator-instructie.md` — korte gebruiksuitleg voor collega's.
