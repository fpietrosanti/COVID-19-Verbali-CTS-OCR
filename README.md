# OCR processing relazioni Comitato Tecnico Scientifico Covid del Governo Italiano

I verbale del CTS pubblicati su https://github.com/pcm-dpc/COVID-19-Verbali-CTS sono delle immagini, quindi non sono accessibili.

Non è possibile cercare stringhe di testo, fare copia e incolla, né i suoi contenuti sono indicizzati sui motori di ricerca.

Di seguito uno sforzo di un paio d'ore, sarebbe ideale se qualcuno lo automatizzasse o PDCM stessa pubblicasse in modo accessibile.

Sono accessibili i PDF con testo ricercabile su http://verbalictscovid.infosecurity.ch/ (alle 21.48 del 5 Settembre sta processando la cartella 2020-03) .

In seguito le istruzioni per replicare il setup, magari automatizzarlo sui nuovi file, indicizzandoli, etc

Da leggere l'incredibile storia della battaglia legale della Fondazione Einaudi per richiederne la pubblicazione tramite l'uso del FOIA https://www.fondazioneluigieinaudi.it/ecco-la-storia-dellaccesso-agli-atti-del-comitato-tecnico-scientifico/ 

## Installare una Debian Buster

Io ho preso un VPS da 5 euro al mese con una Debian Buster da scaleway.com

## Scaricare i verbali

```
mkdir /data/
cd /data
apt-get install git
git clone https://github.com/pcm-dpc/COVID-19-Verbali-CTS.git
cd COVID-19-Verbali-CTS
```

## Verificare che non ci sia testo con pdfextract

```
apt-get install poppler-utils
pdftotext 2020-02/covid-19-cts-verbale-001-20200207.pdf
```

## Analizzare i PDF con pdfmetadata

```
apt-get install origami-pdf
pdfmetadata 2020-02/covid-19-cts-verbale-001-20200207.pdf
```

```
[*] Document information dictionary:
Author              : NAPS2
CreationDate        : D:20200902104601+02'00
Creator             : NAPS2
ModDate             : D:20200902104639+02'00
Producer            : PDFsharp 1.50.4589 (www.pdfsharp.com)
Subject             : Immagine scansionata
Title               : Immagine scansionata

[*] Metadata stream:
ModifyDate          : 2020-09-02T10:46:39+02:00
CreateDate          : 2020-09-02T10:46:01+02:00
MetadataDate        : 2020-09-02T10:46:39+02:00
CreatorTool         : NAPS2
format              : application/pdf
title               : Immagine scansionata
description         : Immagine scansionata
creator             : NAPS2
DocumentID          : uuid:92f2617f-4e8a-4291-8f9e-8d276ba3585f
InstanceID          : uuid:deb62522-a30a-43d3-ad26-aaa34af619cd
Producer            : PDFsharp 1.50.4589 (www.pdfsharp.com)
```

## Installare OCRmyPDF, che usa l'opensource OCR Engine Tesseract e language pack italiano

Oltre a tesseract (usato da ocrmypdf) ci sono anche altri OCR Engine opensource da provare (kraken, Ocrad, calamari) e volendo anche soluzioni Cloud (caricare i PDF su Google Drive e quindi cliccare "apri con google docs" e quindi condividere il file su cui Google avrà fatto OCR automaticamente) oltre ovviamente a soluzioni commerciali (abby fine reader fra i migliori per i documenti).

Per semplicità del PoC da completare prima di cena userò OCRmyPDF https://github.com/jbarlow83/OCRmyPDF

```
apt-get install ocrmypdf tesseract-ocr-ita
```

## Eseguire su tutti i file pdf l'OCR ocrmypdf basato su tesseract

```
find 2020-0* -printf '%p' -name '*.pdf' -exec ocrmypdf --force-ocr -l ita '{}' '{}' \;
```

# NOTE

Gli OCR opensource lanciati così alla buona spesso non si comportano una meraviglia, un batch processing con un OCR commerciale potrebbe essere anche interessante.

Il tricks d'usare Google Drive e uno a uno a mano convertirli in Google Docs, per poi pubblicarli, non sarebbe affatto male da fare (ma non ho tempo, se qualcuno volesse)

Sarebbe bello fare anche un bel pdf2html per vedere cosa esce fuori.

## Rendi accessibili i PDF con OCR con un webserver al volo
Installiamo lighthttpd che è un webserver piccino

```
apt-get install lighttpd
```

Cambiamogli la webroot alla directory con i PDF del repo Git PDCM e abilitiamogli il directory listing

```
vim /etc/lighttpd/lighttpd.conf
```

