---
layout: post
title: "[ICSE 2023] Exploring LLM-based General Bug Reproduction"
date: 2023-08-27 16:08:00+0900
description: "An explanation of my ICSE'23 work, and perspective one year after submitting the paper"
categories: academia LLM test_generation
giscus_comments: true
related_posts: true
---

(Associated files, such as the bibtex or slides, can be found [here](/al-folio/publications/#Kang2023aa).)

TLDR: This paper introduces `LIBRO`, a technique that automatically reproduces bug reports with high precision.

## Motivation

When users or developers find bugs, they can report these bugs on issue tracker software, such as on GitHub as issues or on Jira. Such reports are called "bug reports". Bug reports are generally written in natural language, and may be submitted without a **test** that reproduces the issue. This is a difficult situation for developers, as they will then have to manually try to reproduce the issue in the bug report by writing their own test code. Given the large volume of bug reports being submitted every day, this burden can be overwhelming. Thus, bug reproduction is a situation where automatic test generation could help developers.

Despite this, due to the difficulty of natural language processing, automatic bug reproduction has been mainly limited to crash bugs, where the expected behavior is clear - the program should not crash. This actually leaves some simple cases (from the human perspective) difficult to automatically handle for machines. For example, consider the following [bug report](https://issues.apache.org/jira/browse/MATH-370) from Commons Math: 

> **NaN in "equals" methods**
>
> In "MathUtils", some "equals" methods will return true if both argument are NaN.
> Unless I'm mistaken, this contradicts the IEEE standard.
> 
> If nobody objects, I'm going to make the changes.

In this case, it is clear what the expected behavior is to a human (`equals` methods should return false if both arguments are NaN). However, when trying to deal with this report using a rule-based algorithm, you immediately face two difficulties. First, the report does not explicitly spell out the expected behavior, so it is difficult to extract the expected behavior. Second, even if you get the oracle, all the code references in the report are disconnected, so it can be difficult to automatically write a test based on the available information. It is likely such difficulties that made automatic bug reproduction outside of crash bugs difficult.

## Approach and Results

It is no longer news, but one of the observations of our paper is that large language models (LLMs) do quite a good job of overcoming these challenges. After all, LLMs are good enough at natural language processing to the extent that they are capable of [explaining humor](https://ai.googleblog.com/2022/04/pathways-language-model-palm-scaling-to.html), and they are capable of generating code as well. Consequently, they are a natural solution to the test reproduction problem.

However, to build a tool that is actually useful to developers, simply generating many solutions from an LLM is not enough - we can't hand a hundred candidate reproducing tests to a developer and say "job done". Thankfully, for automatic bug reproduction, we find features that can help reduce this manual inspection cost. For example, a bug reproducing test should actually fail on the current, buggy version of the code, which allows us to automatically filter a portion of the unhelpful solutions. We bundled this heuristic, along with others, as an automated post-processing pipeline, specialized for handling bug reproduction.

With LLMs and the post-processing, which together we group in the tool `LIBRO`, we could both reproduce **one-third** of all of the bugs in the Defects4J benchmark, which is a substantially better performance than the baselines that we compared against. Furthermore, with the postprocessing pipeline we could achieve a precision of up to 0.85, reducing developer hassle with false positives.[^1]

## Perspective A Year After Submitting The Paper

Since then, there has been a substantial amount of work on test generation using large language models. Testing does seem to be one of the most natural applications of LLMs for software engineering - while LLMs have the tendency to be wrong, it is okay for tests to be "wrong", as this may actually lead to the testing of interesting behavior, as we note in [Robert et al.](https://arxiv.org/abs/2306.05152). 

If I can leave one takeaway for any readers who have come this far, it is that while LLM results are certainly valuable, the generation results of LLMs can be much more valuable when they are evaluated by a rule-based system, as once this is done, it is easier for humans to have trust in such results. This is a theme of much of my subsequent work on LLMs: I argued this point in my [GI workshop position paper](/al-folio/publications/#Kang2023lg), and explicitly made this a central theme of my [Automatic Scientific Debugging](/al-folio/publications/#kang2023explainable).

### Footnotes

[^1]: In [prior work](https://dl.acm.org/doi/10.1145/2931037.2931051) on fault localization, most developers reported that they would be satisfied with an FL tool with an accuracy of ca. 90%, so it is feasible that developers would be satisfied with the precision of `LIBRO`.