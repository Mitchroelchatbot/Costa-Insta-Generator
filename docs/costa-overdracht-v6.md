# Costa Weekplanning Generator — overdracht v6 (29 juli 2026)

## Wat het is
Eén zelfstandig HTML-bestand (`costa-generator.html`, ~14 MB) voor Insta-post (4:5), story (9:16) én TV-scherm (16:9) dat wekelijkse Instagram-posts (4:5), stories (9:16) én story-video's (9 sec animatie, MP4 via Safari/iPhone) genereert voor Café Costa. **Status: live** op costainstacreator.vercel.app (deploy = dit bestand als `index.html` naar Vercel slepen). Invoerformaat en parsing ongewijzigd sinds v2 — zie `costa-generator-instructie.md`.

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
2. **Nieuwe events** met origineel logo in map 01/02: Happy Thursday, Neon Party, Student Night, Matchday, Photobooth, Pitcher, Pasen/Pinksteren.
3. **Transparante actie-overlays inbouwen** (map `Acties_transparant` in de templates-zip): als kiesbare badge-laag (10 shots, Stelz met/zonder badge, WK 2026).
3b. **Map "10 Fotos" / meer cutouts**: nieuwe uitgeknipte mensen aanleveren als `cutouts_batch2`; zelfde pipeline (alpha-bbox, WebP).
4. Fontlicentie designer checken (map 05); huidige Oswald is OFL (gratis, geen risico).
5. Netlify-tegoed of definitief Vercel.

## Bestanden
- `costa-generator.html` — actuele v4 (deploy: als `index.html` naar Vercel).
- `costa-generator-instructie.md` — bijgewerkt (fotokiezer + animatie).
- `costa-overdracht-v6.md` — dit document.
