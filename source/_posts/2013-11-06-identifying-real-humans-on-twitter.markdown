---
layout: post
author: Dani Rayan
title: "Identifying Real Humans on Twitter"
date: 2013-11-06 16:58
comments: true
categories: [Machine Learning, Dani Rayan]
---

In CrowdChat platform, we have classified over 66M Twitter accounts and counting. The important precursor
step: Finding out real-humans among those millions of accounts. This may appear as seemingly intuitive task for any average person, but is not so for the machines. This post descirbes how we approached the problem. The reader should refer further for the following concepts, but we will try to introduce them as briefly as possible.

The of problem of identifying, if a twitter account belongs to real-human or not can be translated to the problem of identifying if their self-described bio belongs to a language class H.

Lets assume for now, H is language class of all self-described bios that sound like written by Humans.
And B is the language class of all bios that are written by Bots/Organization Accounts/Non-Humans.

A given tweet (in English) is composed of set of character n-grams.
#EM Algorithm

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
