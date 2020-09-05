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

Da https://github.com/jbarlow83/OCRmyPDF

```
apt-get install ocrmypdf tesseract-ocr-ita
```

## Eseguire su tutti i file pdf l'OCR ocrmypdf 

```
find 2020-0* -printf '%p' -name '*.pdf' -exec ocrmypdf --force-ocr -l ita '{}' '{}' \;
```
