# Analysis 1: Cross-Platform Volume and Surge Detection (MEIU22)

This section examines cross-platform election-related activity in the MEIU22 dataset during the 2022 U.S. midterm period. The goal is to understand (1) which platforms contribute the largest share of activity and how volume changes over time (RQ1), and (2) whether the strongest activity surges are synchronized across platforms or staggered (RQ2).

This analysis uses a reproducible pandas workflow built from a tidy item-level dataset and daily aggregation. It combines descriptive volume summaries with z-score-based surge detection to identify unusually high-activity days while accounting for differences in baseline platform size.
