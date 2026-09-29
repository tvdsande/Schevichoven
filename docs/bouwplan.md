# Bouwplan Quickscan financiële positie

Versie 0.1, 29 september 2026. Gebaseerd op het voorstel van 28 september 2026 en CONTEXT-quickscan.md. Dit plan beschrijft wat er in Claude Code met GitHub als repository gebouwd wordt, in welke volgorde, en welke informatie daarvoor nog moet worden aangeleverd.

## 1. Uitgangspunten uit het voorstel die de bouw bepalen

- Het taalmodel leest de jaarrekening en rubriceert de posten; alle rekenwerk daarna gebeurt met vaste formules in code, zodat dezelfde invoer altijd dezelfde uitkomst geeft.
- Elk cijfer in het rapport draagt zijn herkomst (regel in de jaarrekening of antwoord van de boer); een cijfer zonder herkomst komt niet in het rapport.
- Wat niet kan worden vastgesteld blijft leeg met de reden erbij; er wordt nooit geschat.
- De aansluitingscontroles zijn een blokkade en geen waarschuwing; ze worden gebouwd vóór de vormgeving (voorstel 7.6).
- Het rapport werkt zonder vergelijkingskolom; de vergelijking komt pas in stap 3, met een ondergrens van zeven bedrijven per groep.
- Verboden termen (samenstellings-, beoordelings-, controleverklaring) en goedkeurende taal komen nergens voor; dit wordt in code getoetst.

## 2. Bouwvolgorde

De volgorde volgt de drie stappen uit paragraaf 8.1 van het voorstel, met de risicovolle onderdelen vooraan.

### Fase 0: fundament
- Repository, CI, conventies, CLAUDE.md met de harde regels uit §8 van de context (geen echte data, geen .env lezen, geen assurance-taal).
- Rekenschema en rubriceringsschema vastleggen als versioned document (`docs/rekenregels.md`, versie `0.1`).
- Synthetische fixtures: De Hoge Kamp (melkvee) exact volgens de cijfers uit de context, later één per overige tak.

### Fase 1: rekenkern en controles (geen model, geen UI)
- Datamodel voor het vaste rekenschema (opbrengsten, toegerekende kosten, bedrijfskosten, afschrijving, rente, balansposten, operationele antwoorden), elk veld met herkomst.
- Rekenregels: saldo, resultaat voor rente en afschrijving, resultaat, solvabiliteit, rentelast, aflossingscapaciteit, kritieke melkprijs, kengetallen per 100 kg melk, per tak.
- Aansluitingscontroles als blokkade: balans telt op; som gerubriceerde posten gelijk aan totaal in de jaarrekening; mutatie eigen vermogen verklaard uit resultaat en privé-onttrekkingen.
- Lege toestanden: geen balans betekent geen aflossingscapaciteit en een rapport van vier pagina's; niet te isoleren tak betekent geen takpagina.
- Uitkomst van De Hoge Kamp moet exact de cijfers uit de context reproduceren (zie §5 hieronder voor de controlesom die ik al heb nagerekend).

### Fase 2: rapport
- HTML-sjabloon in de huisstijl (EB Garamond, Libre Baskerville, tokens uit §7 van de context), A4-pagina's met lopende kop, paginanummer en versie van de rekenregels in de voet.
- Inline SVG-grafieken: driejaarsbalken, waterfall omzet naar resultaat, markering kritieke melkprijs.
- Vaste tekstblokken in Nederlands, in het register van de opdrachtgever; de compliance-regels uit §9 van de context als voetregels.
- Weergave met en zonder vergelijkingskolom.
- Export naar PDF via Playwright (Chromium is beschikbaar).
- Eerste concrete oplevering: het voorbeeldrapport (variant A), dat ook als HTML-artifact aan Schevichoven Groeit getoond kan worden.

### Fase 3: extractie met taalmodel
- Invoer: PDF met tekstlaag of scan; tekstextractie en waar nodig OCR.
- Anonimisering vóór extractie: naam, adres, BSN en fiscaal nummer worden verwijderd.
- Model rubriceert posten naar het vaste schema en geeft per post de bronregel en een zekerheidsmaat; onder de drempel blijft de post buiten de berekening.
- Gestructureerde uitvoer (JSON met schema-validatie); bij ongeldige uitvoer wordt niet gegokt.
- Evaluatieset: synthetische jaarrekeningen met bekende uitkomst, plus een meetbare foutmaat per post.
- Bevestigingsstap: de vijf kerncijfers (omzet, saldo, resultaat, balanstotaal, eigen vermogen) aan de boer voorleggen voordat verder wordt gerekend.

