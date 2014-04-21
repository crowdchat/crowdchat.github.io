---
layout: post
author: Dani Rayan
title: "Identifying Real Humans on Twitter"
date: 2013-11-06 16:58
comments: true
author: Dani Rayan
categories: 
- Machine Learning
- by Dani Rayan
---

In CrowdChat platform, we have classified over 66M Twitter accounts and counting. The important precursor
step: Finding out real-humans among those millions of accounts. This may appear as seemingly intuitive task for any average person, but is not so for the machines. This post descirbes how we approached the problem. The reader should refer further for the following concepts, but we will try to introduce them as briefly as possible.

<!-- more -->

Problem:
Given an twitter handle, detect whether the account is a real human or non human 
A non human account could be either organizational or a bot.

The of problem of identifying, if a twitter account belongs to real-human or not can be translated to the problem of identifying if their self-described bio belongs to a language class H.

Lets assume for now, H is language class of all self-described bios that sound like written by Humans.
And B is the language class of all bios that are written by Bots/Organization Accounts/Non-Humans.

There are always more humans than non humans in popular social networks like Twitter. Services such as Twitter mention that percentage of bot accounts is insignifcant and they constantly work towards keeping it as less and possible. Hence, it is valid to assume that in a truly randomized sample the percentage of human accounts should always be more than the percentage of non human accounts.

From a basic study , we identify that humans tend to use words like ‘I’ , ‘my’ etc in their bio while “organizational” accounts use the word ‘official’ , “news” , “we” etc. , we call these seed rules.

Next , we know that a bio is made of natural english words. 

Lets say one’s account has a bio as follows, “I live in California and work at Crowdchat”
this bio can be split into ngrams 
unigram = [“i”, “live”, “in” ,”california”, “and” , “work”, “at” , “crowdchat”]
bigrams = [“i live”, “live in”, “in california”, “california and” ,”and work”, “work at”, “at crowdchat”]


Now its a 0-1 classification problem  , we got inspired by “Edwin chen’s blog post about a similar problem of filtering non english tweets from english tweets” [(Link)](http://blog.echen.me/2011/05/01/unsupervised-language-detection-algorithms/)  and decide to use the same technique.


#Some Basics:

A given tweet (in English) is composed of set of character n-grams.

We use EM Algorithm.

Let’s recall the naive Bayes algorithm: given a tweet (a set of character n-grams), we estimate its language to be the language L that maximizes
```
P(language=L|ngrams)∝P(ngrams|language=L)P(language=L)
Thus, we need to estimate P(ngram|language=L) and P(language=L).
```
This would be easy if we knew the language of each tweet, since we could estimate

```
P(xyz|language=English) as #(number of times “xyz” is a trigram in the English tweets) / #(total trigrams in the English tweets)
P(language=English) as the proportion of English tweets.

```

Or, it would also be easy if we knew the n-gram probabilities for each language, since we could use Bayes’ theorem to compute the language probabilities for each tweet, and then take a weighted variant of the previous paragraph.

#Solution Outline:

```

1. Using Seed rules , assign human (label =1) or non human (label = 0 ) for the accounts in the vertical.
2. If seed rule does not pass , assign a label randomly..

E- Step: Using the labels computed , compute conditional n-gram probability 
P(Ngram / label =1) 
P(Ngram / label = 0)


M-Step: Using the n-gram probability - compute human / non human probability

P(Human  = 1 / features)
P (human = 0 / Features)


Repeat E-M steps until convergence or for arbitrary number of times.
```
