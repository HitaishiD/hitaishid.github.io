---
layout: post
title: AI Study Buddy Overview
category: Study_Buddy
---

## Overview

Students often encounter a range of frustrations related to their classes, be it at high school level or university level. frustrations related to the learning process itself are the most impactful and demotivating for students. The core learning-related frustrations are illustrated below. 

<div align="center">
  <img src="{{ site.baseurl }}/images/study-buddy/problems.png" alt="Learning-related problems faced by students">
</div>

This presents an opportunity to use AI tools to solve these problems. The AI-powered Study Buddy developed in this project aims to address these pain points as illustrated below. 

<div align="center">
  <img src="{{ site.baseurl }}/images/study-buddy/problems_solutions.png" alt="Learning-related problems and proposed solutions">
</div>

## Explanation of how the study buddy works from the user's perspective

The diagram below illustrates the AI Study Buddy from the user's perspective. 

<div align="center">
  <img src="{{ site.baseurl }}/images/study-buddy/webapp_diagram.png" alt="Webapp diagram">
</div>


## Tools/Concepts used 

RAG (textual), LLM inference, prompting, automatic question generation...

## Areas of Improvement

  * **Multi-modal RAG** : Since most study materials contain valuable visual aids in the form of diagrams, images, tables and charts, incorporating multi-modal RAG would improve the quality of the context available to the LLM when generating answers and quiz questions, leading to more complete and accurate retrieval.
  
  * **Prompt refinement** : Iterating on more effective prompt strategies could improve both retrieval quality and answer generation, ensuring outputs are more aligned with user expectations.
  
  * **Support for multiple study materials**: Students often use several sources of information, such as lecture notes, their personal notes, textbooks, while studying. Thus, allowing the upload and merging of several files would provide more comprehensive context for the LLM on one hand, but on the other hand, more importantly, students would not have to cross-reference multiple sources of information manually. 
  
  * **Improvement of retrieval pipeline**: Alternative retrieval algorithms could be tested and potentially a combination of methods can be used to enhance the relevance of retrieved information and strengthen the perdormance of the RAG pipeline.
  
  * **Improvement on question generation**: To ensure that high-quality and pedagogically sound questions are being generated, the question generation process could be further improved with prompt engineering, fine-tuning, or evaluation mechanisms.