### Fase 4: pilot-omgeving (stap 2 uit het voorstel)
- Eenvoudige webinterface: upload, tak kiezen, operationele vragen, bevestiging, rapport.
- Naloopscherm voor de handmatige controle, met vastlegging van elke correctie (basis voor de vraag wanneer de naloop kan worden afgebouwd).
- Opslag in een EU-regio met bewaartermijn en verwijderfunctie.
- Toestemming per scan: apart vinkje voor opname in het referentiebestand.

### Fase 5: referentiebestand (stap 3)
- Geanonimiseerde en geaggregeerde opslag, groepsindeling per tak en omvangsklasse.
- Harde ondergrens van zeven bedrijven per groep, getest; onder de grens wordt geen vergelijking getoond.
- Vergelijkingskolom in rapport en takpagina.

### Volgorde per tak
Melkvee eerst, daarna akkerbouw (voorstel 8.2), daarna varkens en pluimvee, en fruitteelt en schapen pas na besluit (voorstel 8.3).

## 3. Voorgestelde opzet van de repository

```
quickscan/
  CLAUDE.md                harde regels, taalregister, conventies
  docs/
    rekenregels.md         het vaste rekenschema en de formules, met versienummer
    rubriceringsschema.md  de posten waarnaar het model rubriceert
    compliance.md          risico's en maatregelen uit hoofdstuk 7
  packages/
    core/                  schema, rekenregels, aansluitingscontroles (geen I/O)
    extract/               tekstextractie, anonimisering, model-aanroep, validatie
    report/                HTML-sjablonen, grafieken, PDF-export, tekstblokken
    reference/             referentiebestand, groepen, ondergrens
  apps/
    cli/                   scan lokaal draaien op een bestand
    web/                   pilot-interface (fase 4)
  fixtures/                uitsluitend synthetische jaarrekeningen
  .github/workflows/       lint, tests, secret-scan, taalcontrole
```

Voorgestelde techniek: TypeScript op Node, zod voor schema's, vitest voor tests, Playwright voor PDF. Geldbedragen als hele euro's of als decimalen met vaste afrondingsregel; nooit als drijvende komma in de rekenkern. Dit is een voorkeur; zie de vragen in §4.

## 4. Wat ik van jou nodig heb

### Blokkerend voor de start
1. GitHub: naam van de organisatie en de repository (voorstel: privé), en wie er beheerder is. Ik werk via de gekoppelde repository; toegang tot een nieuwe repository moet jij of de eigenaar verlenen.
2. Naam van het product en onder wiens naam het rapport wordt opgesteld (voorstel 8.3 en 7.4: het rapport moet vermelden wie het opstelt en in welke hoedanigheid). Voorlopig gebruik ik de werknaam "Quickscan".
3. Techniekkeuze: akkoord met TypeScript, of voorkeur voor Python. Ook of de bestaande Regeneraid-code een stack heeft waarbij dit moet aansluiten.
4. Het rubriceringsschema: welke posten het vaste rekenschema precies kent. Mijn voorstel is uit te gaan van het Referentiegrootboekschema (RGS) als openbare basis en daar de Quickscan-rubrieken op te leggen; ik controleer eerst of dat past bij wat boekhouders in de agrarische sector leveren. Graag jouw of Riks oordeel.
5. Formules per tak: alleen voor melkvee zijn de uitkomsten bekend. De kritieke melkprijs heb ik teruggerekend (zie §5), maar ik heb de definitie schriftelijk nodig, inclusief welke privé-onttrekkingen meetellen en of aflossing van het komende jaar of van het boekjaar wordt gebruikt.

