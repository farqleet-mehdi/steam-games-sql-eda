# Project notes: talking points, limitations and corrections

These notes accompany the Steam Games SQL and EDA project. They record how the results can be explained, where the analysis is weak, and which predictions turned out wrong. They are kept in the repository on purpose: the limits of an analysis are part of the analysis.

## 1. The project in one minute

I took a raw catalogue of 239,664 Steam store listings, cleaned and validated it, and exported a relational SQLite database of 150,279 games (11 tables, enforced foreign keys, 54 automated checks passing). I then answered six questions in SQL, through DuckDB on a read-only connection, with pandas and matplotlib used only to display results. Each question defines its terms first, checks coverage and sample sizes before computing anything, and ends with a conclusion that lists its limits. Where a pooled gap could come from a different mix of release years, I repeated the comparison inside release-year cohorts.

The main lesson of the project: most of the interesting fields are sparsely or unevenly populated, so a large part of the work was deciding what each number can and cannot say.

## 2. Talking points by question

**Q1: Free games by release year.**
- Finding: free games rise from about 2-3% of releases (2006-2010) to 14.0-15.6% for each year 2019-2024 (12.2% in partial 2025).
- Decision: the `is_free` flag is the definition, not the Free to Play tag. 30.0% of free games (5,692 of 18,994) lack the tag, so the tag would undercount.
- Limit: the tag and flag shares converge in 2024-25, and this comes only from free games (tagged 57-61% of free games in 2021-23, 86.9% in 2024, 97.1% in 2025). The cause is untested; a change in tagging practice and a change in the mix of games are both candidates.

**Q2: Genre share by year.**
- Finding: Indie ranks first every year 2015-2024 (67.7-77.3%). Casual overtakes Action in 2018 and stays in a 42.4-46.2% band. Action falls from 44.3% (2017) to 38.9% (2024).
- Limit: a game counts once per genre, so shares add to more than 100%. The size of Casual's gain depends on the comparison window (about +9.4 points with 2015-17 against 2022-24, roughly +4.5 with 2016-18 as the early period). The Free to Play change is a labelling effect and is ignored.

**Q3: Publisher catalogue size and reception.**
- Finding: 76.1% of 80,733 publishers have one game; 470 have 21 or more. The share of a publisher's games with recommendations reported rises from 5.7% (1 game) to 32.8% (21+ games). Among games that report, the median count is flat (322-353), so the difference is in how many games report, not in how large the counts are.
- Control: the gradient holds in all four release-year cohorts, with a 1-game to 21+ gap of 29.0, 28.7, 22.9 and 20.7 points from oldest to newest. Part of the pooled gap is composition: 49.3% of one-game publisher pairs are from 2023-25, against 35.3% for 21+ publishers.
- Limit: recommendations measure reach, not quality. The Metacritic comparison rests on small, selected samples (43, 142 and 129 publishers) and was not tested.

**Q4: Price tier and Metacritic score.**
- Finding: among 3,742 scored paid games, the median score rises with list price: 69, 72, 76, 78, 80.
- Selection: only 2.8% of games have a score, and the scored share runs from 0.6% (free) to 15.6% (USD 20 to 39.99). Each tier's scored games are a differently selected group, so the result describes scored games only.
- Control: scored cheap games skew old (70% of scored Under USD 10 games are from 2015 or earlier, against 14% of scored USD 20+ games). The Under USD 10 to USD 20+ median gap is 6.5 points (2011-2015) and 6.0 (2016-2019), but only 1 point in 2020-2025. The reason is untested.
- Limit: `price_usd` is the price at the scrape date, not at release.

**Q5: Platform support over time.**
- Finding: Windows is at 99.7-100% in every year. Mac and Linux peak for 2013 releases (46.6% and 39.0%) and fall to 11.9% and 10.0% for 2025 releases.
- Key check: the fall is a growing denominator, not fewer games. Mac-supporting games rose from 216 (2013) to 2,569 (2024) while all games rose from 464 to 20,203. Between 2018 and 2021 Mac counts were roughly flat (1,688 to 1,645) while the total grew 49%.
- Limit: support is the flag at scrape time, so later ports count under the original release year. The cause of the peak is untested.

**Q6: Discounts at the scrape date.**
- Finding: on 2025-09-29, 10.5% of games with a price (9,457 of 90,311) were on sale. Among USD-priced games the share rises with list price, from 9.4% to 15.7%, and the pattern holds in the two largest cohorts (gap 4.6 points for 2016-2019, 6.5 for 2020-2025).
- Limit: one day only. It says nothing about how often a game is discounted. Free games hold no discount value, so they cannot be assessed.

## 3. Weaknesses an interviewer may probe

