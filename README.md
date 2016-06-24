## README

### Environment

The environment necessary to get these scripts working is relatively complex.

You will need the `tesseract`, `ghostscript`, `poppler`, `imagemagick`, and `phantomjs` external libraries.

On Mac you can install all of these in one go using `brew`:

    brew install tesseract
    brew install ghostscript
    brew install poppler
    brew install imagemagick
    brew install phantomjs

Linux (`apt-get`) and Windows (`chocolatey`) has similar package manager tools. However, these scripts are untested
on those platforms.

In addition to these externals you will need a `Python 2.7` environment booted up with the following libraries:

    pip install requests
    pip install bs4
    pip install selenium
    pip install arrow
    pip install tqdm
    pip install pypdforc
    pip install pdfminer
    conda install pandas
    conda install jupyter
    conda install seaborn

Two of these which deserve particular attention are `pypdfocr`, which implements the OCR algorithm used to get around
 the file locking in the C of O documents, and `pdfminer`, which I use to actually mine the text of that PDF (using
 the utility executable `pdf2txt.py`). The signatures are:

    pypdfocr marshalls_2012_record.pdf
    pdf2txt.py pluto_datadictionary.pdf -o temp.html

Note that it is `pdfminer` which imposes the restriction that this environment by `Python 2.7`. It is the only library
 in this stack which, as of mid-2016, is still Python 2 only.

To open the Jupyter notebooks, navigate to this repository on your computer and hit `jupyter notebook`.

### Contents

Read these in order: `Permit Issuance.ipynb` > `...`.