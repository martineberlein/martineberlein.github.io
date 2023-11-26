---
permalink: /projects/
title: "Projects"
excerpt: "Projects"
author_profile: true
---

💡 **Watch out!** This site is under Construction.
{: .notice--warning}

## **EvoGFuzz**: Evolutionary Grammar-Based Fuzzing

_EvoGFuzz_ is an evolutionary grammar-based fuzzing approach to optimize probabilities to generate program inputs that trigger exceptional behavior.
With _EvoGFuzz_, it is possible to explore the program input space systematically by using evolutionary optimization with a user-definable goal.

To detect defects and vulnerabilities efficiently, automatically generated program inputs must conform to the input format's structure; thus, we can use grammars to generate syntactically correct inputs.
In this context, _grammar-based_ _fuzzing_ can be guided by probabilities attached to competing rules in the grammar, leading to the idea of probabilistic grammar-based fuzzing. 
Assigning and optimizing probabilities of individual expansions gives us great control over which inputs should be generated.
Furthermore, by choosing probabilities wisely, we can direct fuzzing towards specific functions and features – for instance, towards functions that are particularly critical, prone to failures, or that have been recently changed.

### Example

Our running example is a simple calculator-subject that accepts arithmetic expressions as inputs.

💡[Info]: This chapter is based on the [fuzzingbook](https://www.fuzzingbook.org) and uses their implementations of specific functions and data structures. Give it a read!
{: .notice--info}

As a first step towards using _EvoGFuzz_,  we formalize the programs input as a context free-grammar in BNF.
```python
from fuzzingbook.Grammars import Grammar

CALCULATOR: Grammar = {
    '<start>': ['<expr>'],
    '<expr>': ['<term> + <expr>', '<term> - <expr>', '<term>'],
    '<term>': ['<factor> * <term>', '<factor> / <term>', '<factor>'],
    '<factor>': ['<sign-1><factor>', '(<expr>)', '<integer><symbol-1>'],
    '<sign>': ['+', '-'],
    '<integer>': ['<digit-1>'],
    '<digit>': ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9'],
    '<symbol>': ['.<integer>'],
    '<sign-1>': ['', '<sign>'],
    '<symbol-1>': ['', '<symbol>'],
    '<digit-1>': ['<digit>', '<digit><digit-1>']
}
```

Now, lets load two initial input samples and observe the calculator's behavior.
To do this, we load the function execute_samples, which calls the calculator program and executes it. 
We can call the function with a list of input samples, and it returns the corresponding execution outcome (label/oracle).
The output is a list of booleans, that state whether an exception (crash) was triggered.


```python
from execution import execute_samples

# Load initial input files
sample_list = ['tan(12)', 'sqrt(-16)']

# we call the program under test with 
# the function "execute_samples(List[str]) -> List[bool]"
oracle = execute_samples(sample_list)

# display the program output
for i, row in enumerate(oracle['oracle']):
    print(sample_list[i].ljust(30) + str(row))
```

Output:
```python
tan(12)         False
sqrt(-900)      True
```

We observe that the sample sqrt(-16) triggers a bug in the calculator, whereas the sample tan(12) does not show unusual behavior.
However, wouldn't it be great if we could find these kind of bugs automatically?

```python
from evogfuzz import EvoGFuzz

evogfuzz = EvogFuzz(
    grammar = CALCULATOR
    initial_input = ['tan(12)', 'sqrt(-16)'],
    evaluation_function = execute_samples
)

evogfuzz.run()
```


### Try it out!

You want to try EvoGFuzz out for your own examples or programs? Then, we recommend our python implementation on [GitHub]() providing an easily accessible introduction and a number of different examples and fitness functions.

Further recommended resources for diving deeper into EvoGFuzz are:

- EvoGFuzz’s project page contains [installation instructions]() for different scenarios.

- Our [scientific paper on EvoGFuzz](), published at SSBSE'2020. The paper describes the individual steps and the idea of fitness functions more formally.


## Sementic Debugging

> WIP


## AlhazenML: 

<details>
  <summary>Click me</summary>
  
  ### Heading
  1. Foo
  2. Bar
     * Baz
     * Qux

  ### Some Code
  ```js
  function logSomething(something) {
    console.log('Something', something);
  }
  ```
</details>
