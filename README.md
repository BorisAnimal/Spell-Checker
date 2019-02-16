# Spell-checker

## Problem & solution
The base solution for spell checking was provided by P.Norvig and it based on Damerau Levenshtein distance.

**Damerau Levenshtein distance** - is a string metric for measuring the edit distance between two sequences. Informally, the Damerau–Levenshtein distance between two words is the minimum number of operations (consisting of **insertions**, **deletions** or **substitutions** of a single character, or **transposition** of two adjacent characters) required to change one word into the other ([wiki](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance)).

Naive implementation of this algorithm has high algorithmic complexity: ```2n+2an+a-1``` where ```a``` is length of alphabet, ```n``` is length of checked word. This complexity can be critical for hieroglyph based alphabets (such as chinese has ~70000 unique symbols)[1].

The solution described by W. Garbe improves Norvig's approach. The idea is: 3 of 4 operations (especially language sensitive operations - insertion, substitution) could be reproduced using only deletions - language independent operation [1]. Another difference of Garbe's solution is index or dictionary structure. Here keeps not only original words and their frequences as Norvig did, but also set of their possible deletions as keys of index (frequency of deletions of words not counted even if they match really existing word) and set of words that are predecessors by deletion operation for given key. For example: 
```python
# Frequences of original words keeps in other data structure
{
    eoo : {'ebook'}
    book : {'brook', 'betook', 'ebook', 'brooks', 'booked', 'book_', 'ebooks', 'books'}
    boo : {'booby', 'boost', 'brook', 'broom', 'boom', 'boots', 'boot', 'blood', 'boone', 'book', 'bosom', 'ebook', 'boor', 'booty', 'boon', 'booth', 'bloom', 'brood', 'books', 'book_'}
}
```
key ```'eoo'``` taken by deletion of 2 chars from word ```'ebook'```. Key ```'book'``` is word by itself, but also can be given from words such as ```'brook'```, ```'ebook'```, etc. 

Using this structure we can represent operations of Damerau Levenshtein distance as:
* **deletion** as it is
* **insertion** - get predecessors for given word from index
* **substitution** - get possible deletions and get set of their predecessors from index
* **transposition** - same as **substitution**

Memory consumption can be underlined as main dissadvantage of such a method. Formally it depend on depth of deletions in phase of indexing and aproximately 
```
O(n,d) ~ n * log(d)
```
where ```d``` is depth of deletions, ```n``` is number of unique words [4].

## Algorithm 
*describes and justifies decisions made by you while building it*
In implementation of such an algorithm was made assumption that maximum depth of deletions is 2. It was made to make implementation more intuitive but, as implication, less general. 

Dataset for indexing and testing was given from Norvig's site.

As was mentioned above, after indexing there are two structures: dictionaries of words and their predecessors (index) and dictionary of (only) original words and their amount of accurances in given dataset.

This is not obvious but Norvig's implementation has priority by distance:
```python
def candidates(word): 
    "Generate possible spelling corrections for word."
    return (known([word]) or known(edits1(word)) or known(edits2(word)) or [word])
```
here by ```or``` operation will be returned first non-empty result of function known (or word itself). It was reproduced in my implementation of SymSpell.

In fact there are bunch of parameters in spell checking algorithms such as *threshold* - minimum value of accurences of a word to be counted as non-typo in dataset, *distance function* (Levenshtein, Damerau-Levenshtein, Hamming, etc), *selection function* - in both implementation this is probability (word with maximum frequency will be selected).
To reproduce results of Norvig algorithm *distance function* is Damerau-Levenshtein was selected and *threshold* = 0 (also it needed because in *Big.txt* dataset Norvig added correct words with frequency = 1, hence *threshold* > 0 would ignore them).

Due to in each case probability counted as 

#### $$ P=freq(w)/NumWords $$

and ```NumWords``` is equal for each word - so I counted just as 

#### $$ P=freq(w) $$


## Algorithm evaluation
*reports an evaluation of your work - find a way to evaluate your system*
For testing were selected datasets with context independent errors or just with typos.

Due to Norvig provided tests for his algorithm that include checking of correctness and performance, I selected them as well. 
Also were used additional datasets: aspell, birkbeck, wikipedia [5]. 

| Dataset | Norvig acc | Norvig speed | SymSpell acc| SymSpell speed | Unknown words |
| --------       | -------- | -------- | -------- | -------- | -------- |
| spell-testset1 | 75%     | 17 words/sec    | 75% | 441 words/sec  | 6%  |
| spell-testset2 | 68%     | 24 words/sec    | 68% | 1257 words/sec | 11% |
| aspell         | 43%     | 12 words/sec    | 43% | 971 words/sec  | 23% |
| wikipedia      | 61%     | 18 words/sec    | 61% | 1006 words/sec | 24% |
| birkbeck       | 31%     | 14 words/sec    | 31% | 496 words/sec  | 11% |

Even if it not reached 1000 times [1] or 1000000 times [4] results, at least it proved order of 10 result with same quality. 

Main part of errors is about unknown words, due to dataset is quiet simple, also there is no preprocessing of a punctuation characters.

## References
[1] [W. Garbe, 1000x Faster Spelling Correction algorithm (2012)](https://medium.com/@wolfgarbe/1000x-faster-spelling-correction-algorithm-2012-8701fcd87a5f)
[2] [P. Norvig, How to Write a Spelling Corrector](http://norvig.com/spell-correct.html)
[3] [G. Rutenberg, Damerau-Levenshtein Distance in Python](https://www.guyrutenberg.com/2008/12/15/damerau-levenshtein-distance-in-python/)
[4] [Fast approximate string matching with large edit distances in Big Data (2015)](https://medium.com/@wolfgarbe/fast-approximate-string-matching-with-large-edit-distances-in-big-data-2015-9174a0968c0b)
[5] [Spelling Corrector](https://www.kaggle.com/bittlingmayer/spelling/version/2#wikipedia.txt)

[6] [Source code]
