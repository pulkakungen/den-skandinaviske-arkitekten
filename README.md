# Bullet Journal

En webbapp för bullet journaling. På Boox plattan skriver man med pennan och på datorn med tangentbordet eller musen. Allt synkas mellan enheterna via en egen Cloudflare Worker.

## Filer

| Fil | Vad den gör |
|---|---|
| `index.html` | Hela appen |
| `manifest.webmanifest`, `sw.js`, `icon-*.png` | Gör att den kan läggas på hemskärmen och startar utan nät |
| `worker/worker.js` | Synken, sparar sidorna i Cloudflare D1 |
| `worker/schema.sql` | Tabellen i databasen |
| `worker/wrangler.toml` | Inställningar för workern |

## Steg 1: Workern på Cloudflare

```bash
cd worker
npm install -g wrangler
wrangler login
wrangler d1 create bujo
```

Kopiera `database_id` som visas och klistra in den i `wrangler.toml`. Skriv också in adressen till GitHub Pages i `ALLOWED_ORIGIN`, till exempel `https://dittnamn.github.io`.

```bash
wrangler d1 execute bujo --remote --file=schema.sql
wrangler secret put SECRET     # välj ett långt lösenord
wrangler deploy
```

Spara adressen som visas, till exempel `https://bujo-sync.dittnamn.workers.dev`.

## Steg 2: Appen på GitHub Pages

1. Skapa ett repo, till exempel `bullet-journal`.
2. Lägg `index.html`, `manifest.webmanifest`, `sw.js`, `icon-192.png` och `icon-512.png` i roten. Mappen `worker` kan ligga med, den används inte av sidan.
3. Gå till Settings, Pages, välj grenen `main` och mappen `/ (root)`.
4. Appen finns sedan på `https://dittnamn.github.io/bullet-journal/`.

## Steg 3: På varje enhet

1. Öppna adressen.
2. Fyll i synkadressen och lösenordet i rutan som visas (kugghjulet öppnar den igen).
3. Lägg till appen på hemskärmen via webbläsarens meny.

På Boox: öppna i Chrome eller Boox webbläsare och ställ in snabb uppdatering för webbläsaren så känns pennan snabbare.

## Så funkar synken

* Varje sida sparas som ett eget dokument i D1.
* Sidan som är öppen hämtar nytt var 8:e sekund och direkt när appen kommer i fokus.
* Workern slår ihop ändringar, så om två enheter skriver samtidigt försvinner ingenting. Det som suddas bort på en enhet försvinner på alla.
* Utan nät sparas allt på enheten och skickas upp när nätet är tillbaka.
* Lösenordet sparas bara i webbläsaren på varje enhet.

Cloudflares gratisnivå räcker gott (D1: 5 GB och 100 000 skrivningar per dag).