```
server.document-root        = "/data/COVID-19-Verbali-CTS"
server.dir-listing = "enable"
```

Riavviamo il webserver
```
 /etc/init.d/lighttpd restart
 ```

E ora sulla porta 80 c'è la directory /data/COVID-19-Verbali-CTS accessibile per condividere

## Creiamo un indice HTML per aiutare i motori di ricerca

Cambiamo l'url di download dei pdf dal file elenco-verbali.md in markdown nel file  elenco-verbali-ocr.md 

 ```
sed -e 's?https://raw.githubusercontent.com/pcm-dpc/COVID-19-Verbali-CTS/master/?http://51.15.85.131/?' -e 's?COVID-19 Verbali CTS?COVID-19 Verbali CTS con PDF con testo?'  elenco-verbali.md > elenco-verbali-ocr.md
 ```

Traduciamo i lmarkdown nell'index.html
 ```
apt-get install git
markdown elenco-verbali-ocr.md > index.html

 ```

## Crea Google Custom Search Engine

Attivato Google CSE, per successivo crawling da google dei PDF con già OCR.

https://cse.google.com/cse?cx=c0bbeb66c5e5e986a

## Ma quanto testo c'è nei PDF originali?

Nella maggioranza dei PDF pubblicati l'accessibilità testuale è scarsina, c'è poco testo ricercabile.

Per renderci conto di quanto testo c'è in ogni pdf, lo si può verificare in modo veloce rapportando il numero di linee di testo per PDF rispetto alla dimensione dei file:

 ```

 for file in `find 2020-* -name \*pdf` ; do  LINEE=`pdftotext $file - | sort -rn | uniq | wc -l` ; echo -n "Linee di testo: $LINEE - Dimensione: " ; du -sh $file ; done

 ```