**31% of paid games have no price.** 40,974 of 131,285 paid games have no price, concentrated in recent listings. The cause is undetermined, and it is not explained by unreleased status. Every price analysis therefore leans toward older listings. This is the biggest limit in the project.

**Sparse reception data.** Metacritic covers 4,166 games (2.8%) and recommendations cover 13.2%. Both are subsample analyses, and the covered games are not a random sample.

**Snapshot data.** Price, discount and platform flags are values at the scrape date. None of the results describe launch conditions or change over a game's life.

**No causal claims.** Every result is an association in observational data. Explanations that were not tested are labelled as hypotheses in the notebook.

**Entity resolution is shallow.** Developers and publishers were merged only on trimmed, lowercased names. Other spelling variants still split one company into several, which affects Q3. No name-based deduplication was applied to games; `appid` is the key.

**Missing release years.** 23.5% of games have no valid release year (including dates set to null as outside 1990-01-01 to 2025-09-29), so year trends exclude them. Those games are less often free (8.9% against 13.8%), so the exclusion is not neutral. 2025 is a partial year.

**Manual raw-database step.** The raw SQLite database is built by hand in DB Browser, which is not reproducible by script. A script that builds it from the CSVs is on the improvement list.

**No automated tests.** The cleaning logic is checked by 54 in-notebook validation checks, but there are no tests outside the notebook.

**Small cells.** Some cohort cells are tiny (12 games in the Up to 2010 and USD 20+ Metacritic cell, 15 in the same discount cell). I did not draw conclusions from them.

## 4. Questions I should be ready for

**Why use the junction table instead of the raw platform flags?** The raw `supports_mac` and `supports_linux` flags were true for about 99.9% of apps, while the platform table gave about 23% and 17%. They contradicted each other, so I treated the junction table as the source of truth and dropped the flags.

**Why `is_free` and not the Free to Play tag?** The tag misses 30.0% of free games and also appears on 423 games flagged as paid. The flag is complete and consistent, so it carries the definition.

**Why medians and quartiles for scores?** Scores are bounded and left-skewed (every tier mean sits below its median), and quartiles show how much neighbouring tiers overlap.

**Why a release-year control?** Pooled gaps can come from a different mix of release years rather than from the variable of interest. In Q3 and Q4 part of the pooled gap was composition; in Q6 it was not. Testing it rather than assuming it is the point.

**Why SQL first?** The queries are the analysis. Keeping pandas and matplotlib to display only means every number in the notebook traces to a query that can be read and rerun.

**Why DuckDB over a SQLite file?** DuckDB reads the SQLite file directly and provides analytical functions such as `median` and `quantile_cont`, which Q4 uses. (Edit this answer into your own words before an interview.)

**What would you do next?** Investigate the missing prices first, since they limit every price result. Then test the open explanations (the 2024-25 tag convergence, the 2013 Mac and Linux peak) and add tests for the cleaning logic.

## 5. Corrections log

Predictions and statements that were wrong, and what the data showed. They are listed because checking predictions against output was the working method.

**Notebook 01 (cleaning)**
- Predicted 41 validation checks; the real count is 54.
- Predicted all ID columns would be text; genre and category IDs were integers.
- A data-dictionary entry said currency is USD or NULL. That was wrong: the 1,368 non-USD games keep their currency and have a NULL price. Fixed.

**Q1**
- Predicted under 10% of free games lack the Free to Play tag; the real figure is 30.0%.
- Predicted a stable tag-to-flag ratio with no trend; the ratio drifts and jumps in 2024.

**Q2**
- Described Indie as flat; its share fell 2.8 points. The early shape of RPG was misstated and corrected.

**Q3**
- Predicted Metacritic coverage under 5% per publisher band; the real range is 0.8-8.9%.
- Quoted 5,299 games with a Metacritic score; that figure counted all app types. The correct figure for games is 4,166.

**Q4**
- Predicted the score gap between lowest and highest tier would be under 8 points; it is 11.
- Predicted the free tier would sit lowest; its median (74.5) is above the two cheapest paid tiers.
- Assumed cheap games skew toward recent listings. Among scored games the reverse holds.
- An interquartile range of 15 points was attached to the wrong tier in a first draft of the conclusion (it belongs to Under USD 5, not USD 5 to 9.99). Fixed.

**Q5**
- Used a wrong column name in the first query (`id` instead of `platform_id`), taken from a note and not checked against the schema.
- Predicted Mac at 10-30% and Linux at 5-20%; both peak far higher (46.6% and 39.0% in 2013).
- Predicted no clean trend; there is a clear rise to a 2013 peak and a steady fall.

**Q6**
- Predicted older cohorts would have a higher share on sale; there is no age pattern.
- Predicted the deepest discount would be 90% or less; it is 95%.
- A cohort share was stated as 71% for Under USD 10 games in a first draft; the correct figure is 70%. Fixed.
