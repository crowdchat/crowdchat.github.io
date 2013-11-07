---
layout: post
author: Dani Rayan
title: "Identifying Real Humans on Twitter"
date: 2013-11-06 16:58
comments: true
categories: [Machine Learning, Dani Rayan]
---

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
