---
title: "Nlp &#58; Detect Noun Phrase and Verb Phrase"
layout: post
date: 2019-01-04 18:30
tag:
- nlp
- python
category: blog
author: taranjeet
description: This post is about detecting noun phrase and verb phrase.
---

This post is about detecting noun phrase and verb phrase using [stanford-corenlp](https://github.com/Lynten/stanford-corenlp) and [nltk](https://www.nltk.org/).

First lets us install `stanford-corenlp` and `nltk` libraries. We will be using `stanford-corenlp` library to detect noun and verb phrase and then extract them using `nltk`.

```sh
pip install stanfordcorenlp

pip install nltk
```

Now lets us take a sample sentence and detect noun phrase from it

```python

from stanfordcorenlp import StanfordCoreNLP
from nltk.tree import Tree

nlp = StanfordCoreNLP('/path/to/stanford-corenlp-full-2018-10-05')

sentence = 'Who drives a tractor?'


def extract_phrase(tree_str, label):
    phrases = []
    trees = Tree.fromstring(tree_str)
    for tree in trees:
        for subtree in tree.subtrees():
            if subtree.label() == label:
                t = subtree
                t = ' '.join(t.leaves())
                phrases.append(t)

    return phrases


tree_str = nlp.parse(sentence)

print tree_str
# u'(ROOT\n  (SBARQ\n    (WHNP (WP Who))\n    (SQ\n      (VP (VBZ drives)\n        (NP (DT a) (NN tractor))))\n    (. ?)))'

nps = extract_phrase(tree_str, 'NP')
print nps
# [u'a tractor']
```

Note if you face any error of `AccessDenied` (psutil.AccessDenied), run/open the python shell using `sudo`

```sh
sudo python
```

Now let us use the same function to detect verb phrase

```python
from stanfordcorenlp import StanfordCoreNLP
from nltk.tree import Tree

nlp = StanfordCoreNLP('/path/to/stanford-corenlp-full-2018-10-05')

sentence = 'Farmer drives a tractor'


def extract_phrase(tree_str, label):
    phrases = []
    trees = Tree.fromstring(tree_str)
    for tree in trees:
        for subtree in tree.subtrees():
            if subtree.label() == label:
                t = subtree
                t = ' '.join(t.leaves())
                phrases.append(t)

    return phrases


tree_str = nlp.parse(sentence)
print tree_str
# u'(ROOT\n  (SBARQ\n    (WHNP (WP Who))\n    (SQ\n      (VP (VBZ drives)\n        (NP (DT a) (NN tractor))))\n    (. ?)))'

vps = extract_phrase(tree_str, 'VP')

print vps
# [u'drives a tractor']
```

### Summary

This post is about how can we use `stanford-corenlp` and `nltk` to detect noun phrase and verb phrase.
