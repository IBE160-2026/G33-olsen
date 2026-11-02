# Git-oppdatering og PRD-status

## 2026-11-02 kl. 20:07 — Jarle Austreim Olsen

### Oppgave
Hente siste endringer fra GitHub og lese nye regler for sesjonslogging. Brukeren presiserte at Claude-referanser i arbeidsflyten også skal gjelde Codex, og at arbeidet skal loggføres for hver sesjon. Samtalen omfattet også status etter forrige økt, ordrette utdrag av faglærerens tilbakemelding og forklaring av PRD.

### Plan
- Kontrollere lokal Git-status og hente fjernhistorikken.
- Lese nye instruksjoner og flette dem inn med bevaring av eksisterende lokalt arbeid.
- Dokumentere status og åpne produktbeslutninger med sesjonslogg-skillen.

### Resultat
- Hentet origin og flettet inn c849df1 og 599ba70 fra 26. oktober med merge-commit 168aaab.
- Oppdateringen la til CLAUDE.md og .claude/skills/sesjonslogg/SKILL.md. Skillen er lest i sin helhet og brukes også for Codex etter brukerens instruks.
- Lokale dokumentendringer og den utracked PRD-mappen ble bevart. Ingen push ble utført.
- Leste tilbakemeldingen datert 6. oktober, PRD-utkastet og tilhørende arbeidslogger/gjennomganger. Forklarte at PRD betyr Product Requirements Document og konkretiserer produktets oppførsel og testbare krav.
- Dokumentene viser at beregningsregler, nøkkelfri standardforklaring og prioritert v1-omfang er innarbeidet. PRD står fortsatt som draft; kvalitetsgjennomgangen peker på antakelser som må avklares før godkjenning.
- Verifikasjon: Git-merge fullførte uten konflikt; de eksisterende lokale dokumentendringene står fortsatt urørt. Ingen programkode ble endret, og tester var ikke relevante.
- Kl. 20:17: Brukte bmad-help, leste den installerte arbeidsflytkatalogen og kjørte konfigurasjonsleseren. Bekreftet at PRD fortsatt er draft, og at ingen UX-, arkitektur-, epic/story- eller sprintplanleggingsdokumenter finnes i de konfigurerte dokumentmappene. Anbefalt neste steg er bmad-prd for å ferdigstille eksisterende utkast og avklare A-1/A-2; deretter UX (anbefalt), arkitektur og epics/stories. Ingen nye åpne punkter ble opprettet.
- Kl. 20:27: Sammenlignet brief.md med faglærerens tre hovedpunkter, scope-spesifikasjonen, siste brief-beslutninger og PRD-ens brukerdefinisjon. Briefen er allerede oppdatert 18. oktober med beregningsregler, KI-fallback og trinnvis omfang. Anbefalte en kort samordning av Solution (standardforklaring først, KI valgfri), produksjonsplanleggerens rolle og den avgrensede demoen. Ingen produktdokumenter ble endret i denne vurderingen.

- Kl. 20:33: I pågående Assumption Audit godkjente brukeren at lagerdekning og materialbehov skal komme før vurdering av levering i tide. La presiseringen inn i briefens The Solution, oppdaterte datoen og registrerte beslutningen via BMAD memlog.py. Fast ledetid er et vurderingsgrunnlag, ikke leverandørbekreftelse; innkjøperen gir selv beskjed til produksjon. Elicitation og øvrig samordning av briefen fortsetter; ingen nye åpne punkter.
- Kl. 20:35: Brukeren godkjente begrepene lagerbeholdning, allokert mengde og disponibel mengde, med automatisk allokering fra tidligere produksjon i valgt uke. Presiserte dette i briefen og registrerte beslutningen i BMAD-minneloggen. Manuelle reservasjoner og innkommende ordre er fortsatt utenfor v1-beregningen. Kontrollerte samsvar med eksisterende ukentlig beregningsmodell og git diff --check; ingen kodeendring eller nye åpne punkter.

### Åpne punkter
- Avklare PRD A-1: Tilbakestilling av demodata krever bekreftelse før lokale endringer overskrives.
- Avklare PRD A-2: Navngitt scenariovelger med separate planøyeblikksbilder, felles leverandørhistorikk og definert startscenario etter tilbakestilling.
- Samordne briefens overordnede formuleringer med senere beslutninger: valgfri KI, produksjonsplanlegger som interessent uten egen v1-flyt og tydelig avgrenset Blueberry Yogurt-demo.