Con questo output:
 ```

Linee di testo: 1 - Dimensione: 408K	2020-02/covid-19-cts-verbale-009-20200226.pdf
Linee di testo: 1 - Dimensione: 160K	2020-02/covid-19-cts-verbale-005-20200218.pdf
Linee di testo: 4 - Dimensione: 52K	2020-02/covid-19-cts-verbale-002-20200210.pdf
Linee di testo: 1 - Dimensione: 460K	2020-02/covid-19-cts-verbale-007-20200222.pdf
Linee di testo: 1 - Dimensione: 2.2M	2020-02/covid-19-cts-verbale-013-20200229.pdf
Linee di testo: 130 - Dimensione: 1.3M	2020-02/covid-19-cts-verbale-011-20200227.pdf
Linee di testo: 1 - Dimensione: 216K	2020-02/covid-19-cts-verbale-001-20200207.pdf
Linee di testo: 8 - Dimensione: 196K	2020-02/covid-19-cts-verbale-010-20200227.pdf
Linee di testo: 1 - Dimensione: 168K	2020-02/covid-19-cts-verbale-006-20200221.pdf
Linee di testo: 1 - Dimensione: 952K	2020-02/covid-19-cts-verbale-003-20200212.pdf
Linee di testo: 1 - Dimensione: 300K	2020-02/covid-19-cts-verbale-004-20200214.pdf
Linee di testo: 1 - Dimensione: 804K	2020-02/covid-19-cts-verbale-008-20200224.pdf
Linee di testo: 1 - Dimensione: 720K	2020-02/covid-19-cts-verbale-012-20200228.pdf
Linee di testo: 1 - Dimensione: 2.4M	2020-03/covid-19-cts-verbale-017-20200304.pdf
Linee di testo: 3 - Dimensione: 25M	2020-03/covid-19-cts-verbale-030-20200317.pdf
Linee di testo: 1 - Dimensione: 1.9M	2020-03/covid-19-cts-verbale-016-20200303.pdf
Linee di testo: 4 - Dimensione: 1.5M	2020-03/covid-19-cts-verbale-033-20200320.pdf
Linee di testo: 3 - Dimensione: 11M	2020-03/covid-19-cts-verbale-027-20200314.pdf
Linee di testo: 3 - Dimensione: 16M	2020-03/covid-19-cts-verbale-023-20200310.pdf
Linee di testo: 3 - Dimensione: 13M	2020-03/covid-19-cts-verbale-021-20200307.pdf
Linee di testo: 1 - Dimensione: 280K	2020-03/covid-19-cts-verbale-018-20200304.pdf
Linee di testo: 1 - Dimensione: 5.1M	2020-03/covid-19-cts-verbale-019-20200305.pdf
Linee di testo: 33 - Dimensione: 5.7M	2020-03/covid-19-cts-verbale-039-20200330.pdf
Linee di testo: 1 - Dimensione: 4.6M	2020-03/covid-19-cts-verbale-022-20200309.pdf
Linee di testo: 3 - Dimensione: 3.4M	2020-03/covid-19-cts-verbale-029-20200316.pdf
Linee di testo: 3 - Dimensione: 32M	2020-03/covid-19-cts-verbale-032-20200319.pdf
Linee di testo: 1 - Dimensione: 2.7M	2020-03/covid-19-cts-verbale-026-20200313.pdf
Linee di testo: 8 - Dimensione: 5.2M	2020-03/covid-19-cts-verbale-040-20200331.pdf
Linee di testo: 717 - Dimensione: 2.6M	2020-03/covid-19-cts-verbale-037-20200326.pdf
Linee di testo: 1 - Dimensione: 3.0M	2020-03/covid-19-cts-verbale-020-20200306.pdf
Linee di testo: 8 - Dimensione: 3.7M	2020-03/covid-19-cts-verbale-034-034bis-20200321.pdf
Linee di testo: 7 - Dimensione: 3.2M	2020-03/covid-19-cts-verbale-035-20200324.pdf
Linee di testo: 1 - Dimensione: 1008K	2020-03/covid-19-cts-verbale-015-20200302.pdf
Linee di testo: 3 - Dimensione: 6.5M	2020-03/covid-19-cts-verbale-031-20200318.pdf
Linee di testo: 3 - Dimensione: 7.6M	2020-03/covid-19-cts-verbale-028-20200315.pdf
Linee di testo: 6 - Dimensione: 120K	2020-03/covid-19-cts-verbale-014-20200301.pdf
Linee di testo: 512 - Dimensione: 3.2M	2020-03/covid-19-cts-verbale-036-20200325.pdf
Linee di testo: 1033 - Dimensione: 2.9M	2020-03/covid-19-cts-verbale-038-20200327.pdf
Linee di testo: 1 - Dimensione: 5.5M	2020-03/covid-19-cts-verbale-025-20200312.pdf
Linee di testo: 3 - Dimensione: 12M	2020-03/covid-19-cts-verbale-024-20200311.pdf
Linee di testo: 497 - Dimensione: 19M	2020-03/covid-19-cts-verbale-028-20200528.pdf
Linee di testo: 92 - Dimensione: 4.9M	2020-04/covid-19-cts-verbale-053-20200416.pdf
Linee di testo: 3467 - Dimensione: 13M	2020-04/covid-19-cts-verbale-057-20200422.pdf
Linee di testo: 2503 - Dimensione: 5.5M	2020-04/covid-19-cts-verbale-049-20200409.pdf
Linee di testo: 7 - Dimensione: 3.1M	2020-04/covid-19-cts-verbale-041-20200401.pdf
Linee di testo: 3 - Dimensione: 25M	2020-04/covid-19-cts-verbale-054-20200417.pdf
Linee di testo: 595 - Dimensione: 1.6M	2020-04/covid-19-cts-verbale-048-20200408.pdf
Linee di testo: 539 - Dimensione: 6.9M	2020-04/covid-19-cts-verbale-045-20200406.pdf
Linee di testo: 1 - Dimensione: 12M	2020-04/covid-19-cts-verbale-052-20200415.pdf
Linee di testo: 787 - Dimensione: 3.8M	2020-04/covid-19-cts-verbale-060-20200427.pdf
Linee di testo: 11 - Dimensione: 2.3M	2020-04/covid-19-cts-verbale-046-20200407.pdf
Linee di testo: 1076 - Dimensione: 8.7M	2020-04/covid-19-cts-verbale-043-20200403.pdf
Linee di testo: 83 - Dimensione: 3.2M	2020-04/covid-19-cts-verbale-044-20200404.pdf
Linee di testo: 1 - Dimensione: 37M	2020-04/covid-19-cts-verbale-056-20200420.pdf
Linee di testo: 9 - Dimensione: 4.3M	2020-04/covid-19-cts-verbale-058-20200423.pdf
Linee di testo: 214 - Dimensione: 5.1M	2020-04/covid-19-cts-verbale-063-20200430.pdf
Linee di testo: 8 - Dimensione: 1.8M	2020-04/covid-19-cts-verbale-042-20200402.pdf
Linee di testo: 4 - Dimensione: 11M	2020-04/covid-19-cts-verbale-055-20200418.pdf
Linee di testo: 230 - Dimensione: 8.5M	2020-04/covid-19-cts-verbale-051-20200414.pdf
Linee di testo: 4842 - Dimensione: 3.7M	2020-04/covid-19-cts-verbale-062-20200429.pdf
Linee di testo: 1757 - Dimensione: 6.4M	2020-04/covid-19-cts-verbale-050-20200411.pdf
Linee di testo: 1 - Dimensione: 4.2M	2020-04/covid-19-cts-verbale-047-20200407.pdf
Linee di testo: 1835 - Dimensione: 12M	2020-04/covid-19-cts-verbale-059-2020042425.pdf
Linee di testo: 2580 - Dimensione: 7.5M	2020-04/covid-19-cts-verbale-061-20200428.pdf
Linee di testo: 1766 - Dimensione: 8.9M	2020-05/covid-19-cts-verbale-083-20200529.pdf
Linee di testo: 3 - Dimensione: 4.6M	2020-05/covid-19-cts-verbale-069-20200511.pdf
Linee di testo: 1475 - Dimensione: 4.7M	2020-05/covid-19-cts-verbale-072-20200513.pdf
Linee di testo: 122 - Dimensione: 5.4M	2020-05/covid-19-cts-verbale-064-20200502.pdf
Linee di testo: 497 - Dimensione: 19M	2020-05/covid-19-cts-verbale-082-20200528.pdf
Linee di testo: 3 - Dimensione: 22M	2020-05/covid-19-cts-verbale-076-20200518.pdf
Linee di testo: 258 - Dimensione: 8.8M	2020-05/covid-19-cts-verbale-080-20200525.pdf
Linee di testo: 8 - Dimensione: 4.5M	2020-05/covid-19-cts-verbale-070-20200511.pdf
Linee di testo: 1635 - Dimensione: 14M	2020-05/covid-19-cts-verbale-075-20200516.pdf
Linee di testo: 1141 - Dimensione: 7.4M	2020-05/covid-19-cts-verbale-071-20200512.pdf
Linee di testo: 1818 - Dimensione: 6.7M	2020-05/covid-19-cts-verbale-073-20200514.pdf
Linee di testo: 1097 - Dimensione: 11M	2020-05/covid-19-cts-verbale-068-2020050810.pdf
Linee di testo: 3 - Dimensione: 2.3M	2020-05/covid-19-cts-verbale-081-20200526.pdf
Linee di testo: 3 - Dimensione: 9.7M	2020-05/covid-19-cts-verbale-077-20200519.pdf
Linee di testo: 1425 - Dimensione: 17M	2020-05/covid-19-cts-verbale-065-20200503.pdf
Linee di testo: 1435 - Dimensione: 9.8M	2020-05/covid-19-cts-verbale-066-202005040506.pdf
Linee di testo: 332 - Dimensione: 2.9M	2020-05/covid-19-cts-verbale-067-20200507.pdf
Linee di testo: 3 - Dimensione: 8.3M	2020-05/covid-19-cts-verbale-079-20200522.pdf
Linee di testo: 4 - Dimensione: 9.8M	2020-05/covid-19-cts-verbale-078-20200521.pdf
Linee di testo: 3240 - Dimensione: 13M	2020-05/covid-19-cts-verbale-074-20200515.pdf
Linee di testo: 71 - Dimensione: 7.3M	2020-06/covid-19-cts-verbale-089-20200616.pdf
Linee di testo: 11 - Dimensione: 16M	2020-06/covid-19-cts-verbale-088-20200612.pdf
Linee di testo: 11 - Dimensione: 2.9M	2020-06/covid-19-cts-verbale-085-20200603.pdf
Linee di testo: 53 - Dimensione: 18M	2020-06/covid-19-cts-verbale-090-20200622.pdf
Linee di testo: 174 - Dimensione: 7.4M	2020-06/covid-19-cts-verbale-084-20200603.pdf
Linee di testo: 59 - Dimensione: 7.4M	2020-06/covid-19-cts-verbale-086-20200605.pdf
Linee di testo: 3 - Dimensione: 9.9M	2020-06/covid-19-cts-verbale-091-20200623.pdf
Linee di testo: 63 - Dimensione: 3.8M	2020-06/covid-19-cts-verbale-087-20200608.pdf
Linee di testo: 4377 - Dimensione: 15M	2020-07/covid-19-cts-verbale-092-2020070102.pdf
Linee di testo: 107 - Dimensione: 7.0M	2020-07/covid-19-cts-verbale-093-20200703.pdf
Linee di testo: 303 - Dimensione: 7.6M	2020-07/covid-19-cts-verbale-094-20200707.pdf
Linee di testo: 635 - Dimensione: 18M	2020-07/covid-19-cts-verbale-095-2020071620.pdf
 ```