### Nodig voor de extractie (fase 3)
6. Voorbeelden van de opmaak van jaarrekeningen in het veld: welke boekhoudpakketten en accountantskantoren leveren aan bij Schevichoven Groeit, en in welke vorm (PDF met tekstlaag, scan, Excel, XAF). Echte stukken mogen niet in de repository; ik werk met synthetische nabootsingen, maar die moeten realistisch zijn. Ik stel voor dat jij of Rik per tak twee of drie geanonimiseerde opmaken beschrijft (welke rubrieken, welke volgorde, welke afwijkende namen).
7. Keuze taalmodel en hosting: welke aanbieder en welke EU-regio, en of er al een verwerkersovereenkomst met een no-training-clausule is (voorstel 7.2). API-sleutels lever je niet in de chat aan; die komen als GitHub-secret of lokale omgevingsvariabele en ik lees geen .env-bestanden.
8. Of scans (zonder tekstlaag) in de eerste versie ondersteund moeten worden. Dit bepaalt of OCR nodig is en vergroot het foutrisico aanzienlijk.

### Nodig voor de pilot en het rapport
9. Bevestiging van de vier of vijf takken: het voorstel spreekt van vier takken, de context-tabel telt schapen apart, en 8.3 laat open of schapen en fruitteelt in de eerste versie zitten. Wat is de scope van v1?
10. Operationele vragen per tak: bijlage B is een goede basis; ik heb bevestiging nodig dat de lijst voor melkvee definitief is voordat het formulier wordt gebouwd.
11. Definitie van het versienummer van de rekenregels en wie een nieuwe versie mag vrijgeven.
12. Inhoud van de opdrachtvoorwaarden en de verwerkerstekst die in de interface getoond moeten worden (grondslag, bewaartermijn, apart vinkje voor het referentiebestand). Ik kan een concept schrijven, maar een jurist of accountant moet het nalezen (voorstel 7.4).
13. Welke accountant of jurist de tekstblokken nalees vóór de pilot.
14. Logo, kleurgebruik en of de Regeneraid-huisstijl ook geldt voor een rapport onder de naam van Schevichoven Groeit.

### Later
15. Eigendom en voorwaarden van het referentiebestand (voorstel 8.3), zodat ik de opslag en de rechten daarop goed kan inrichten.
16. Pilotbedrijven en werkwijze van de handmatige naloop: wie loopt na en met welk protocol.

## 5. Controle die ik al heb uitgevoerd op de cijfers van De Hoge Kamp

Alle bedragen in de context sluiten op elkaar: omzet 312.000 (252.200 + 31.000 + 22.800 + 6.000); saldo 210.100 (67,3%); bedrijfskosten 90.500; resultaat voor rente en afschrijving 119.600; resultaat 46.200; solvabiliteit 47,7%; aflossingscapaciteit 119.600 / 83.400 = 1,43; per 100 kg melk: melkopbrengst 48,50, toegerekende kosten 19,60, voersaldo 34,85, bewerkingskosten 9,33.

De kritieke melkprijs volgt uit: tekort = aflossing + privé-onttrekkingen − (resultaat + afschrijving) = 62.000 + 48.000 − 98.200 = 11.800; per kg 11.800 / 520.000 = 0,0227; kritieke prijs 0,485 + 0,0227 = 0,508. Rente zit al in het resultaat en telt dus niet nogmaals mee. Dit is mijn reconstructie en moet door jou worden bevestigd (vraag 5).

Voor 2023 en 2024 zijn alleen omzet, resultaat, solvabiliteit en melkprijs gegeven. De kritieke melkprijs en de aflossingscapaciteit voor die jaren kunnen daarom niet worden berekend en blijven leeg met de reden; dat is een bruikbare lege toestand voor het voorbeeldrapport.

## 6. Kwaliteitsborging in de pijplijn

- Elke rekenregel heeft een test met een handgerekend voorbeeld; De Hoge Kamp is de gouden referentie.
- Een taalcontrole in CI zoekt naar verboden termen en em-dashes in alle zichtbare teksten.
- Een secret-scan en een controle dat geen bestand met echte data (zoals het XAF-bestand of het KWIN-document) in de repository terechtkomt.
- Snapshot-tests van het rapport per tak, met en zonder vergelijkingskolom en met elke lege toestand.
- Extractie-evaluatie op de synthetische set met een vaste foutdrempel voordat een wijziging in prompt of model wordt vrijgegeven.

## 7. Voorstel voor de eerste stap na akkoord

Repository aanmaken, CLAUDE.md en `docs/rekenregels.md` schrijven, de rekenkern voor melkvee bouwen met De Hoge Kamp als testcase, en daarna het voorbeeldrapport (variant A) uit die kern renderen. Daarmee heeft Schevichoven Groeit snel iets concreets en ligt de basis voor de rest.
