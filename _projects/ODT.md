---
name: Optimal Decision Trees (ODT)
tools: [R]
category: Package
domain: Methods/Explainable AI
image:
description: R package built with my team based on the algorithm described in the paper “Precision oncology, a review to assess interpretability in several explainable methods”.
date: 2021-06-01
---
# Optimal Decision Trees (ODT)

This package was built with my team based on the algorithm described in the paper “Precision oncology, a review to assess interpretability in several explainable methods.” ODT builds an optimal decision tree by solving an optimization problem at each split, assigning treatments that optimize a chosen sensitivity measure while respecting group size constraints. The method leverages clinical/omics and drug sensitivity data to recommend robust, individualized therapies.

- Focus: interpretable, optimization-driven treatment recommendations
- Stack: R, optimization, clinical/omics data integration

{% include elements/button.html link="https://www.rdocumentation.org/packages/ODT/versions/1.0.0" text="ODT on RDocumentation" %}
