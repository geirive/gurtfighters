# Tilbakemeldingstjeneste

En liten Cloudflare Worker som tar imot tilbakemeldinger fra spillet og lager en GitHub-issue av hver. Spillet kan ikke lage issues selv, for da måtte GitHub-nøkkelen ligget åpent i `index.html`.

## Oppsett

1. **GitHub-nøkkel:** GitHub → Settings → Developer settings → Fine-grained tokens. Gi den tilgang til bare dette repoet, med «Issues: Read and write» og ingenting annet.
2. **Worker:** Lag en ny Worker på Cloudflare (gratisnivået holder) og lim inn `worker.js`.
3. **Hemmeligheter og variabler** (Worker → Settings → Variables and Secrets):

   | Navn | Type | Verdi |
   |---|---|---|
   | `GITHUB_TOKEN` | Secret | nøkkelen fra steg 1 |
   | `FEEDBACK_CODE` | Secret | kodeordet spillerne må skrive |
   | `REPO` | Tekst | `eier/gurtfighters` |
   | `ALLOWED_ORIGIN` | Tekst | adressen spillet ligger på, uten sti, f.eks. `https://eier.github.io` |

4. **Koble til spillet:** Lim Worker-adressen inn i `FEEDBACK_URL` øverst i skriptet i `index.html`, og øk `VERSION`.

Så lenge `FEEDBACK_URL` er tom, åpner spillet i stedet et ferdig utfylt issue-skjema på GitHub, som krever GitHub-konto.

## Vern mot spam

- Kodeordet sjekkes i Workeren og står ikke i spillet eller i repoet. Bytt det i Cloudflare hvis det kommer på avveie.
- Bare forespørsler fra `ALLOWED_ORIGIN` slippes inn. Det stopper andre nettsider, men ikke skript.
- Maks 3 meldinger per IP-adresse og 30 totalt per 10 minutter. Tellingen ligger i minnet og er omtrentlig; legg til en «Rate limiting rule» i Cloudflare for et hardere tak.
- Teksten kuttes til 1000 tegn, og et skjult felt fanger enkle roboter.

## Vern mot prompt injection

- Hver issue merkes `feedback` og starter med en linje om at innholdet er ubekreftet tekst fra en spiller.
- Spillerteksten legges i en kodeblokk. Kontrolltegn, usynlige tegn og `~~~` fjernes, så teksten kan ikke bryte ut av blokken, lage lenker eller nevne brukere.
- Ingenting leser disse issuene automatisk. Ikke koble en KI-agent eller GitHub Action til issues med `feedback`-merket; les dem selv og gi videre det du vil ha gjort.
