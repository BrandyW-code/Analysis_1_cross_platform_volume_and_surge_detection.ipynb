# Analysis 1: Cross-Platform Volume and Surge Detection (MEIU22)

This section examines cross-platform election-related activity in the MEIU22 dataset during the 2022 U.S. midterm period. The goal is to understand (1) which platforms contribute the largest share of activity and how volume changes over time (RQ1), and (2) whether the strongest activity surges are synchronized across platforms or staggered (RQ2). This analysis uses a reproducible pandas workflow built from a tidy item-level dataset and daily aggregation. It combines descriptive volume summaries with z-score-based surge detection to identify unusually high-activity days while accounting for differences in baseline platform size.

# Data Assembly 

For this part of the project, I used the assembled MEIU22 tidy index file (meiu22_posts_index_tidy_labeled.csv), which standardizes records across platforms into a pandas-friendly format. In this file, each row represents one item (e.g., a post, comment, or tweet identifier), and includes variables such as platform, collection_type, item_id, url, created_at_utc, source_file, and a derived day field used for time-based aggregation. This analysis includes Facebook, Instagram, Reddit, and Twitter in the overall platform totals. However, because the Twitter keyword sample in the tidy file does not include usable day-level timestamps, the daily time-series and surge analyses in this section are most comparable to those for Facebook, Instagram, and Reddit.

| platform  | items  | percent_share |
|-----------|--------|---------------|
| facebook  | 314564 | 0.558429      |
| reddit    | 161770 | 0.287182      |
| twitter   | 50000  | 0.088762      |
| instagram | 36968  | 0.065627      |

| platform  | total_rows | missing_day_rows | percent_missing_day |
|-----------|------------|------------------|---------------------|
| facebook  | 314564     | 0                | 0.0                 |
| instagram | 36968      | 0                | 0.0                 |
| reddit    | 161770     | 0                | 0.0                 |
| twitter   | 50000      | 50000            | 1.0                 |

| platform  | min_day     | max_day     | n_distinct_days |
|-----------|-------------|-------------|-----------------|
| facebook  | 2022-10-01  | 2022-12-25  | 86              |
| instagram | 2022-10-01  | 2022-12-25  | 86              |
| reddit    | 2022-10-01  | 2022-12-25  | 86              |

| platform  | day        | count |
|-----------|------------|-------|
| facebook  | 2022-10-01 | 1440  |
| instagram | 2022-10-01 | 144   |
| reddit    | 2022-10-01 | 54    |
| facebook  | 2022-10-02 | 1365  |
| instagram | 2022-10-02 | 126   |
| reddit    | 2022-10-02 | 49    |
| facebook  | 2022-10-03 | 2400  |
| instagram | 2022-10-03 | 285   |
| reddit    | 2022-10-03 | 79    |
| facebook  | 2022-10-04 | 2783  |
| reddit    | 2022-12-22 | 355   |
| facebook  | 2022-12-23 | 554   |
| instagram | 2022-12-23 | 43    |
| reddit    | 2022-12-23 | 299   |
| facebook  | 2022-12-24 | 327   |
| instagram | 2022-12-24 | 29    |
| reddit    | 2022-12-24 | 292   |
| facebook  | 2022-12-25 | 226   |
| instagram | 2022-12-25 | 21    |
| reddit    | 2022-12-25 | 275   |







# Understanding the Dataset (Time Coverage and Quality Checks)
Before analyzing trends, I verified whether each platform had usable day-level coverage. This step is important because the daily volume analysis and z-score surge detection depend on valid date values in the day column. I also checked platform-level missingness to identify where time-series comparisons are valid and where they are limited. The checks show a clear platform-specific constraint: Facebook, Instagram, and Reddit have complete day-level coverage in the working period, while the Twitter keyword sample has missing day values for all rows in the tidy file. Rather than ignoring this issue, I treat it as a measurement limitation and proceed with daily comparisons for the platforms that support valid time-series analysis.




# Analysis 1A: Cross-Platform Volume Over Time 
To address RQ1, I first summarized the total number of items contributed by each platform in the assembled tidy dataset. This provides a baseline view of platform contribution and helps contextualize later surge analyses. Because platform totals can reflect both substantive activity and collection design, I report them as a descriptive starting point rather than a direct measure of platform influence. Next, I aggregated the item-level data into daily counts by platform (platform × day × count) for records with usable day values. This daily time series makes it possible to compare how activity changes over time and to identify visually whether volume intensifies around the election period before applying a formal surge metric.

