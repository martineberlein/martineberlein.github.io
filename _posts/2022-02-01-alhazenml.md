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

💡 **Coming Soon! This Site is currently under construction and is only updated periodically (i.e. when I have time :) )**
{: .notice--warning}