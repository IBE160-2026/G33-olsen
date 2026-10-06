# Tilbakemelding på product brief

| | |
|---|---|
| **Gruppe** | G33 – G33-olsen |
| **Product brief** | `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md` (commit 97c3b6b) |
| **Tilbakemelding fra** | Faglærer i IBE160 (utarbeidet med KI-støtte) |
| **Dato** | 2026-10-06 |

## Samlet vurdering

- **Godt utgangspunkt med justeringer.** Gruppen kan gå videre og innarbeide punktene under.

**Det som er bra:**

1. Problemet og brukeren er svært konkrete. Innkjøperen hos en fiktiv meieriprodusent som må vite om beger og lokk til blåbæryoghurt kommer i tide når produksjonsplanen endres, er et godt og realistisk scenario.
2. Suksesskriteriene er svært gode: tre forhåndsdefinerte fiktive scenarier (økt produksjonsplan, emballasjemangel og forsinket leveranse) som hvert skal vise berørt materiale, årsak og handlingsalternativer. Det gir direkte grunnlag for tester og demo. Fiktive data og tydelig «ikke automatisk bestilling» er også gode valg.

**De viktigste endringene:**

1. Beregningsreglene må skrives ut. Hvordan regnes bruttobehov fra BOM, nettobehov mot lager, «recommended order quantity» og «latest safe order date»? Brukes gjennomsnittlig eller verste registrerte ledetid? Finnes sikkerhetslager og minste bestillingsmengde? Dette er kjernen dere må kunne kontrollere at Claude Code har regnet riktig.
2. Planlegg AI-forklaringen slik at sensor kan kjøre appen: hvilken språkmodell, hvor nøkkelen ligger, og en fallback (for eksempel en malbasert forklaring) når nøkkel mangler. Beskriv også at KI-en bare forklarer tall appen allerede har beregnet, og ikke regner selv.
3. Vurder omfanget i v1. Behovsberegning, fristvarsler, leverandørstatistikk, registrering av leveranser, AI-forklaring, trendstatistikk og innlogging er mye for et semester. Prioriter, og flytt trendstatistikken til et senere trinn.

## Vanskelighetsgrad og gjennomførbarhet

### Vurdert vanskelighetsgrad

- **Vanskelig**

**Sammenlignbart med:** 4) KI-støttet MRP II, avgrenset til delmodulene 4.4 BOM og 4.5 MRP, med leverandøroppfølging og KI-forklaring i tillegg. Avgrensningen gjør prosjektet mer håndterbart enn hele forslag 4, men det er fortsatt domenelogikk som må stemme, og flere moduler som henger sammen.

**Begrunnelse:**

| Faktor | Nivå (lav / middels / høy) | Kommentar |
|---|---|---|
| Domenelogikk – hvor mange og hvor kompliserte regler og beregninger må stemme? | Høy | BOM-eksplosjon, nettobehov per uke, bestillingsmengde, siste trygge bestillingsdato ut fra ledetid, og leverandørstatistikk (snittledetid og forsinkelser). Små feil gir feil varsler. |
| Datamodell – antall entiteter og relasjoner mellom dem | Middels–høy | Produkt, materiale, BOM-linjer, lager, leverandør, leverandør–materiale, innkjøpsordre/leveranse og produksjonsplan per uke. |
| Brukere, roller og innlogging | Lav–middels | «Simple login» med én rolle. Vurder om innlogging trengs i det hele tatt i en demo med fiktive data. |
| KI-funksjonalitet i appen, f.eks. kall til språkmodell, prompts i koden og håndtering av usikre svar | Middels | KI forklarer varsler og anbefalinger i klartekst. Avgrenset og fornuftig, men KI-en må ikke finne på tall. |
| Integrasjoner og eksterne tjenester, f.eks. API-er, betaling og e-post | Lav–middels | Ingen ERP-integrasjon. Bare LLM-API-et er eksternt. |
| Sanntid, samtidighet eller flere brukere som påvirker hverandre | Lav | Ingen sanntid. |
| Filhåndtering, f.eks. opplasting, PDF-lesing og eksport | Lav | Forhåndslastede data, ingen opplasting nevnt. |
| Sikkerhet og personvern | Lav | Fiktive data, ingen personopplysninger. |

**Hva vanskelighetsgraden betyr for dere:**