|index|platform|items|percent\_share|
|---|---|---|---|
|0|facebook|314564|0\.558428693666985|
|1|reddit|161770|0\.28718165389080813|
|2|twitter|50000|0\.08876233352624348|
|3|instagram|36968|0\.06562731891596338|

| index | platform  | day        | count |
|------:|-----------|------------|------:|
| 0     | facebook  | 2022-10-01 | 1440  |
| 1     | instagram | 2022-10-01 | 144   |
| 2     | reddit    | 2022-10-01 | 54    |
| 3     | facebook  | 2022-10-02 | 1365  |
| 4     | instagram | 2022-10-02 | 126   |
| 5     | reddit    | 2022-10-02 | 49    |
| 6     | facebook  | 2022-10-03 | 2400  |
| 7     | instagram | 2022-10-03 | 285   |
| 8     | reddit    | 2022-10-03 | 79    |
| 9     | facebook  | 2022-10-04 | 2783  |

| index | platform  | day        | count |
|------:|-----------|------------|------:|
| 248   | reddit    | 2022-12-22 | 355   |
| 249   | facebook  | 2022-12-23 | 554   |
| 250   | instagram | 2022-12-23 | 43    |
| 251   | reddit    | 2022-12-23 | 299   |
| 252   | facebook  | 2022-12-24 | 327   |
| 253   | instagram | 2022-12-24 | 29    |
| 254   | reddit    | 2022-12-24 | 292   |
| 255   | facebook  | 2022-12-25 | 226   |
| 256   | instagram | 2022-12-25 | 21    |
| 257   | reddit    | 2022-12-25 | 275   |






<img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/e079945e-34ae-4989-ab30-1c540477be72" />


# Interpreting the Results (RQ1)
The platform totals show that activity in the working MEIU22 tidy dataset is highly uneven across platforms. Facebook contributes the largest share (314,564 items; ~55.8%), followed by Reddit (161,770; ~28.7%), Twitter (50,000; ~8.9%, keyword sample), and Instagram (36,968; ~6.6%). This confirms that the assembled dataset is platform-skewed and that raw-volume comparisons should be interpreted as reflecting both the empirical signal and measurement conditions.

The daily counts further show that platform attention is dynamic rather than stable. For the platforms with valid day-level coverage (Facebook, Instagram, Reddit), daily activity increases sharply during a narrow election-week window, which supports moving to a standardized surge analysis in the next step. Because the Twitter keyword sample lacks usable day values in the tidy file, the strongest day-level comparisons in this section are limited to Facebook, Instagram, and Reddit.

# Analysis 1B: Surge Detection and Shared Surge Windows (RQ2)
To address RQ2, I use z-scores to identify unusually high-volume days within each platform. A z-score compares each day’s count to that platform’s average daily count and standard deviation, which makes it possible to compare surge intensity across platforms with very different baseline sizes. In this project, z-scores are used as a descriptive tool for surge detection, not as a causal test.

After calculating z-scores for each platform-day, I identify the top surge day for each platform and the top five surge days. I then compare those top surge sets to determine whether the strongest surges occur on the same dates across platforms (synchronized) or within the same short event window but on different dates (staggered / lead-lag pattern).

| index | platform  | day        | count | z        | days_from_earliest_peak |
|------:|-----------|------------|------:|---------:|------------------------:|
| 0     | instagram | 2022-11-08 |  5616 | 7.241904 | 0                       |
| 1     | reddit    | 2022-11-09 | 19804 | 6.218456 | 1                       |
| 2     | facebook  | 2022-11-08 | 27073 | 5.453272 | 0                       |


