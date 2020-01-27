# Finding adverse events with the FAERS API
### By: Jaclyn Jeffrey-Wilensky (jaclyn@spectrumnews.org)


This GitHub repo contains the Jupyter Notebook I used to find adverse events in autistic children prescribed aripiprazole for the Spectrum story [Hed TK](https://www.spectrumnews.org). In this document, I'll describe how you can use this notebook for your own purposes. I'll also lay out the steps I took to go from the raw FAERS data to the visualization that you see in the story.

These steps are written for relative beginners like me; if you're a more advanced user, you can skip straight to [Adapting the notebook](https://github.com/jjw-spectrum/abilify-aes/blob/master/README.md#adapting-the-notebook).

## About FAERS
[FAERS](https://open.fda.gov/data/faers/) is the FDA's Adverse Events Reporting System. It's essentially a huge database of every bad reaction to a drug ever reported to the FDA.

The data in FAERS is not comprehensive or FDA-verified, and it can't be used on its own to, say, compare two drugs and say which causes more bad reactions. But if used correctly, it can be a useful complement to the rest of your reporting. Other examples of FAERS data used to great effect include [this](https://www.jsonline.com/story/news/investigations/2019/05/30/arthritis-psoriasis-drugs-darker-aspect-34-000-reports-deaths/1206103001/) *Milwaukee Journal-Sentinel* story on biologics and the *Palm Beach Post*'s [reporting](https://www.palmbeachpost.com/news/20180404/how-post-unearthed-local-roots-of-insys-story) on the opioid medication Subsys.

The other thing to know about FAERS is that it's a huge mess! Duplicates run rampant, records are often incomplete, and drug names are often inconsistent or misspelled. For example, here are just a few of the terms I used to capture the aripiprazole records:
- aripiprazol (left off the "e" to include Italian and other names for the drug)
- ariprazol (a common misspelling)
- abilify (the drug's brand name)
- aristada (the drug's brand name in the UK)
Even when the information is there, it's often scattered across a variety of fields, including

FAERS is accessible through a [public dashboard](https://www.fda.gov/drugs/questions-and-answers-fdas-adverse-event-reporting-system-faers/fda-adverse-event-reporting-system-faers-public-dashboard), which is great for research purposes. But because of all the weirdness described above, I recommend either querying the openFDA [drug adverse event API](https://open.fda.gov/apis/drug/event/) *or* downloading, cleaning and collating the [quarterly data files](https://fis.fda.gov/extensions/FPD-QDE-FAERS/FPD-QDE-FAERS.html) provided by the FDA. Because my query was so specific and ultimately only corresponded to a tiny slice of the jillions of records available, I opted for the former method, which I'll describe below.

## How I did it

### Building the API query

The openFDA drug adverse event API is an easy way to get specific slices of data from FAERS without doing an enormous amount of cleaning, sorting and searching all by yourself. But because the data is so incomplete and inconsistent, I kept my initial search fairly broad and narrowed it down myself.

That's what my script does- it queries the API for aripiprazole-related adverse events from each year, then winnows the 80K+ records down to fewer than 400 that were relevant to the investigation. Ahead of time, I decided that my criteria for what reactions to include were as follows:
1. The event occurred in a child between the ages of 3-17.
2. Aripiprazole is suspected to have caused the event.
3. The aripiprazole was prescribed to treat autism.

But first, I had to get all the records where aripiprazole was listed. I tried out a number of queries that searched various nested fields for all the name variations I could think of. A few attempts, and the number of records they retrieved, are listed below:

- requests.get('https://api.fda.gov/drug/event.json?search=patient.drug.medicinalproduct:"aripiprazole"&limit=100')
  Total: 15936
- requests.get('https://api.fda.gov/drug/event.json?search=(patient.drug.medicinalproduct:"aripiprazole")+OR+(patient.drug.medicinalproduct:"abilify")&limit=100')
  Total: 78723
- response = requests.get('https://api.fda.gov/drug/event.json?api_key=QO1trbbs0hp1HnDlpauIWNjXcJ96LAdZfzBZgixY&search=(patient.drug.medicinalproduct:"aripiprazole")+(patient.drug.medicinalproduct:"abilify")+(patient.drug.activesubstance.activesubstancename:"aripiprazole")+(patient.drug.drugauthorizationnumb:"021436")&limit=100')
  Total: 80719

By comparison, a search of FAERS public dashboard yielded just 60K results for a search of "aripiprazole." So you can see why it's better to get the data yourself when you're looking at FAERS for reporting purposes.

Ultimately, because different records stored drug names in different fields, (medicinalproduct, activesubstancename, etc.) I decided to search each record in its entirety for a list of aripiprazole name variants, yielding 80915 results total. See blocks 2 and 3 of the notebook for the exact query and parameters I used.

### Collecting 81K records

There are some limits on how much data you can collect from the adverse events API at once. Here are some tricks I used to get around them.

The API uses two parameters to page through results: "limit," which controls how many results are returned at once, and "skip," which controls how many records you pass by before you collect the next batch. There's a max limit of 100 and a max skip of 25K. To get around the skip limit, I batched my results by year. Within each year, I simply incremented "skip" by 100 each time I queried the API until I'd collected all the records for that year.

See block 4 of the notebook for more detail on how I iterated through the "pages" of results.

### Cleaning the data

Another quirk about FAERS data—while most adverse event reports include the patient's age in years, others give that info in weeks, months, or even decades. Others have no age info whatsoever. So I stripped out 32310 records with nothing in the 'patientonsetage' field. For the rest, I changed the contents of 'patientonsetage' based on the contents of the 'patientonsetageunit' field, which has a code for each possible age unit. See blocks 7 and 8 for details.

### Filtering the data

Once the ages were standardized, I made a new list that included only records for patients who were between 3 and 17 years old at the time the reactions occurred—6750 in total. I also caught 53 stragglers with "None" in the 'patientonsetageunit' field, which means the cleaning method described above should be improved.









### Data analysis

## How you can do it

### Getting started with Jupyter Notebooks
*Note: This setup is almost identical to what I learned in [Lam Thuy Vo](https://github.com/lamthuyvo)'s data journalism class at the CUNY Graduate School of Journalism. All credit for it goes to her*

The first step is to get set up with Jupyter inside a virtual environment.

Once you've downloaded the notebook, put it in its own folder. In the Terminal, navigate to the new folder (in my case, abilify-aes) and set up a virtual environment using the following command:

`python3 -m venv venv`

Whenever you want to play with your notebook, you'll activate the virtual environment using the following command:

`source venv/bin/activate`

Now, you'll want to install Jupyter within your virtual environment. I use [pip](https://pip.pypa.io/en/stable/) to install Python packages, so I run the following command:

`pip install jupyter`

To open Jupyter notebooks, run:

`jupyter notebook`

### Getting the notebook running

You'll need to make a few modifications to the notebook to get it working on your computer.

First, you'll need your own key for querying the openFDA API. You can get that [here](https://open.fda.gov/apis/authentication/). Enter the key in the `parameters` dictionary defined in lines 3-7 of the second block of code in the notebook.

You'll also need to modify the filepaths throughout the script. These can be found in blocks 5, 6, and 17.

### Customizing your query and filtering steps