- _Vanskelig:_ Et vanskelig prosjekt gir større mulighet for toppkarakter, men også større risiko. Definer en minimal versjon som sikkert kan bli ferdig, og legg resten i tydelige trinn etterpå. For ProsjektMelk er minimumsversjonen: endre produksjonsplan → se beregnet behov og mangel per materiale → se siste trygge bestillingsdato. Når de tre scenariene gir riktig svar, legger dere til leverandørsammenligning, AI-forklaring og statistikk.

### Gjennomførbarhet med BMAD og Claude Code

Dere skal planlegge med BMAD (product brief → PRD → arkitektur → epics og stories) og implementere med Claude Code. Vurderingen under tar hensyn til at det må være tid til hele denne flyten, og til testing, retting og README til slutt.

| Spørsmål | Vurdering (OK / risiko / stor risiko) | Kommentar |
|---|---|---|
| **Tid og omfang** – kan v1 realistisk bli ferdig og stabil i løpet av semesteret, med tid til flere iterasjoner? | Risiko | Kjerneberegningen er overkommelig, men summen av funksjoner i «First Version» er stor. Git-loggen viser bare briefen så langt, så PRD bør komme raskt. |
| **BMAD-flyten** – er briefen konkret nok til at PRD, arkitektur og stories kan lages uten store hull, og blir det overkommelig mange stories? | OK | Svært konkret om data, brukere og scenarier. PRD-en må legge til beregningsreglene. |
| **Egnet for Claude Code** – bruker løsningen en vanlig, godt dokumentert teknologistakk som Claude Code håndterer godt, eller krever den nisjeteknologi, spesialmaskinvare eller mye manuell konfigurasjon? | OK | Et nettleserdashboard med database og enkle beregninger passer godt. |
| **Kontroll på KI-ens arbeid** – kan gruppen selv avgjøre om koden gjør det riktige? Krever domenet kunnskap gruppen ikke har, f.eks. avanserte beregninger eller fagregler, så er det vanskelig å kvalitetssikre. | Risiko | MRP-logikk er lett å få subtilt feil. Med de tre scenariene og håndregnede fasitsvar (for eksempel i et regneark) kan dere likevel kontrollere resultatet godt. |
| **Testbarhet** – finnes det tydelige regler og forventede resultater som tester kan skrives mot? | OK | De predefinerte scenariene er nesten ferdige testtilfeller. Skriv forventet resultat (materiale, mengde, dato) for hvert av dem. |
| **Kjørbar for sensor** – kan appen kjøres lokalt etter README, uten gruppens nøkler, betalte kontoer eller egen infrastruktur? | Risiko | Med forhåndslastede fiktive data går det fint, så lenge AI-forklaringen har en fallback uten nøkkel og testbrukeren for innloggingen står i README. |
| **Avhengigheter og kostnader** – krever løsningen betalte API-er, f.eks. språkmodeller, og finnes det en plan for kostnad, testmodus eller mock-data? | Risiko | Språkmodell er ikke valgt, og kostnad og testmodus er ikke omtalt. |

**Konklusjon om gjennomførbarhet:**

- **Gjennomførbart med justert omfang.** Se forslagene under.

**Forslag til justering av omfang eller vanskelighetsgrad:**

1. Del v1 i trinn: (1) behovsberegning, mangelvarsel og siste trygge bestillingsdato for de tre scenariene, (2) leverandørsammenligning og registrering av leveranser, (3) AI-forklaring, (4) trendstatistikk. Trinn 4 kan flyttes til «Outside the First Version» hvis tiden blir knapp.
2. Hold BOM-en på ett nivå (produkt → råvare og emballasje) og bruk én fast regel for ledetid i v1. Dropp innlogging hvis den ikke tilfører noe i demoen, eller bruk en ferdig testbruker.

## Hvorfor product brief er viktig for mappen

Product brief er utgangspunktet for PRD, arkitektur, stories og til slutt koden. Del 1 av mappen vurderes blant annet på om sensor kan følge en sporbar vei fra plan til ferdig app. Den vurderes også på om appen gjør det dere har beskrevet, om den er testet, om den er godt designet, og om den kan kjøres etter README. Et uklart, for stort eller for lite brief gjør alt dette vanskeligere senere. Det er mye enklere å rette nå enn sent i semesteret.

## 1. Gjennomgang av briefens deler

