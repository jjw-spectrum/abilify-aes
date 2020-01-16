# Finding adverse events with the FAERS API
### By: Jaclyn Jeffrey-Wilensky (jaclyn@spectrumnews.org)


This GitHub repo contains the Jupyter Notebook I used to find adverse events in autistic children prescribed aripiprazole for the Spectrum story [Hed TK](https://www.spectrumnews.org). In this document, I'll describe how you can use this notebook for your own purposes. I'll also lay out the steps I took to go from the raw FAERS data to the visualization that you see in the story.

These steps are written for relative beginners like me; if you're a more advanced user, you can skip straight to [Adapting the notebook](https://github.com/jjw-spectrum/abilify-aes/blob/master/README.md#adapting-the-notebook).

## Getting started
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

## Customizing the notebook

## Analyzing the data
