# Costa Insta Generator

Wekelijkse Instagram-posts (4:5), stories (9:16) en TV-schermen (16:9) voor Café Costa — in huisstijl, zonder designer. Eén zelfstandig HTML-bestand: plak je weekplanning en download PNG of video (9 sec, MP4 op iPhone/Safari).

**Live:** costainstacreator.vercel.app

## Gebruik

Zie [docs/costa-generator-instructie.md](docs/costa-generator-instructie.md) voor de korte gebruiksinstructie.

## Deploy

`index.html` is de volledige app (alle templates, logo's, foto's en fonts ingebakken, ~14 MB). Elke push naar `main` deployt automatisch via Vercel zodra de repo aan het Vercel-project gekoppeld is; handmatig kan ook door `index.html` naar Vercel te slepen.

## Ontwikkeling

Technische details, beslissingen en openstaande punten: [docs/costa-overdracht.md](docs/costa-overdracht.md). Testen gaat headless (Playwright): invoer via echte input-events, canvas exporteren, renders visueel beoordelen — zie de overdracht.
