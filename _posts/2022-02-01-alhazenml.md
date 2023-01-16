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
**In this notebook, we will build the tool _Alhazen_ proposed by Kampmann et al. and extend it with additional machine learning models.**

Parts of this notebook were a joint work with my colleague [Hoang Lam Nguyen](https://www.informatik.hu-berlin.de/en/Members/hoang-lam-nguyen) from Humboldt-Universität Zu Berlin.

💡 **Coming Soon! This Site is currently under construction and is only updated periodically (i.e. when I have time :) )**
{: .notice--warning}

## Why do we need to explain program behavior?

💡 [Info]: We use the functionality provided by [The Fuzzingbook](https://www.fuzzingbook.org). For a more detailed description of grammars, have a look at the chapter [Fuzzing with Grammars](https://www.fuzzingbook.org/html/Grammars.html).
{: .notice--info}

To illustrate Alhazen's use case and necessity, we start with a quick motivating example.
First, let me introduce to you our program under test: **The Calculator**.
This infamous program is similar to the one described in the original paper;
however, we introduced a synthetic `BUG` that you have to explain utilizing the machine learning models learned by Alhazen.

💡[Note] We altered the calculator's behavior to make the bug explanation more challenging. The introduced bug by Kampmann et al. is different from ours.
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

The `CALCULATOR_GRAMMAR` consists of several production rules, non-terminals, and terminals. Therefore, with this syntactic input format specification, the calculator only accepts inputs conforming to this grammar definition.

Now, let us load two mathematical equations and test our calculator:

```python
from alhazen.helper import read_files

# Load initial input files
sample_list = read_files(['src/samples/calculator.1.expr', 'src/samples/calculator.2.expr'])
if display_output:
    display(sample_list)
```

Output:
```
sqrt(-16)
tan(4)
```

Let's execute our two input samples and observe the calculator's behavior. To do this, we load the function evaluate_samples. We can call the function with a list of input samples, and it returns the corresponding execution outcome (label/oracle).
The output is a set of tuples (input: str, oracle: OracleResult).

```python
from typing import Set, Tuple
# Load function execute samples
from alhazenML.calculator import evaluate_samples
from alhazenML.oracle import OracleResult

# evaluate_samples(List[str])
oracle: Set[Tuple[str, OracleResult]] = evaluate_samples(sample_list)
if display_output: 
    display(oracle)
```

This gives us the following output:

```python
{
    ('sqrt(-16)', OracleResult.BUG),
    ('tan(4)', OracleResult.NO_BUG)
}
```

We observe that the sample sqrt(-16) triggers a bug in the calculator, whereas the sample tan(4) does not show unusual behavior.
Of course, we want to know why the input sample fails the program. In a typical use case, the developers of the calculator program would now try other input samples and evaluate if similar inputs also trigger the program's failure.
Let's try some more input samples; maybe we can refine our understanding of why the calculator crashes:

```python
# Our guesses (maybe the failure is also in the cos or tan function?)
guess_samples = ['cos(-16)', 'tan(-16)', 'sqrt(-100)', 'sqrt(-20.23412431234123)']

# lets obtain the execution outcome for each of our guess
guess_oracle = evaluate_samples(guess_samples)

# lets show the results
if display_output:
    display(guess_oracle)
```

Output:

```python
{
    ('cos(-16)', OracleResult.NO_BUG),
    ('tan(-16)', OracleResult.NO_BUG)
    ('sqrt(-100)', OracleResult.NO_BUG),
    ('sqrt(-20.23412431234123)', OracleResult.BUG)
}
```

We observe that the failure only seems to occur in the _sqrt(x)_ function, however, only for specific _x_ values. We could try other values for _x_ and repeat the process.
However, this would be highly time-consuming and not an efficient debugging technique for a larger and more complex test subject.

**Wouldn't it be great if there was a tool that automatically does this for us? And this is precisely what Alhazen is used for. It helps us explain under which circumstances input files fail a program.**

💡 [Info]: Alhazen is a tool that automatically learns the circumstances of program failure by associating syntactical features of sample inputs with the execution outcome. The produced explanations (in the form of a decision tree) help developers focus on the input space's relevant aspects.
{: .notice--info}

## Implementing Alhazen

**Stay Tuned! More coming soon!**
{: .notice--info}