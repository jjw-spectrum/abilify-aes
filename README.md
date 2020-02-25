# Finding adverse events with the FAERS API
### By: Jaclyn Jeffrey-Wilensky (jaclyn@spectrumnews.org)


This GitHub repo contains the Jupyter Notebook I used to find adverse events in autistic children who took aripiprazole. *Spectrum* used this data to create a chart for the story [How aripiprazole’s promise for treating autism fell short](https://www.spectrumnews.org/features/deep-dive/the-rise-of-aripiprazole/). Here, I'll describe how you can use this notebook for your own purposes. I'll also lay out the steps I took to go from the raw FAERS data to the visualization that you see in the story.

These steps are written for relative beginners like me; if you're already an API/FAERS wiz, you can skip straight to the [Getting the notebook running](https://github.com/jjw-spectrum/abilify-aes#getting-the-notebook-running) section.

## About FAERS
[FAERS](https://open.fda.gov/data/faers/) is the FDA's Adverse Events Reporting System. It's essentially a huge database of every bad reaction to a drug ever reported to the FDA.

The data in FAERS are not comprehensive or FDA-verified, and it can't be used on its own to, say, compare two drugs and say which causes more bad reactions. But if used correctly, it can be a useful complement to the rest of your reporting. Other examples of FAERS data used to great effect include [this](https://www.jsonline.com/story/news/investigations/2019/05/30/arthritis-psoriasis-drugs-darker-aspect-34-000-reports-deaths/1206103001/) *Milwaukee Journal-Sentinel* story on biologics and the *Palm Beach Post*'s [reporting](https://www.palmbeachpost.com/news/20180404/how-post-unearthed-local-roots-of-insys-story) on the opioid medication Subsys.

The other thing to know about FAERS is that it's a huge mess! Duplicates run rampant, records are often incomplete, and drug names are often inconsistent or misspelled.

FAERS is accessible through a [public dashboard](https://www.fda.gov/drugs/questions-and-answers-fdas-adverse-event-reporting-system-faers/fda-adverse-event-reporting-system-faers-public-dashboard), which is great for research purposes. But because of the messiness of the data, I recommend either querying the openFDA [drug adverse event API](https://open.fda.gov/apis/drug/event/) *or* downloading, cleaning and collating the [quarterly data files](https://fis.fda.gov/extensions/FPD-QDE-FAERS/FPD-QDE-FAERS.html) provided by the FDA. Because my query was so specific and ultimately only corresponded to a tiny slice of the jillions of records available, I opted for the former method, which I'll describe below.

## How I did it

### Building the API query

The openFDA drug adverse event API is an easy way to get specific slices of data from FAERS without doing an enormous amount of cleaning, sorting and searching all by yourself. It also takes care of duplicates for you, unlike the quarterly data files. But because the data are so incomplete and inconsistent, I kept my initial search fairly broad and narrowed it down myself.

That's what my script does — it queries the API for aripiprazole-related adverse events from each year, then winnows the 80K+ records down to fewer than 400 that were relevant to the story. Ahead of time, I decided that my criteria for what reactions to include were as follows:
1. The event occurred in a child between the ages of 3-17.
2. Aripiprazole is suspected to have caused the event.
3. The aripiprazole was prescribed to treat autism.

But first, I had to get all the records where aripiprazole was listed. I tried out a number of queries that searched various nested fields for all the name variations I could think of. Ultimately, because different records stored drug names in different fields, (medicinalproduct, activesubstancename, etc.) I decided to search each record in its entirety for a list of aripiprazole name variants, yielding 80,915 results total. (By comparison, a search of FAERS public dashboard yielded just 60K results for a search of "aripiprazole." So you can see why it's better to get the data yourself when you're looking at FAERS for reporting purposes.) See blocks 2 and 3 of the notebook for the exact query and parameters I used.

### Collecting 81K records

There are some limits on how much data you can collect from the adverse events API at once. Here are some tricks I used to get around them.

The API uses two parameters to page through results: "limit," which controls how many results are returned at once, and "skip," which controls how many records you pass by before you collect the next batch. There's a max limit of 100 and a max skip of 25K. To get around the skip limit, I batched my results by year. Within each year, I simply incremented "skip" by 100 each time I queried the API until I'd collected all the records for that year.

See block 4 of the notebook for more detail on how I iterated through the "pages" of results.

### Cleaning the data

Another quirk about FAERS data—while most adverse event reports include the patient's age in years, others give that info in weeks, months, or even decades. Others have no age info whatsoever. So I stripped out 32310 records with nothing in the 'patientonsetage' field. For the rest, I changed the contents of 'patientonsetage' based on the contents of the 'patientonsetageunit' field, which has a code for each possible age unit. See blocks 7 and 8 for details.

Later on, to make subsequent filtering easier, I stripped out records where aripiprazole (or one of its spelling variants) was not included in the 'medicinalproduct' field. I then searched the removed rows to make sure I wasn't accidentally removing any related to autism. (See blocks 10 and 11.)<sup>[1](#myfootnote1)</sup>

### Filtering the data

Once the ages were standardized, I made a new list that included only records for patients who were between 3 and 17 years old at the time the reactions occurred—6750 in total. I also caught 53 stragglers with "None" in the 'patientonsetageunit' field, which means the cleaning method described above should be improved.

Then, after some more cleaning, I filtered again to include only records with a 'drugcharacterization' value of 1, indicating that aripiprazole is suspected to have caused the bad reaction.

Finally, I filtered by 'drugindication,' including only records in which aripiprazole was prescribed to treat autism.

### Classification and analysis

I created a dataframe in pandas and iterated through the records, adding every individual reaction, stored in field 'reactionmeddrapt,' as a row. (Many adverse event reports include multiple reactions.) The result was a raw list of reactions, with one entry for every time a given reaction, like weight gain, was reported.

I exported the df to a csv; in Excel, I created a pivot table to count the frequency of each reaction. (You could probably just do the entire analysis in pandas too.) Theoretically, you could just plug the counts into a pie chart and be done with it. But this data won't make a good viz as-is; there are hundreds of unique reactions, some of which, like dyskinesia and other movement disorders, seemed like they should be considered together. Thankfully, because the reactions are already named using a medical taxonomy called [MedDRA](https://www.meddra.org/), it's simple enough to label them with MedDRA's own category information. I applied for a subscription and got free access to the MedDRA browser, which will do this "hierarchy analysis" in seconds on a properly formatted spreadsheet.

Once the reactions were categorized, I was able to see which broad classes of reactions occurred most frequently in the FDA data. The final DV includes only the broad categories that represented more than 5% of the reported adverse events. And within each broad category, only subtypes that represent more than 5% of their category are included.

I also excluded reports of the drug being taken incorrectly, as well as most of the "Investigations" category, which mainly includes lab test results. The exceptions to this rule are weight gain and weight loss, which I included under the "Metabolic" category.

Finally, I adapted the names of the broad classes and subcategories to be shorter and more easily understood.

For more details on my analysis, check out my [data diary](https://docs.google.com/document/d/1cRD68SWKSaM6tS_kHSqI6HiZumHVEm77wfyD0pNMtic/edit?usp=sharing). The spreadsheet where I did this work can be found in this repo in the "data" folder.


## How you can do it

### Getting started with Jupyter Notebooks
*Note: This setup is almost identical to what I learned in [Lam Thuy Vo](https://github.com/lamthuyvo)'s data journalism class at the CUNY Graduate School of Journalism. All credit for it goes to her.*

The first step is to get set up with Jupyter inside a virtual environment. Mine is included in the repo for completeness's sake, but you'll probably want to create your own. Here's how:

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

At minimum, you'll want to change the drug names in blocks 3, 4, and 10, plus the variable names and print statements in block 12.

Depending on what indication you're interested in, you may also want to update blocks 11, 13 and 14.

## Any questions?
If you need help or want more information, drop me a line at jaclyn@spectrumnews.org. And if this tutorial helps you with a project, I'd love to see the end result!

---


<a name="myfootnote1">Footnote 1</a>: This isn't ideal- after all, what's the point of searching all fields when we're just going to limit ourselves to the 'medicinalproduct' field anyway? In my case specifically, all the reports I was interested in ended up having aripiprazole or one of its spelling variants in this field. And being able to name a specific field made it much, much easier to filter the data by drug characterization and indication. But your results may be different depending on what sort of drug and age range you're interested in. So my recommendation is to search all fields for your drug name, then adapt blocks 10-13 depending on how many relevant records have the 'medicinalproduct' field filled out.
