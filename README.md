# NHL Goalie Height Analysis

## Description

This project explores whether a goaltender's height meaningfully influences their performance in the NHL.
I analyzed over two decades of goalie statistics, prospect rankings, and team defensive metrics using publicly available NHL data. 
I examined trends in goalie height by season and birth country, and modelled performance using save percentage (SV%) and goals against 
average (GAA) as outcome variables.

To assess the relationship between height and performance, I used a combination of generalized additive models (GAMs), decision trees, 
and random forests in R. While NHL teams have increasingly favoured taller goalies, the modelling results show that height alone is 
not a strong predictor of SV% or GAA. Instead, team context and other individual factors play a larger role.

The analysis combines data wrangling, visualization, and interpretable machine learning to provide a data-driven perspective on 
goaltender evaluation and scouting trends.

The full report can be read [here](https://github.com/rayceramsay/nhl-goalie-height-analysis/blob/main/04_final_report/04_final_report.pdf).

## Website

Along with the original Rmd notebooks found in this repository, you can explore the EDA, modelling, interactive report, and additional visualizations
online through my [GitHub pages website](https://rayceramsay.github.io/nhl-goalie-height-analysis).