| Del av brief | Status | Kommentar |
|---|---|---|
| Executive Summary – er det klart hva appen er, og hvilket problem den løser? | OK | Klart: beslutningsstøtte for innkjøper som viser hvordan ukeplanen påvirker materialbehov, leverandørrisiko og innkjøpsbehov. |
| The Problem – er problemet konkret, med reelle situasjoner og brukere? | OK | Konkret med sene planendringer, mangel på beger og lokk og ukjente faktiske ledetider. |
| The Solution – beskriver løsningen brukeropplevelsen, ikke bare teknologi? | OK | God beskrivelse av prioritetslisten brukeren får (mangelvarsel, bestillingsmengde og -dato, leverandørsammenligning). |
| What Makes This Different – er vurderingen ærlig og realistisk? | OK | Ærlig sammenligning med regneark og manuelle kontroller, og ingen oppdiktet «moat». |
| Who This Serves – er primærbrukerne tydelige, og vet vi hva de trenger? | OK | Innkjøper som primærbruker og produksjonsplanlegger som sekundærbruker er tydelig. |
| Success Criteria – kan kriteriene faktisk sjekkes eller testes? | OK | De tre scenariene er testbare. Legg til forventede tall for hvert scenario, så blir de fullverdige testtilfeller. |
| Scope – er det klart hva som er med i første versjon, og hva som ikke er det? | Juster | Tydelig avgrensning utad, men «First Version» inneholder mange funksjoner. Prioriter dem og beskriv beregningsreglene. |
| Vision – henger visjonen sammen med resten uten å blåse opp omfanget? | OK | Prognoser, produksjonsplanlegging og ERP-integrasjon er holdt utenfor v1. |

## 2. Utgangspunkt for del 1 av mappen

Punktene følger kriteriene i sensorveiledningen for del 1. Vektene i parentes viser hvor mye hvert kriterium teller i del 1.

| Kriterium i del 1 | Hva briefen bør legge til rette for | Status | Kommentar |
|---|---|---|---|
| **1. Prosess og KI-styring** (30 %) | Brief som er presis nok til at PRD og stories kan bygges direkte på den, slik at krav kan spores fra brief til kode. | OK | Briefen er presis og er oppdatert i flere commits. Fortsett å oppdatere den når beregningsreglene er avklart. |
| **2. Funksjonalitet og omfang** (20 %) | Realistisk omfang for gruppen og semesteret: en tydelig kjerneflyt som kan bli ferdig og stabil, og nok innhold til å vise reell funksjonalitet. | Juster | Mye reell funksjonalitet, men fare for at ikke alt blir ferdig og stabilt. Del opp i trinn. |
| **3. Kvalitetssikring og testing** (15 %) | Suksesskriterier og funksjoner som er konkrete nok til å bli testtilfeller. | OK | Scenariene og beregningene egner seg svært godt for automatiske tester. |
| **4. Design og brukeropplevelse** (10 %) | Tydelige brukere og brukssituasjoner som designet kan bygges rundt, gjerne med de viktigste skjermbildene eller flytene skissert. | OK | Prioritetslisten og «identifiser neste handling uten å regne selv» er gode designmål. Skisser dashboardet og detaljvisningen for et varsel. |
| **5. Kodekvalitet og arkitektur** (10 %) | Teknologivalg som er begrunnet og ikke mer komplekse enn appen trenger. | OK | Ingen teknologivalg i briefen. Hold beregningslogikken i egne, testbare funksjoner adskilt fra brukergrensesnitt og KI-kall. |
| **6. README og kjørbarhet** (10 %) | Løsning som andre kan kjøre lokalt uten betalte kontoer, og uten tilgang til gruppens egne tjenester og nøkler. | Juster | Fiktive seed-data er bra. Planlegg `.env.example`, fallback for AI-forklaringen og testbruker. |
| **7. Ryddighet i repoet** (5 %) | En plan for hvor hemmeligheter, testdata og dokumentasjon skal ligge. | Juster | Planleggingsdokumentene ligger i `.docs/`, som er en skjult mappe. Lenk til den fra README, slik at sensor finner den. Planlegg også hvor seed-data og testscenarier skal ligge. |

## 3. Neste steg for gruppen

1. Skriv beregningsreglene (bruttobehov, nettobehov, bestillingsmengde, siste trygge bestillingsdato, ledetidsregel) og regn ut forventet resultat for de tre scenariene for hånd.
2. Prioriter funksjonene i v1 i trinn, og flytt trendstatistikk og eventuelt innlogging til senere trinn.
3. Velg språkmodell og beskriv fallback uten nøkkel. Gå deretter videre til PRD.

Oppdater product brief i repoet når dere har gjort endringene, slik at historikken viser hvordan planen utviklet seg. Det er en del av prosessen sensor ser etter.
