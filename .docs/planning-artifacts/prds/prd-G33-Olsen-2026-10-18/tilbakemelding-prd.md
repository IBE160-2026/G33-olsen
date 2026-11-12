# Tilbakemelding på PRD

| | |
|---|---|
| **Gruppe** | G33 – G33-olsen |
| **PRD vurdert** | `.docs/planning-artifacts/prds/prd-G33-Olsen-2026-10-18/prd.md` (status `final`, sist endret 2026-11-03, commit 7135421) |
| **Dato** | 2026-11-12 |
| **Type** | Formativ tilbakemelding, ikke karakter |

## 1. Samlet inntrykk

PRD-en har en tydelig tese: en deterministisk beregningsmotor produserer alle tall, og historikk og KI forklarer bare. Den er kontrollert for «hva skjer hvis det går galt», og de fleste FR-ene kan bli testtilfeller. Risikoen er omfanget og lengden. Etter briefen har PRD-en vokst med varelager-posteringer, scenarioeide saldoer og tilgjengelighetskrav, og på ca. 7 800 ord er den vanskelig å bruke som felles referanse for UX, arkitektur og stories.

## 2. Dette fungerer godt

1. **Ærlig avgrensning og rekkefølge.** §9 deler arbeidet i fire inkrementer med «Work starts on a later increment only after the previous increment and its tests are stable». §8 sier tydelig hva som ikke bygges (sikkerhetslager, innlogging, trenddashbord, automatisk bestilling).
2. **KI under kontroll.** FR-21 («never calculate, correct, rank, extrapolate»), FR-22 (lukket plan, serveren eier teksten) og FR-23 (nøkkelløs fallback) gir en testbar avgrensning. DAR-2 forbyr live API-kall i tester.
3. **Konkrete verdier gir ferdige testtilfeller.** SM-6 og SM-7 oppgir hånd-regnede tall (f.eks. allokering/tilgjengelig 0/75, 60/15, 75/0 kg og 60 kg anbefaling), og SM-2 sier ærlig at fordelen over regneark er en hypotese, ikke påvist.

## 3. Oversikt per dimensjon

| Dimensjon | Vurdering | Kort begrunnelse |
|---|---|---|
| Beslutningsklarhet | sterk | Beslutninger, avveininger og grenser står eksplisitt (§1.1, §7, §9). |
| Substans | sterk | Ingen persona- eller NFR-pynt. NFR-ene har tall (1280×720, 390×844, 4,5:1). |
| Strategisk sammenheng | god | Tydelig tese, men lagerposteringene (FR-25/26) tjener den knapt. |
| Testbarhet | god | Mange FR-er er svært gode. FR-10, 11, 12, 16, 18 og 24 mangler egne testbare konsekvenser. |
| Scope og avgrensning | god | Utelatelser er eksplisitte, men det finnes ingen kuttlinje som er realistisk for ett semester. |
| Brukbarhet for neste fase | god | ID-ene er sammenhengende, men innholdet er repetert og delegert til to eksterne dokumenter. |
| Formtilpasning | god | Passer et beslutningsstøtteverktøy. UJ-ene har navnløs hovedperson, og dokumentet er tyngre enn nødvendig. |

## 4. Viktigste forbedringer

### 4.1 Omfanget er større enn et semester tåler
- **Alvorlighet:** høy
- **Hvor:** §9 Increment 2, FR-25, FR-26, FR-1 («New goods receipts … belong only to the active Demo Scenario»)
- **Problem og hvorfor det betyr noe:** Briefen ble bedt om å prioritere og holde det enkelt. PRD-en prioriterer (inkrementer, trender og innlogging ute), men legger samtidig til delvis kompleks logikk: varemottak med avvik, atomisk kobling til leveringshistorikk, replay-beskyttelse, produksjonsbekreftelse med allokeringskonflikt og scenarioeide saldoer. Dette er plassert i Increment 2, foran leverandørsammenligning (Increment 3), som scenario 3 trenger. Scenario 3s risikovisning bruker bare seed-historikk og er uavhengig av lagerposteringene, men blir likevel forsinket av dem.
- **Forslag:** Bytt rekkefølge, slik at leverandørhistorikk og risiko kommer som Increment 2 og lagerposteringer som Increment 3. Marker FR-25/26 som `[STRETCH]` og skriv en kuttlinje: «Versjon 1 er komplett etter inkrement 1, 2 og 4. Inkrement 3 leveres bare hvis alle tester for 1–2 er stabile.» Hvis dere beholder FR-26, kan allokeringskonflikt-varselet bli en senere forbedring (bare blokker ved for lite fysisk lager).

### 4.2 Flere FR-er mangler testbare konsekvenser
- **Alvorlighet:** høy
- **Hvor:** FR-10 («must distinguish positive shortage, no shortage, blocked calculation, order-today, and overdue states»), FR-11, FR-12, FR-16, FR-18, FR-24
- **Problem og hvorfor det betyr noe:** Disse har ingen «Consequences», eller de peker bare til eksterne kontrakter. En story-skriver kan ikke se hva «ferdig» er uten å åpne kontraktene og tolke dem.
- **Forslag:** Skriv FR-10 slik: «Hver Material-rad viser nøyaktig én av statusene `Shortage`, `No shortage`, `Blocked`. For `Shortage` vises i tillegg `Order today` eller `Overdue by N calendar days` etter FR-7. Consequences: (1) I scenario 3 viser Yogurt lid `Shortage` med 100 lids og deadline 2026-11-07. (2) Mangler Fixed Lead Time, vises `Shortage` uten deadline, og raden er ikke `Overdue`. (3) Hver status har en ikke-fargebasert markør (NFR-5).» Gi FR-11 ett eksempel på en Standard Explanation for ett scenario.

