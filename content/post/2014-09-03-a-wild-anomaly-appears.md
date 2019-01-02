---
title: "A wild anomaly appears!"
date: 2014-09-02
comments: true
---

So, I'm working on the new Data Team at [Stack Exchange](http://stackexchange.com) now. Truth is we have no idea what we're doing ([WANNA JOIN US?](http://careers.stackoverflow.com/jobs/62161/data-science-engineer-stack-exchange)). But every now and then we come across something that works a little too well and wonder why we haven't heard about it before.

We run a [niche job board for programmers](http://careers.stackoverflow.com/jobs) that has about 2900 jobs on it this morning. Quality has been pretty easy to maintain. We have a great spam filter called "The $350 Price Tag". Then we have some humans that look over the jobs that get posted looking for problems. Overall the system works well, but at 2900 jobs a month that means someone has to look through about 150 jobs every working day. They're looking for a needle in a haystack as most (>95%) of the jobs posted are perfectly appropriate for the board, so there's a lot of "wasted" time spent looking for ghosts that aren't there. And it's pretty boring to boot. I'm sure that person would rather do other things with their time.

It'd be nice if we had an automated way of dealing with this. We have no idea what we're doing, so we frequently just reach into our decidedly meager bag of tricks, pull one out, and try it on a problem. I'd done that a few times before on this problem, trying Naive Bayes or Regularized Logistic Regression, but had gotten nowhere. There are a lot of different ways a job can be appropriate for the board and there are a lot of different ways a job could be not appropriate for the board which makes coming up with a representative training set difficult.

Last week while taking another whack at the problem I Googled "Text Anomaly" and came across David Guthrie's 186 page Ph. D. thesis, [Unsupervised Detection of Anomalous Text](http://nlp.shef.ac.uk/Completed_PhD_Projects/guthrie.pdf). There's a lot there, but the novel idea was simple enough (and it worked in his experiments and mine) that I'm surprised I haven't heard about it until now.

###Distance to the Textual Complement

Say you have a bunch of documents. You pull one out and want to determine how anomalous it is with respect to all the others. Here's what you do:

1. Choose some features to characterize the document in question.
2. Convert the document to its feature representation.
3. Treat all the other documents as one giant document and convert that to its feature representation.
4. Calculate the distance between the two.

Do this for every document and sort the results descending by the distance calculated in step 4. The documents at the top of the list are the "most anomalous".

That's it. Pretty simple to understand and implement. There are two choices to make: which features, and which distance metric to use.

##Obscurity of Vocabulary, I choose you!

In any machine learning problem you have to come up with features to characterize the set of things you're trying to work on. This thesis is chock full of features, 166 of them broken up into a few different categories. This part of the paper was a goldmine for me (remember, I have no idea what I'm doing). The text features I knew about before this were word counts, frequencies, tf-idf and maybe getting a little into part of speech tags. The kinds of features he talks about are stuff I never would've come up with on my own. If you're doing similar work and are similarly lost, take a look there for some good examples of feature engineering.

The set of features that stood out to me the most were the ones in a section called "Obscurity of Vocabulary Usage". The idea was to look at a giant reference corpus and rank the words in the corpus descending by frequency. Then you make lists of the top 1K, 5K, 10K, 50K, etc. words. Then you characterize a document by calculating the percentages of the document's words that fall into each bucket.

##Manhattan Distance, I choose you!

Guthrie pits a bunch of distance metrics against eachother and for Distance to the Textual Complement method the [Manhattan distance](http://en.wikipedia.org/wiki/Taxicab_geometry) got the blue ribbon, so I used that.

#Setup

When I've been looking through the jobs before I can pretty much tell by their titles whether they're busted or not, so my documents were just the job titles. There isn't really a good reference corpus from which to build the Top-N word lists, so I just used the job titles themselves. I tried a couple different sets of Ns but ended up on 100, 300, 1000, 3000, and 10000 (Really ~7,000 as that's the number of unique terms in all job titles).

<a href="http://i.imgur.com/UxtpK5K.png"><img src="http://i.imgur.com/UxtpK5K.png" /></a>

#Results

Here's the sort of all the jobs that were on the board yesterday.

<a href="http://i.imgur.com/tfdPgaE.png"><img src="http://i.imgur.com/tfdPgaE.png" /></a>

Basically everything on the right is good and has low scores (distances).

Most of the jobs on the left have something anomalous in their titles. Let's group up the anomalies by the reasons they're broken and look over some of them.

####Stuff that just ain't right

These jobs just don't belong on the board. We need to follow up with these people and issue refunds.

1. Supervisor Commercial Administration Fox Networks Group
1. Sales Executive Risk North Americas
1. Associate Portfolio Manager Top Down Research
1. Senior Actuarial Pre-Sales Consultant
1. Ad Operations Coordinator
1. NA Customer Service Representative for Sungard Energy
1. Manager, Curation

####Just Terrible Titles

These jobs would belong, but the titles chosen were pretty bad. Misspellings, too many abbreviations, etc. We need to follow up with our customers about these titles to improve them.

1. Javascript Devlopers Gibraltar
1. Sr Sys Admin Tech Oper Aux Svcs
1. VR/AR Developer

####Duplicate Information

![Duplicate Information](http://i.imgur.com/lUUTeNX.png)

Anywhere you see the title for a job on [Stack Overflow](http://stackoverflow.com) or [Careers](http://careers.stackoverflow.com) we also show you things like the location of the job, whether it's remote or not, and the company's name. These titles duplicate that information. We need to follow up with our customers to improve them.



1. Delivery Manager, Trade Me, New Zealand
1. Visualization Developer, Calgary
1. Technical Expert Hyderabad 
1. Technical Expert Pune 
1. Technical Expert Chennai 
1. Technical Expert Gurgaon 
1. New York Solutions Architect
1. Sr Fullstack Eng needed for Cargurus We reach over 10MM unique visitors monthly
1. Sr. Tester, Sky News, West London
1. Chief Information Officer/CIO Audible.com
1. Machine Learning Engineer Part Time Remote Working Available

##What about the false positives?

A number of false positives are produced (this is just a sample):

1. Computer Vision Scientist
1. Mac OSX Guru
1. Angularjs + .NET + You
1. Android Developer 100% Boredom Free Zone
1. Java Developer 100% Boredom Free Zone
1. DevOps Engineer - Winner, Hottest DC Startups!!! $10M Series A
1. Jr. Engineer at Drizly

Some of these (Computer Vision, Mac OSX) are just infrequently found on our board. Some of these people are trying to be unique (and are successful, by this analysis) so that their listing stands out.

Guthrie goes into a bit of detail about this in a section on precision and recall in the paper. His conclusion is that this kind of anomaly detection is particularly suited to when you have a human layer of detectors as the last line of defense and want to reduce the work they have to do. An exhaustive exploration of the scores finds that all of the jobs we need to follow up on are in the top 10% when ordered descending by their anomaly scores. Setting that threshold should cut the job our humans have to do by 90%, making them happier and less bored, and improving the quality of the job board.



