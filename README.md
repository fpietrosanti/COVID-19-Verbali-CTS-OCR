# OCR processing relazioni Comitato Tecnico Scientifico Covid del Governo Italiano

I verbale del CTS pubblicati su https://github.com/pcm-dpc/COVID-19-Verbali-CTS sono delle immagini, quindi non sono accessibili.

Non è possibile cercare stringhe di testo, fare copia e incolla, né i suoi contenuti sono indicizzati sui motori di ricerca.

Di seguito uno sforzo di un paio d'ore, sarebbe ideale se qualcuno lo automatizzasse o PDCM stessa pubblicasse in modo accessibile.

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
Solo alcuni dei PDF parrebbero contenere del testo, sarebbe da fare elenco analitico.

Gli OCR opensource lanciati così alla buona spesso non si comportano una meraviglia, un batch processing con un OCR commerciale potrebbe essere ideale.

Il tricks d'usare Google Drive e uno a uno a mano convertirli in Google Docs, per poi pubblicarli, non sarebbe affatto male da fare (ma non ho tempo, se qualcuno volesse)