### 4.3 PRD-en er lang og repeterer regler
- **Alvorlighet:** medium
- **Hvor:** FR-25 (ca. 15 «consequences»), scenarioeierskap i FR-1, FR-3, FR-4, FR-13 og FR-25; §12–§13
- **Problem og hvorfor det betyr noe:** Samme regel står flere steder, og PRD-en inneholder endringshistorikk («resolved 2026-11-02», «The resumed review … found no remaining blocking finding»). Det øker risikoen for at tekstene glir fra hverandre, og gjør det tungt å lage epics. Verdiene for scenario 1 og 2 står heller ikke i PRD-en, bare i eksterne dokumenter.
- **Forslag:** La hver regel stå ett sted og henvis dit. Flytt §12–§13 til en addendum eller endringslogg. Legg inn en tabell: `FR → inkrement → UJ → SM`, og en liten scenario-tabell (scenario | endring | Material | mangel | siste bestillingsdato) slik at §9 er selvstendig.

### 4.4 Uavklart modelltilgang er ikke registrert som åpent punkt
- **Alvorlighet:** medium
- **Hvor:** FR-24 («approved `gpt-5.6-sol` model»), §12 («No unresolved product assumptions remain»), siste avsnitt i §13
- **Problem og hvorfor det betyr noe:** Det siste avsnittet sier at tilgang til modellen ikke er verifisert. Likevel sier §12 at alt er avklart. Nøkkel-, kostnads- og tilgangsrisiko er det briefen ba om å få planlagt, og FR-20 inneholder dessuten løsningsvalg («signed anonymous demo session»). Kostnadsgrensene ligger bare i den eksterne AI-kontrakten.
- **Forslag:** Legg inn `OQ-4: Er modellen tilgjengelig i prosjektets egen konto? Frist: før Increment 4. Hvis nei: Standard Explanation er endelig leveranse, og Increment 4 utgår.` Skriv i FR-24 de to–tre kostnadsgrensene (maks kall per økt, tidsavbrudd, maks tokens) i én linje. Flytt «signed session» til arkitekturen eller vis til hvorfor den er et krav.

### 4.5 SM-2 mangler bestått-kriterium
- **Alvorlighet:** medium
- **Hvor:** SM-2 («in a documented usability check, the tester identifies …»)
- **Problem og hvorfor det betyr noe:** Måleren sier ikke hvor mange testpersoner, hvilke oppgaver eller hva som regnes som bestått. Da kan den ikke avgjøres.
- **Forslag:** «Minst 3 testpersoner utenfor gruppen løser oppgavene T1–T5 (T1: finn manglende materiale og mengde i scenario 3) uten hjelp. Bestått: minst 4 av 5 oppgaver per person. Observerte problemer logges i `.docs/`.»

## 5. Mekaniske funn

- ID-ene er sammenhengende og unike: FR-1–26, UJ-1–5, SM-1–9, SM-C1–C3, NFR-1–8, DAR-1–6. Krysshenvisninger (f.eks. FR-13 → FR-25) og lenkene til kontraktdokumentene og `deferred-work.md` løser seg.
- Ingen inline `[ASSUMPTION]`-tagger; indeksen i §12 er konsekvent tom (se 4.4).
- Glossar-avvik: «Material need» (§1, UJ-1, §9) er ikke definert (glossaret har Gross/Net Need). «Usable Inventory» og «Available Quantity» overlapper og forklares først i FR-8. «Completed», «Alert» og «Incomplete delivery record» (FR-25) mangler i glossaret.
- Bare noen FR-er sier «Realizes UJ-x». FR-5–FR-10 og FR-14–FR-24 mangler kobling. UJ-ene har hovedpersonen «the buyer» uten navn.

## 6. Før dere går videre

**Må avklares før UX, arkitektur og epics:**
- Hvilke inkrementer er faktisk «ferdig» for dere, og i hvilken rekkefølge (4.1)?
- Hva er forventet resultat for scenario 1 og 2, direkte i PRD-en (4.3)?
- Er modelltilgangen realistisk, og hva leveres hvis ikke (4.4)?

**Kan vente:** eksempeltekst for Standard Explanation (fylles ut i UX), SM-2-oppgavene, glossar-opprydding.

**Oppfølging av briefen-tilbakemeldingen: delvis.** Beregningsregler (MRP-R1–R10 i `calculation-rules.md`) og AI-plan med nøkkelfri fallback og valgt modell er fulgt opp. Trender er flyttet ut, innlogging er fjernet og inkrementer er laget. Men det samlede omfanget er økt (lagerposteringer, scenarioeide saldoer), ikke redusert.

## 7. Refleksjonsspørsmål

1. FR-25 og FR-26 er langt mer detaljerte enn de øvrige kravene. Kom detaljene fra dere, fra en gjennomgang med KI, eller fra en reell kundeting? Hvordan avgjorde dere at dette var verdt tiden i et semesterprosjekt?
2. Hvordan sjekket dere at KI-genererte konsekvenser (f.eks. «980-cup current-balance example») er riktige? Er de regnet ut for hånd, eller godtatt fordi de så rimelige ut?

_Vurdert mot kvalitetsrubrikken i bmad-prd. Dette er veiledning, ikke sensur._
