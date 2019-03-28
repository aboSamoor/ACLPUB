# Generating proceedings for the ACL Anthology

## 0a. Clone this repository

```
git clone https://github.com/acl-org/ACLPUB
```

The scripts you will be using are in the subdirectory
`ACLPUB/anthologize`. Below, we assume that you have this directory in
your `PATH`; if not, you'll need to provide full pathnames to scripts.

## 0b. Install dependencies

You'll need Python >=3.5 and the Python packages `latexcodec` and
`pybtex`; these can usually be installed by running:

```
cd ACLPUB/anthologize
pip install -r requirements.txt
```

## 1. Create `acronyms_list`

Create a file that contains a list of the START names of all
tracks/workshops associated with the conference, one per line. You can
call this file whatever you want, but below, we assume that it is
called `acronyms_list`. For example, the `acronyms_list` for NAACL
2018 included:

```
naacl2018-longpapers
naacl2018-shortpapers
SemEval-2018
starsem18
```

(Note to users of previous versions: You no longer need to include the
volume id and number. You can, but they will be ignored.)

## 2. Download all proceedings from START

```
download-proceedings.sh <conference> acronyms_list
```
where `<conference>` is replaced by the START name of the conference
(found in the URL of its START page).

This downloads each track/workshop's proceedings. The result should
include the following directories and files (using the above subset of
NAACL 2018 as an example):

```
data/
  naacl2018-longpapers/
    proceedings/
      meta
      cdrom/
  naacl2018-shortpapers/
    proceedings/
      meta
      cdrom/
  SemEval-2018/
    proceedings/
      meta
      cdrom/
  starsem18/
    proceedings/
      meta
      cdrom/
```

Each `meta` file is just a collection of key/value pairs, one per
line, with the key and value separated by whitespace. The lines of
interest are (using SemEval-2018 as an example):

```
abbrev semeval
year 2018
bib_url http://www.aclweb.org/anthology/S18-1%03d
```

These have all been set by publications and book chairs in START
(Publication Console -> ACLPUB -> CDROM). (The abbreviation has
deliberately been altered to make it clear that it may be different
from both its START name (`SemEval-2018`) and its Anthology ID
(`S18-1`).)

Do not edit any of the fields except for `bib_url`. You _can_ edit
`bib_url` (for example, if an Anthology ID changed or is incorrect).

## 3. Convert to Anthology format

Just run:
```
make-anthology.sh
```
(Note to users of previous versions: You no longer need to provide the
`acronyms_list`; you can, but it will be ignored.)

This script does two jobs.

### 3a. Create and populate Anthology directories

First, it runs `anthologize.pl` to create the Anthology directory and
populate it with symlinks to the proceedings downloaded from START.

The `proceedings.tgz` downloaded from START has the following layout
when unpacked (again using SemEval 2018 as an example):

```
proceedings/
  meta                          Information about the conference
  cdrom/
    semeval-2018.bib            BibTeX file for proceedings and all papers
    semeval-2018.pdf            PDF of whole proceedings
    additional/
      semeval1001_Software.tgz  Software attached to paper 1001
      semeval1003_Dataset.zip   Dataset attached to paper 1003
      semeval1003_Note.pdf      Note attached to paper 1003
    bib/
      S18-1000.bib              BibTeX entry for whole proceedings
      S18-1001.bib              BibTeX entry for paper 1001
      S18-1002.bib               etc.
      S18-1003.bib
    pdf/
      S18-1000.pdf              PDF of front matter
      S18-1001.pdf              PDF for paper 1001
      S18-1002.pdf               etc.
      S18-1003.pdf
```

The anthology directory must have the following layout:

```
anthology/
  S/
    S18/
      S18.xml                   Metadata for proceedings and all papers
      S18-1.bib                 BibTeX file for proceedings and all papers
      S18-1.pdf                 PDF of whole proceedings
      S18-1000.bib              BibTeX entry for whole proceedings
      S18-1000.pdf              PDF of front matter
      S18-1001.bib               etc.
      S18-1001.pdf
      S18-1001.Software.tgz
      S18-1002.bib
      S18-1002.pdf
      S18-1003.bib
      S18-1003.pdf
      S18-1003.Dataset.zip
      S18-1003.Note.pdf
```

If you need to run this step manually, the usage is `anthologize.pl
data/<name>/proceedings anthology`, where `<name>` is the START name
of the track/workshop to process.

### 3b. Generate XML files

Next, `make-anthology.sh` runs `anthology_xml.py` to generate the XML
files that the Anthology uses to store all metadata and pointers to
papers and their attachments. One is generated for each venue+year; in
our example, there would be `anthology/N18.xml` for N18-1 and N18-2,
and `anthology/S18.xml` for S18-1 and S18-2.

The XML file lists attachments under several different fields; the
most general one looks like

```
  <attachment type="software">S18-1001.Software.tgz</attachment>
```

`anthology_xml.py` will generate these fields automatically by
looking at the attachment filename.

If you need to run this step manually, the usage is `anthology_xml.py
anthology/<x>/<x><yy> -o anthology/<x>/<x><yy>/<x><yy>.xml`, where
`<x>` is the one-letter code of the conference (e.g., `N` for NAACL)
and `<yy>` is the two-digit year.

## 4. Package and send the Anthology directory

```
tar czhvf <conference>_anthology.tgz anthology
```