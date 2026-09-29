# Quickscan financiële positie (Schevichoven Groeit)

Dit project bouwt een instrument waarmee een boer uit de jaarrekeningen van zijn laatste drie boekjaren en een beperkt aantal operationele antwoorden een gestandaardiseerd rapport van drie tot vijf pagina's krijgt. Het taalmodel leest en rubriceert; alle rekenwerk daarna gebeurt met vaste formules in code. Het benchmark (referentiebestand) is het product, het rapport is de vorm.

Lees eerst `docs/bouwplan.md` (fasering, architectuur, open vragen) en `docs/rekenregels.md` (definities en formules).

## Taal

- Gesprek met de ontwikkelaar mag in het Nederlands of Engels; alles wat een boer of Schevichoven Groeit te zien krijgt (rapport, interface, foutmeldingen, documentatie in `docs/`) is Nederlands.
- Registers: lange zinnen met puntkomma's en bijzinnen, onpersoonlijk waar dat natuurlijk leest ("Voorgesteld wordt", "dient te worden"), zinsopeningen als Daarbij, Daarnaast, Tevens, Aan de andere kant, Concluderend.
- Geen em-streepjes. Geen vetgedrukte kopjes binnen een alinea. Geen retorische tegenstellingen of korte pakkende slotzinnen.
- Code, variabelen en commit-berichten mogen Engels zijn; gebruikersgerichte tekst nooit.

## Harde regels (een wijziging die er een breekt is fout)

1. Elk cijfer in het rapport heeft een herkomst: de regel in de jaarrekening of "opgave van de boer". Een getal zonder herkomst komt niet op de pagina en niet in het datamodel.
2. Onttrekken in plaats van verzinnen: wat niet kan worden vastgesteld blijft leeg, met de reden ernaast. Nooit een schatting presenteren als waarneming.
3. Geen assurance-taal. De woorden samenstellingsverklaring, beoordelingsverklaring en controleverklaring komen nergens voor, ook niet in commentaar of tests van zichtbare teksten, en er staat niets dat suggereert dat de jaarrekening is gecontroleerd of goedgekeurd. Zeg dat de aangeleverde cijfers als uitgangspunt zijn genomen zonder dat de juistheid is vastgesteld.
4. Het rapport vermeldt dat het niet bestemd is voor de beoordeling van kredietwaardigheid.
5. Geen vergelijking in een groep van minder dan zeven bedrijven.
6. Geen echte data in de repository: geen echte jaarrekeningen, geen auditbestanden, geen gelicentieerde publicaties (zoals KWIN) en geen normcijfers daaruit. Fixtures zijn uitsluitend synthetisch en gelabeld "Voorbeeldbedrijf; de cijfers zijn fictief." Gebruik niet de testbedrijven van Regeneraid.
7. Geen credentials in de repository, geen `.env` lezen of committen. Sleutels komen uit omgevingsvariabelen of GitHub-secrets.
8. Aansluitingscontroles (balans telt op, som van gerubriceerde posten gelijk aan totaal, mutatie eigen vermogen verklaard) zijn een blokkade en geen waarschuwing.
9. Rekenwerk gebeurt niet door het taalmodel. Geldbedragen nooit als drijvende komma; gebruik hele euro's of decimalen met een vaste afrondingsregel.

## Bouwvolgorde

Eerst rekenkern en controles, dan rapport, dan extractie, dan pilotomgeving, dan referentiebestand. Melkvee eerst, daarna akkerbouw. Zie `docs/bouwplan.md`.

## Compliance in het rapport

Het rapport en de voetregels bevatten, in het Nederlands: dat de posten met hulp van een taalmodel zijn gerubriceerd en dat de berekening met vaste regels (versie x.y) is uitgevoerd; dat de aangeleverde jaarrekening als uitgangspunt is genomen zonder dat de juistheid is vastgesteld en dat dit geen samenstellings-, beoordelings- of controleopdracht is; dat het rapport niet bestemd is voor de beoordeling van kredietwaardigheid; en dat identificerende gegevens vóór het uitlezen zijn verwijderd en cijfers alleen geaggregeerd en geanonimiseerd in het referentiebestand komen, na aparte toestemming.

## Vormgeving van het rapport

Geen app-schermen: pagina's op perkament, A4-verhoudingen, ruime marges, lopende kop met bedrijfsnaam en boekjaar, paginanummer, voet met versie van de rekenregels. Lettertypen EB Garamond (lopende tekst) en Libre Baskerville (koppen en cijfers). Kleuren: pagina #F5F0E8, papier #FDFAF4, rand #D8D2C4, gedempte inkt #9A9E7E, lopende inkt #2D3A1E, tabelinkt #4A5530, groen kop #2D4A2D, groen accent #4A7A4A, waarschuwing #FBF0DC / #C4861A, negatief #8B3A2A. Grafiekkleuren: #417F2C, #C08312, #2A6E9E, #9E3A20. Tabelkoppen als gedempte hoofdletters, cijfers rechts uitgelijnd met `font-variant-numeric: tabular-nums`. Grafieken klein, inline SVG, zonder animatie.

## Werkwijze

- Werk op een branch en open een pull request; commit niet direct op de hoofdbranch.
- Elke rekenregel krijgt een test met een handgerekend voorbeeld; De Hoge Kamp (`fixtures/de-hoge-kamp.json`) is de gouden referentie.
- Bij twijfel over een definitie: stel één vraag met concrete opties in plaats van te gokken.
