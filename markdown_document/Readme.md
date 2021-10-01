---
title: "Verktyg för RVU-rapport från Kollbar-data"
author: "Nikos Papakatsikas"
date: '2021-09-29'
output:
  html_document: default
  pdf_document: default
---

## Bakgrund
WSP har utvecklat ett verktyg för automatiserad produktion av RVU-rapporter baserade på data från undersökningar genomförda från Kollektivtrafikbarometern. Verktyget använder *markdown* för rapporten, *LaTeX* för rapportens grafiska element och stil samt programmeringsspråket *R* för dataanalyserna.

Verktyget möjliggör att en rapport för en tätort, kommun eller region skapas med väldigt liten ansträngning. Körtiden för verktyget estimeras till ~1 minut, beroende på datorkraft och status på installerade krävda R-paket och LaTeX-paket.

## Metod
Ett dataset som innehåller data från undersökningen och har sammanställts av Region Uppsala matas in i skriptet. WSP har använt sig av paketet *bookdown* för att producera en PDF-fil i LaTeX-format med R markdown.

## Skriptanvändning

### Krav på datat
Indata från kollektivtrafiksbarometern tillhandahålls av Region Uppsala så att de är i lämplig format. Detta försäkras med Regionens egna skript och beskrivs alltså inte här. Indatat ligger i tre filer, *rvu.csv*, *pers.csv*, och *avst.csv*.

Skriptet anropar SCBs API verktyg för att hämta befolkningsdata i de undersökta områdena.

Det krävs dessutom en shapefil, antingen med DESO-områden eller med kommuner, beroende på vilken geografisk aggreggering rapporten ska tas fram för. Detta används för att skapa en karta med den geografiska indelningen av det undersökta området. Om rapporten delas upp på DESO-områden, bör ett geopaket finnas som en .gpkg-fil (DeSO_2018_v2.gpkg). Om rapporten delas upp på kommunnivå, bör kartfiler finnas i en mapp med .shp-fil.

### Krav på mappstruktur
De tre .csv indatafilerna kan ligga antingen i samma mapp som skriptet eller i en annan mapp, men adressen behöver vara känd (det ska anges i skriptets förutsättningar).

Om rapporten gäller en tätort eller kommun behöver DESO-shapefilen ligga in undermapp i indatamappen. Undermappen ska heta *deso_2018_v2* och ska innehålla den .gpkg-filen som beskrivits ovan. Det enklaste är att ladda ner den zippade filen från SCB^[https://www.scb.se/vara-tjanster/oppna-data/oppna-geodata/] och packa upp den i indatamappen.

Om rapporten gäller en region behöver kommun-shapefilen ligga i en undermapp till indatamappen. Undermappen ska heta *KommunSweref99TM* och innehålla .shp-filen *Kommun_Sweref99TM_region.shp*. Det enklaste är att ladda ner den zippade filen från SCB^[https://www.scb.se/hitta-statistik/regional-statistik-och-kartor/regionala-indelningar/digitala-granser/] och packa upp mappen *KommunSweref99TM.zip* i skriptets mapp.

Resultatet av skriptet blir en PDF eller HTML-fil i samma mapp som skriptet.

### Krav på paket
Verktyget har testats med den senaste (sept 2021) versionen av R, dvs 4.1.1.

Alla krävda paket installeras och laddas under skriptets exekvering. Här kommer en lista med dessa paket samt dess version som verktyget har testats med.

* *knitr* - v1.34
* *kableExtra* - v1.3.4
* *tidyverse* - v1.3.1
* *readxl* - v1.3.1
* *bookdown* - v0.24
* *pxweb* - v0.11.0
* *scales* - v1.1.1
* *string* - v1.7.4
* *leaflet* - v2.0.4.1
* *sf* - v1.0-2
* *mapview* - v2.10.0
* *processx* - v3.5.2
* *tinytex* - v0.33
* *jsonlite* - v1.7.2
* *htmltools* - v0.5.2
* *webshot* - v0.5.2
* *farver* - v2.1.0

Under exekveringen kommer de paket som inte är installerade på datorn att installeras. I det här fallet installeras den senaste versionen av de olika paketen. Vi (och utvecklarna av R samt RStudio) rekommenderar att använda den senaste versionen av R-paketen. Vi har använt oss av de mest populära och välskötta paketen, vilket minskar felrisken vid nya uppdateringar. Vi kan dock inte utesluta att något fel kan komma i framtiden med uppdateringar av paketen ovan. Därför hänvisar vi till de fungerande paketversionerna ovan för att underlätta eventuell felsökning.

### Krav på indata i skriptet
I skriptet finns det några variabler som behöver specificeras innan körningen och möjliggör att skriptet kan användas i olika sammanhang och geografier samt att dataanalysen kan göras med olika förutsättningar. Alla dessa specificeringar görs i den del av koden som ligger i den R chunk som heter *indata*. Det ligger precis efter kodens *preamble* som används av LaTeX för att specificera dokumentets format.

* *rapportomrade*: en vektor av en eller flera strängar som definierar rapportens intresseområde. Detta används i dataanalysen, t.ex. i filtreringar av datat. T.ex. "Uppsala län" eller c("Bålsta", "Bålsta Skärplinge")
* *rapportomrade_text*: en sträng som används i löptext när man refererar till rapportens intresseområde. T.ex. "Bålsta"
* *rapportomrade_rubrik*: en sträng som används i rubriker (t.ex. i rapportens titel) när man refererar till rapportens intresseområde. T.ex. "Bålsta tätort"
* *rapportniva*: en sträng som definierar rapportens geografiska avgränsningen. Tre möjliga värden finns, *tatort*, *kommun*, eller *lan*.
* *omradesniva*: en sträng som definierar den geografiska indelningen som resultatet ska redovisas på. Två möjliga värden finns, *deso* eller *kommun*. Om rapportområdet är en region, bör *kommun* väljas och om det är en kommun eller tätort bör *deso* väljas.
* *starttid*: ett 6-siffrigt heltal som definierar starttiden för filtreringen av data, i format YYYYMM
* *sluttid*: ett 6-siffrigt heltal som definierar sluttiden för filtreringen av data, i format YYYYMM
* *palett*: en vektor av sträng med HTML-färgkoder av olika färger som ska användas i diagramm. Vektorn behöver innehålla minst sex färger. T.ex. Region Uppsalas färgskala: c("#5144d3","#53a17f","#3599d5","#e6ad00","#d57667","#9e1863")
* *table_api_address*: en sträng som definierar länk till SCB-tabell som ska användas för befolkningsmängd. "http://api.scb.se/OV0104/v1/doris/en/ssd/BE/BE0101/BE0101Y/FolkmDesoAldKonN" bör användas, då inläsningen av befolkningsmängden är hårdkodad för denna API.
* *sokvag*: en sträng som definierar sökvägen till csv-indatafilerna. Lämnas strängen tom, används skriptets mapp som sökväg
* *lagt_underlag*: en siffra som fungerar som gräns till vad som ska anses som lågt antal respondenter i dataanalysen. Alla resultat som baseras på antal respondenter under denna gräns kommer att markeras i rapporten.

### Att köra verktyget
För att köra verktyget krävs det att indatat uppdateras. Sedan ska användaren trycka på *Knit* i R-Studio. Som standard exporteras en PDF rapport. Det är dock också möjligt att välja HTML som format.
