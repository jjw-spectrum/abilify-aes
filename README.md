# Finding adverse events with the FAERS API
### By: Jaclyn Jeffrey-Wilensky (jaclyn@spectrumnews.org)


This GitHub repo contains the Jupyter Notebook I used to find adverse events in autistic children prescribed aripiprazole for the Spectrum story [Hed TK](https://www.spectrumnews.org). In this document, I'll describe how you can use this notebook for your own purposes. I'll also lay out the steps I took to go from the raw FAERS data to the visualization that you see in the story.

These steps are written for relative beginners like me; if you're a more advanced user, you can skip straight to [Adapting the notebook](https://github.com/jjw-spectrum/abilify-aes/blob/master/README.md#adapting-the-notebook).

## About FAERS
[FAERS](https://open.fda.gov/data/faers/) is the FDA's Adverse Events Reporting System. It's essentially a huge database of every bad reaction to a drug ever reported to the FDA.

The data in FAERS is not comprehensive or FDA-verified, and it can't be used to, say, compare two drugs and say which causes more bad reactions. But if used correctly, it can be a useful complement to the rest of your reporting. The other thing to know about FAERS is that it's a huge mess! Duplicates run rampant, records are often incomplete, and drug names are often inconsistent or misspelled. For example, here are just a few of the terms I used to capture the aripiprazole records:
-aripiprazol (left off the "e" to include Italian and other names for the drug)
-ariprazol (a common misspelling)
-abilify (the drug's brand name)
-aristada (the drug's brand name in the UK)
-

FAERS is accessible through

## How I did it

### Data collection

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
