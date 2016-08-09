## README
### Project Description

This project examines the timeline for going from submitting your first new building job application to getting your
first new building construction permit to getting your first new building certificate of occupancy (in New York City).
How long does that all take?

New York City's Department of Buildings (DOB) distributes (via the [Open Data Portal](https://nycopendata.socrata.com/))
 a pair of datasets which inform this project: the [job applications dataset](https://data.cityofnewyork.us/Housing-Development/DOB-Job-Application-Filings/ic3t-wcy2),
 which contains all construction job applications filed with the city, and the [permit issuance dataset](https://data.cityofnewyork.us/Housing-Development/DOB-Permit-Issuance/ipu4-2q9a),
 which contains all of the ultimate construction permit approvals and denials issued by the DOB.

Usually multiple construction jobs must be submitted as a part of a complex construction bid&mdash;and constructing
an entirely new building basically invariably falls under the label "complex". So the first thing this project tries
to tackle is examining new building construction jobs from the first dataset and seeing how often we can match them
with permits from the second.

Although this is quite possible to do, because of the time-series limitations of the data (they both go back only to 2013)
it turns out that generating a statistically meaningful sample out of this data is difficult. Consider this: suppose
that a building received its permit in 2015, but new building job applications for that building go back three years
further, to 2012. We can capture the first of these that appears in the dataset, so we will know that the process is
*ongoing*, but since we can't capture that first one from 2012&mdash;the data just doesn't go back that far&mdash;our
 algorithm will merely report that first one that we *do* have, from 2013, as the first job, even though it's not.

We can somewhat hedge around this by taking only very recent permits; this will maximize the time period in which
these permits may appear. There are nevertheless two problems with this. One is that we don't *really* know how
effective that is in fixing this problem&mdash;without ground truths this really is an unknown unknown. The second is
that we have the same problem trying to understand the permit-to-occupancy process! We can't combine these two
metrics together meaningfully because we are squeezed on both sides: an observation of a permit in early 2015, for
example, would only have a year and a half to result in a certificate of occupancy and two years in which it had
gotten its first job application.

Resolving this problem resolutely would require lots more data (but hey, if you're reading this in 2020, you're in
luck...). In the meantime we can look at these two processes seperately. Since the latter process is more
interesting, I chose to focus on that. That is, while the code for building a application-to-permit timing dataset
exists, as demarcated below, I haven't run any analytics on it (at least none that I'm confident in).

In the second case, the  we are fuzzy matching permits with certificates of occupancy. In this case we invoke
[BISweb](http://a810-bisweb.nyc.gov/bisweb/bsqpm01.jsp), DOB's impressive albeit byzantine 2001-era mainframe
application that is the public system of record for all (physical) things buildings, including their certificates of
occupancy, the bits of paper a new building gets when it finishes construction that demonstrates that it is
certifiably safe for inhabitation.

(What the heck does a certificate of occupancy look like? I'm glad you asked! [I put together an example set you can parooze through](https://github.com/ResidentMario/nyc-certificates-of-occupancy) in a seperate repository.

However, this data is locked up in PDF form, requiring extraction of data where it effectively literally didn't exist
before. This is not the first project which reads data from BISweb, as [a quick search on GitHub attests](https://github.com/search?q=bisweb&type=Code&utf8=%E2%9C%93),
but it is the first one, that I'm aware of, that reads in specifically occupancy data.

This implementation is the `co_reader` module in the `src` folder. For this project I threaded this module through
the notebooks responsible for generating my core dataset, but this module is importable and freely available for use
elsewhere too. Note, however, that it has (what I consider to be) very restrictive environmental requirements: see
the [Environment](#Environment) section for more details on why.

What I ultimately settled on was taking a sample of buildings which recieved permits between May 24 2013 and December
31 2014 and matching those with any certificates of occupancy that I could find for those buildings which are more
recent, in terms of date, than the permits themselves.

This means that, importantly, each record comes with a 18-to-36 month observational period, which the precise
observed period dependent on when the initial permit was filed. In statistical terms, this means that this dataset is
 a [censored](https://en.wikipedia.org/wiki/Censoring_(statistics)) [time-series](https://en.wikipedia.org/wiki/Time_series)
dataset, requiring certain interpretive care.

I also examined, more briefly, the trend in the daily/weekly volumes of permit responses by the DOB
(overall result: reasonably stable in the long term, reasonably volatile in the short).

For still more details read the memo.

### Contents

(Only those interesting from a high-level perspective.)

#### Notebooks

The analytic contents of this repository are broken up into three parts:

1. `notebooks/Permit-to-Occupancy Processing.ipynb` &mdash; This Jupyter notebook contains the code which builds
`permit_occupancy_join_sample.csv`. This dataset is a statistical sample of permit data and corresponding
first certificate of occupancy data for all buildings which received a new build construction permit in between May
24 2013 and December 31 2014.


2. `notebooks/Application-to-Permit Processing.ipynb` &mdash; This Jupyter notebook contains the code which builds
`application_permit_join_sample.csv`, the master file containing the permit data and corresponding job application
data for all buildings which recieved a new building construction permit in between January 1 2015 and June 24 2016.
3. `notebooks/Application-Permit-Occupancy Analysis.ipynb` &mdash; This Jupyter notebook contains the actual analytics
that I ran on the two datasets generated by the above.

#### Data

1. `permit_occupancy_join_sample.csv` &mdash; The master file containing the permit data and corresponding first
certificate of occupancy data for all buildings which received a new build construction permit in between May 24 2013 and December
31 2014.

    Contains columns encoding initial permit issuance data, initial certificate of occupancy issuance date, and the
    number of days in between the two observations (if applicable).

2. `application_permit_join_sample.csv`, the master file containing the permit data and corresponding job application
data for all buildings which recieved a new building construction permit in between January 1 2015 and June 24 2016.

    Contains columns encoding initial new building job application date, initial permit issuance date, and the number
     of days in between the two observations (if applicable).

#### Modules

`co_reader` (specifically `co_reader.get_co_date()`) is an implementation of a Certificate of Occupancy getter
which reads C-of-O data off of the New York City Department of Building's [BISweb](http://a810-bisweb.nyc.gov/bisweb/).

This is a necessary part of the process for `Permit-to-Occupancy Processing.ipynb`. The code is very well-commented
so you can read it yourself.

`wait_page.html` and `wait_page.png` in the `figures` folder are code from and a screenshot of, respectively, the
BISweb wait page (which require implementing nice waiting for).

`co_reader` has also been seperately released as `https://github.com/ResidentMario/co_reader`

### Environment

You will need a `Python 2.7` environment booted up with the following libraries:

    pip install requests
    pip install bs4
    pip install selenium
    pip install arrow
    pip install tqdm
    pip install pdfminer
    conda install pandas
    conda install jupyter
    conda install seaborn

**You can install everything all at once using the packaged `environment.yml` by running `conda env create`**.

Notably, `pdfminer` (using the utility executable `pdf2txt.py`) is used to scrape text from the Certificate of
Occupancy PDFs using a command like the following one:

    pdf2txt.py marshalls_2012_record.pdf

(You do not need to do so yourself manually; this is just how, internally, `co_reader` operates.)

It is `pdfminer` which imposes the restriction that this environment by `Python 2.7`. It is the only library
 in this stack which, as of mid-2016, is still Python 2 only.

To open the Jupyter notebooks, navigate to this repository on your computer and hit `jupyter notebook`.