| index | platform  | day        | count | z        |
|------:|-----------|------------|------:|---------:|
| 0     | facebook  | 2022-11-08 | 27073 | 5.453272 |
| 1     | facebook  | 2022-11-09 | 23499 | 4.620910 |
| 2     | facebook  | 2022-11-07 | 15811 | 2.830423 |
| 3     | facebook  | 2022-11-10 | 10378 | 1.565111 |
| 4     | facebook  | 2022-11-04 |  9235 | 1.298913 |
| 5     | instagram | 2022-11-08 |  5616 | 7.241904 |
| 6     | instagram | 2022-11-09 |  3248 | 3.935238 |
| 7     | instagram | 2022-11-07 |  1947 | 2.118527 |
| 8     | instagram | 2022-11-04 |  1170 | 1.033528 |
| 9     | instagram | 2022-11-03 |   982 | 0.771005 |
| 10    | reddit    | 2022-11-09 | 19804 | 6.218456 |
| 11    | reddit    | 2022-11-08 | 12258 | 3.600334 |
| 12    | reddit    | 2022-11-10 | 10712 | 3.063942 |
| 13    | reddit    | 2022-11-13 |  6728 | 1.681674 |
| 14    | reddit    | 2022-11-07 |  6551 | 1.620263 |


| index | day        | num_platforms | platforms_present            |
|------:|------------|--------------:|------------------------------|
| 0     | 2022-11-07 | 3             | facebook, instagram, reddit  |
| 1     | 2022-11-08 | 3             | facebook, instagram, reddit  |
| 2     | 2022-11-09 | 3             | facebook, instagram, reddit  |
| 3     | 2022-11-04 | 2             | facebook, instagram          |
| 4     | 2022-11-10 | 2             | facebook, reddit             |


| index | platform  | day        | count | z        |
|------:|-----------|------------|------:|---------:|
| 0     | facebook  | 2022-11-04 |  9235 | 1.298913 |
| 1     | instagram | 2022-11-04 |  1170 | 1.033528 |
| 2     | facebook  | 2022-11-07 | 15811 | 2.830423 |
| 3     | instagram | 2022-11-07 |  1947 | 2.118527 |
| 4     | reddit    | 2022-11-07 |  6551 | 1.620263 |
| 5     | facebook  | 2022-11-08 | 27073 | 5.453272 |
| 6     | instagram | 2022-11-08 |  5616 | 7.241904 |
| 7     | reddit    | 2022-11-08 | 12258 | 3.600334 |
| 8     | facebook  | 2022-11-09 | 23499 | 4.620910 |
| 9     | instagram | 2022-11-09 |  3248 | 3.935238 |
| 10    | reddit    | 2022-11-09 | 19804 | 6.218456 |
| 11    | facebook  | 2022-11-10 | 10378 | 1.565111 |
| 12    | reddit    | 2022-11-10 | 10712 | 3.063942 |


# Interpreting the Results (RQ2)
The surge analysis shows that the strongest platform surges cluster around Election Day week, but the peaks are not perfectly synchronized. Facebook and Instagram both reach their top z-score surge on 11-08-2022, while Reddit reaches its top z-score surge on 11-09-2022, indicating a short lag rather than a fully simultaneous cross-platform spike.

The top-surge summaries strengthen this interpretation. Several dates appear in the top five surge days across all three comparable platforms, especially 11-07-2022, 11-08-2002, and 11-09-2022. Two additional dates (11-04-2022 and 11-10-2022) appear in the top surge sets on multiple platforms. Together, these results suggest a shared, concentrated attention window around the election period with platform-specific timing differences consistent with a lead/lag pattern. The results in this section are descriptive and do not establish causality. However, they provide a strong foundation for later lead-lag testing and event-aligned comparisons in the project's next steps.





<img width="1189" height="590" alt="image" src="https://github.com/user-attachments/assets/dbe0593e-719a-4b1c-8fff-fdc095d109bf" />

# Figures and Tables Used in Analysis 1
This analysis is supported by two main figures and several summary tables. Figure 1 visualizes daily election-related item volume across comparable platforms (Facebook, Instagram, Reddit), while Figure 2 visualizes the top surge days by platform using z-scores. Together, these figures support both the baseline-volume interpretation (RQ1) and the shared-window/lead-lag-surge interpretation (RQ2).

# Conclusion 
This section establishes the core cross-platform volume and surge patterns in the MEIU22 dataset. The results show that platform activity is uneven in magnitude, that election-week surges cluster in a shared window, and that the strongest platform peaks are not perfectly synchronized (with Reddit peaking one day after Facebook and Instagram in the top z-score results).

These findings support the project's broader suspicion that cross-platform attention during major political events is both shared and platform-specific. The next analysis builds on this by introducing a supporting measurement comparison (candidate-based vs. keyword-based collection) to show how collection strategy can materially change what "activity" looks like in the same election-week window.
