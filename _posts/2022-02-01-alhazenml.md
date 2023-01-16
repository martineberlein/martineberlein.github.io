---
title: 'AlhazenML: Explaining Program Behavior'
date: 2022-08-01
permalink: /posts/2022/02/alhazenml/
tags:
  - evogfuzz
  - fuzzing
  - testing
---

Considering the difficulties of determining the circumstances of a program’s behavior, Kampmann et al. [[KHSZ20]](https://dl.acm.org/doi/abs/10.1145/3368089.3409687) presented an approach to automatically learn the associations between the failure of a program and the input data.
Their proposed idea affiliates syntactical input features, like input length or the presence of particular derivation sequences, with faulty program behavior.
Thus, based on these associations, their tool _Alhazen_ forms hypotheses on why failure-inducing input files result in a program defect.
In this notebook, we will build the tool _Alhazen_ proposed by Kampmann et al. and extend it with additional machine learning models.

Parts of this notebook were a joint work with my colleague [Hoang Lam Nguyen](https://www.informatik.hu-berlin.de/en/Members/hoang-lam-nguyen) from Humboldt-Universität Zu Berlin.

💡 **Coming Soon! This Site is currently under construction and is only updated periodically (i.e. when I have time :) )**
{: .notice--warning}

💡 [Info]: We use the functionality provided by [The Fuzzingbook](https://www.fuzzingbook.org). For a more detailed description of grammars, have a look at the chapter [Fuzzing with Grammars](https://www.fuzzingbook.org/html/Grammars.html).
{: .notice--info}

## Why do we need to explain program behavior?

To illustrate Alhazen's use case and necessity, we start with a quick motivating example.
Let me introduce to you our program under test: **The Calculator**.
This infamous program is similar to the one described in the original paper;
however, we introduced a synthetic `BUG` that you have to explain utilizing the machine learning models learned by Alhazen.

💡[Note] **We altered the calculator's behavior to make the bug explanation more challenging. The introduced bug by Kampmann et al. is different from ours.**
{: .notice--warning}

Our program under test is a typical _calculator_ that accepts arithmetic equations and trigonometric functions and allows us to calculate the square root. To help us determine faulty behavior, i.e., a crash, we implemented an evaluation function that takes an input file and returns whether a bug occurred during the evaluation of the mathematical equations (`BUG`, `NO_BUG`). 
As the calculator only accepts a subset of all mathematical equations, input files must conform to a syntactical input format called grammar.
So let us have a look at the grammar definition:


```python
# Load Grammar Definition
from fuzzingbook.Grammars import Grammar

# Custom Calculator Grammar from Kampmann et al. (See paper - with out regex)
CALCULATOR_GRAMMAR: Grammar = {
    "<start>":
        ["<function>(<term>)"],

    "<function>":
        ["sqrt", "tan", "cos", "sin"],
    
    "<term>": ["-<value>", "<value>"], 
    
    "<value>":
        ["<integer>.<integer>",
         "<integer>"],

    "<integer>":
        ["<digit><integer>", "<digit>"],

    "<digit>":
        ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
}
START_SYMBOL = "<start>"
```