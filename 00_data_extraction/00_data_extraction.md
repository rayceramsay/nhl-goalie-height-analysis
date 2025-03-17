The Impact of Height on NHL Goalies (Data Extraction)
================
Rayce Ramsay
2025-03-13

## Resources

- [(Unofficial) NHL API
  Docs](https://github.com/Zmalski/NHL-API-Reference?tab=readme-ov-file)
- [NHL Goalie Bios API
  Example](https://api.nhle.com/stats/rest/en/goalie/bios?limit=-1&start=&sort=&cayenneExp=seasonId=20232024)
- [Official NHL Goalie
  Stats](https://www.nhl.com/stats/goalies?report=bios&reportType=season&seasonFrom=20242025&seasonTo=20242025&gameType=2&sort=a_goalieFullName&page=0&pageSize=100)

## Setup

``` r
# Load libraries
library(tidyverse)
library(httr2)
```

``` r
# Base URLS for NHL API
NHL_API_BASE_URL = "https://api.nhle.com/stats/rest/"
NHLWEB_API_BASE_URL = "https://api-web.nhle.com/"
```

## Get NHL Goalie Season Data

``` r
# Get season IDs to query NHL API for starting with the 2000-2001 season
START_YEARS = 2000:2024
season_ids = NULL
for (start_year in START_YEARS) {
  season_ids = c(season_ids, paste0(start_year, start_year + 1))
}

season_ids
```

    ##  [1] "20002001" "20012002" "20022003" "20032004" "20042005" "20052006"
    ##  [7] "20062007" "20072008" "20082009" "20092010" "20102011" "20112012"
    ## [13] "20122013" "20132014" "20142015" "20152016" "20162017" "20172018"
    ## [19] "20182019" "20192020" "20202021" "20212022" "20222023" "20232024"
    ## [25] "20242025"

``` r
# Create URLs to fetch NHL goalie info by season
summary_urls = paste0(NHL_API_BASE_URL, "en/goalie/summary?limit=-1&start=&sort=&cayenneExp=seasonId=", season_ids)
bio_urls = paste0(NHL_API_BASE_URL, "en/goalie/bios?limit=-1&start=&sort=&cayenneExp=seasonId=", season_ids)

# Create http request objects
bio_reqs = lapply(bio_urls, httr2::request)
summary_reqs = lapply(summary_urls, httr2::request)

# Perform http requests
bio_resps = req_perform_sequential(bio_reqs)
```

    ## Iterating ■■■■■■■■ 24% | ETA: 3sIterating ■■■■■■■■■ 28% | ETA: 3sIterating
    ## ■■■■■■■■■■■■ 36% | ETA: 3sIterating ■■■■■■■■■■■■■ 40% | ETA: 2sIterating
    ## ■■■■■■■■■■■■■■ 44% | ETA: 2sIterating ■■■■■■■■■■■■■■■■■ 52% | ETA: 2sIterating
    ## ■■■■■■■■■■■■■■■■■■ 56% | ETA: 2sIterating ■■■■■■■■■■■■■■■■■■■ 60% | ETA:
    ## 2sIterating ■■■■■■■■■■■■■■■■■■■■■ 68% | ETA: 1sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■ 72% | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■ 76% |
    ## ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■■■ 84% | ETA: 1sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■ 88% | ETA: 0sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 92% | ETA: 0sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 96% | ETA: 0s

``` r
summary_resps = req_perform_sequential(summary_reqs)
```

    ## Iterating ■■■■■■■■■■■■ 36% | ETA: 2sIterating ■■■■■■■■■■■■■■ 44% | ETA:
    ## 2sIterating ■■■■■■■■■■■■■■■ 48% | ETA: 2sIterating ■■■■■■■■■■■■■■■■■■■■ 64% |
    ## ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■ 72% | ETA: 1sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■ 80% | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■■■ 84%
    ## | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 92% | ETA: 0sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 96% | ETA: 0s

``` r
# Extract bios and summaries from requests
bio_data = map2(bio_resps, season_ids, function(resp, season) {
  goalies = resp_body_json(resp)$data
  lapply(goalies, function(goalie) {
    goalie$seasonId = season
    map(goalie, ~ifelse(is.null(.), NA, .))
  })
})
summary_data = map(summary_resps, function(resp) {
  goalies = resp_body_json(resp)$data
  lapply(goalies, function(goalie) {
    goalie$seasonId = as.character(goalie$seasonId)
    map(goalie, ~ifelse(is.null(.), NA, .))
  })
})

# Convert extracted data to tibbles
bios = list_rbind(map(list_c(bio_data), ~as_tibble(.x)))
summaries = list_rbind(map(list_c(summary_data), ~as_tibble(.x)))

dim(bios)
```

    ## [1] 2636   24

``` r
head(bios)
```

    ## # A tibble: 6 × 24
    ##   birthCity  birthCountryCode birthDate birthStateProvinceCode currentTeamAbbrev
    ##   <chr>      <chr>            <chr>     <chr>                  <chr>            
    ## 1 Toronto    CAN              1965-01-… ON                     <NA>             
    ## 2 Montréal   CAN              1970-06-… QC                     <NA>             
    ## 3 Gresham    USA              1979-06-… OR                     <NA>             
    ## 4 Surahammar SWE              1971-02-… <NA>                   <NA>             
    ## 5 Saskatoon  CAN              1974-05-… SK                     <NA>             
    ## 6 Windsor    CAN              1967-01-… ON                     <NA>             
    ## # ℹ 19 more variables: draftOverall <int>, draftRound <int>, draftYear <int>,
    ## #   firstSeasonForGameType <int>, gamesPlayed <int>, goalieFullName <chr>,
    ## #   height <int>, isInHallOfFameYn <chr>, lastName <chr>, losses <int>,
    ## #   nationalityCode <chr>, otLosses <int>, playerId <int>, shootsCatches <chr>,
    ## #   shutouts <int>, ties <int>, weight <int>, wins <int>, seasonId <chr>

``` r
tail(bios)
```

    ## # A tibble: 6 × 24
    ##   birthCity  birthCountryCode birthDate birthStateProvinceCode currentTeamAbbrev
    ##   <chr>      <chr>            <chr>     <chr>                  <chr>            
    ## 1 Lugnvik    SWE              1993-07-… <NA>                   OTT              
    ## 2 Richmond … CAN              1993-07-… ON                     STL              
    ## 3 Bern       CHE              2000-05-… <NA>                   VGK              
    ## 4 St-Hyacin… CAN              1992-03-… QC                     NYR              
    ## 5 Milford    USA              1986-01-… CT                     NYR              
    ## 6 Thunder B… CAN              1994-05-… ON                     TOR              
    ## # ℹ 19 more variables: draftOverall <int>, draftRound <int>, draftYear <int>,
    ## #   firstSeasonForGameType <int>, gamesPlayed <int>, goalieFullName <chr>,
    ## #   height <int>, isInHallOfFameYn <chr>, lastName <chr>, losses <int>,
    ## #   nationalityCode <chr>, otLosses <int>, playerId <int>, shootsCatches <chr>,
    ## #   shutouts <int>, ties <int>, weight <int>, wins <int>, seasonId <chr>

``` r
dim(summaries)
```

    ## [1] 2257   23

``` r
head(summaries)
```

    ## # A tibble: 6 × 23
    ##   assists gamesPlayed gamesStarted goalieFullName     goals goalsAgainst
    ##     <int>       <int>        <int> <chr>              <int>        <int>
    ## 1       0           8            7 Stephane Fiset         0           19
    ## 2       1          79           78 Tommy Salo             0          194
    ## 3       1          53           53 Ron Tugnutt            0          127
    ## 4       3          80           80 Dominik Hasek          0          166
    ## 5       0          38           35 Damian Rhodes          0          116
    ## 6       0          48           46 John Vanbiesbrouck     0          126
    ## # ℹ 17 more variables: goalsAgainstAverage <dbl>, lastName <chr>, losses <int>,
    ## #   otLosses <int>, penaltyMinutes <int>, playerId <int>, points <int>,
    ## #   savePct <dbl>, saves <int>, seasonId <chr>, shootsCatches <chr>,
    ## #   shotsAgainst <int>, shutouts <int>, teamAbbrevs <chr>, ties <int>,
    ## #   timeOnIce <int>, wins <int>

``` r
tail(summaries)
```

    ## # A tibble: 6 × 23
    ##   assists gamesPlayed gamesStarted goalieFullName    goals goalsAgainst
    ##     <int>       <int>        <int> <chr>             <int>        <int>
    ## 1       1          26           25 John Gibson           0           66
    ## 2       1          14           14 Frederik Andersen     0           30
    ## 3       0          15           11 Aleksei Kolosov       0           45
    ## 4       0          30           30 Charlie Lindgren      0           78
    ## 5       3          40           40 Dustin Wolf           0          102
    ## 6       0           1            0 Pheonix Copley        0            2
    ## # ℹ 17 more variables: goalsAgainstAverage <dbl>, lastName <chr>, losses <int>,
    ## #   otLosses <int>, penaltyMinutes <int>, playerId <int>, points <int>,
    ## #   savePct <dbl>, saves <int>, seasonId <chr>, shootsCatches <chr>,
    ## #   shotsAgainst <int>, shutouts <int>, teamAbbrevs <chr>, ties <int>,
    ## #   timeOnIce <int>, wins <int>

``` r
# Select columns from summaries to keep for joining
summaries_limited = summaries |>
  select(!c(goalieFullName, lastName, shootsCatches)) # only keep summary/stats (non-bio) columns

# Join goalie biographies and summaries into one tibble
goalies = bios |>
  select(!c(currentTeamAbbrev, firstSeasonForGameType, gamesPlayed, losses, otLosses, shutouts, ties, wins)) |> # only keep bio (non-summary) columns
  distinct() |> # ignore duplicates since data has separate rows for playoffs and regular season
  inner_join(summaries_limited, by = join_by(seasonId, playerId))
  
dim(goalies)
```

    ## [1] 2257   34

``` r
head(goalies)
```

    ## # A tibble: 6 × 34
    ##   birthCity  birthCountryCode birthDate  birthStateProvinceCode draftOverall
    ##   <chr>      <chr>            <chr>      <chr>                         <int>
    ## 1 Toronto    CAN              1965-01-14 ON                               69
    ## 2 Montréal   CAN              1970-06-17 QC                               24
    ## 3 Gresham    USA              1979-06-21 OR                               NA
    ## 4 Surahammar SWE              1971-02-01 <NA>                            118
    ## 5 Saskatoon  CAN              1974-05-11 SK                               98
    ## 6 Windsor    CAN              1967-01-29 ON                               24
    ## # ℹ 29 more variables: draftRound <int>, draftYear <int>, goalieFullName <chr>,
    ## #   height <int>, isInHallOfFameYn <chr>, lastName <chr>,
    ## #   nationalityCode <chr>, playerId <int>, shootsCatches <chr>, weight <int>,
    ## #   seasonId <chr>, assists <int>, gamesPlayed <int>, gamesStarted <int>,
    ## #   goals <int>, goalsAgainst <int>, goalsAgainstAverage <dbl>, losses <int>,
    ## #   otLosses <int>, penaltyMinutes <int>, points <int>, savePct <dbl>,
    ## #   saves <int>, shotsAgainst <int>, shutouts <int>, teamAbbrevs <chr>, …

``` r
tail(goalies)
```

    ## # A tibble: 6 × 34
    ##   birthCity     birthCountryCode birthDate  birthStateProvinceCode draftOverall
    ##   <chr>         <chr>            <chr>      <chr>                         <int>
    ## 1 Lugnvik       SWE              1993-07-31 <NA>                            163
    ## 2 Richmond Hill CAN              1993-07-11 ON                               88
    ## 3 Bern          CHE              2000-05-12 <NA>                            136
    ## 4 St-Hyacinthe  CAN              1992-03-06 QC                              138
    ## 5 Milford       USA              1986-01-21 CT                               72
    ## 6 Thunder Bay   CAN              1994-05-25 ON                               83
    ## # ℹ 29 more variables: draftRound <int>, draftYear <int>, goalieFullName <chr>,
    ## #   height <int>, isInHallOfFameYn <chr>, lastName <chr>,
    ## #   nationalityCode <chr>, playerId <int>, shootsCatches <chr>, weight <int>,
    ## #   seasonId <chr>, assists <int>, gamesPlayed <int>, gamesStarted <int>,
    ## #   goals <int>, goalsAgainst <int>, goalsAgainstAverage <dbl>, losses <int>,
    ## #   otLosses <int>, penaltyMinutes <int>, points <int>, savePct <dbl>,
    ## #   saves <int>, shotsAgainst <int>, shutouts <int>, teamAbbrevs <chr>, …

``` r
# Tidy up some columns and sort
goalies = goalies |>
  rename(
    catches = shootsCatches,
    fullName = goalieFullName
  ) |>
  arrange(seasonId, fullName)

dim(goalies)
```

    ## [1] 2257   34

``` r
head(goalies)
```

    ## # A tibble: 6 × 34
    ##   birthCity  birthCountryCode birthDate  birthStateProvinceCode draftOverall
    ##   <chr>      <chr>            <chr>      <chr>                         <int>
    ## 1 Belleville CAN              1980-05-04 ON                              135
    ## 2 Riga       LVA              1967-02-02 <NA>                            196
    ## 3 Toronto    CAN              1965-01-14 ON                               69
    ## 4 Farmington USA              1977-03-12 MI                              129
    ## 5 Woonsocket USA              1977-01-02 RI                               22
    ## 6 Sussex     GBR              1971-02-25 <NA>                             35
    ## # ℹ 29 more variables: draftRound <int>, draftYear <int>, fullName <chr>,
    ## #   height <int>, isInHallOfFameYn <chr>, lastName <chr>,
    ## #   nationalityCode <chr>, playerId <int>, catches <chr>, weight <int>,
    ## #   seasonId <chr>, assists <int>, gamesPlayed <int>, gamesStarted <int>,
    ## #   goals <int>, goalsAgainst <int>, goalsAgainstAverage <dbl>, losses <int>,
    ## #   otLosses <int>, penaltyMinutes <int>, points <int>, savePct <dbl>,
    ## #   saves <int>, shotsAgainst <int>, shutouts <int>, teamAbbrevs <chr>, …

``` r
tail(goalies)
```

    ## # A tibble: 6 × 34
    ##   birthCity       birthCountryCode birthDate birthStateProvinceCode draftOverall
    ##   <chr>           <chr>            <chr>     <chr>                         <int>
    ## 1 Surrey          CAN              1995-04-… BC                               44
    ## 2 Espoo           FIN              1999-03-… <NA>                             54
    ## 3 Helsinki        FIN              1995-02-… <NA>                             94
    ## 4 Havlickuv Brod  CZE              1996-01-… <NA>                             39
    ## 5 Dollard-des-Or… CAN              2000-03-… QC                               NA
    ## 6 Omsk            RUS              2002-06-… <NA>                             11
    ## # ℹ 29 more variables: draftRound <int>, draftYear <int>, fullName <chr>,
    ## #   height <int>, isInHallOfFameYn <chr>, lastName <chr>,
    ## #   nationalityCode <chr>, playerId <int>, catches <chr>, weight <int>,
    ## #   seasonId <chr>, assists <int>, gamesPlayed <int>, gamesStarted <int>,
    ## #   goals <int>, goalsAgainst <int>, goalsAgainstAverage <dbl>, losses <int>,
    ## #   otLosses <int>, penaltyMinutes <int>, points <int>, savePct <dbl>,
    ## #   saves <int>, shotsAgainst <int>, shutouts <int>, teamAbbrevs <chr>, …

``` r
# Export NHL goalies data to a csv
write_csv(goalies, "../data/nhl_goalies_data.csv")
```

## Get NHL Draft Ranking Data

``` r
DRAFT_YEARS = 2008:2025  # data only goes back until 2008

# Create URLs to fetch goalie prospect rankings by draft year.
# Notice there are separate urls for North American and International goalies
na_urls = paste0(NHLWEB_API_BASE_URL, "v1/draft/rankings/", DRAFT_YEARS, "/3")
itl_urls = paste0(NHLWEB_API_BASE_URL, "v1/draft/rankings/", DRAFT_YEARS, "/4")

# Create http request objects
na_reqs = lapply(na_urls, httr2::request)
itl_reqs = lapply(itl_urls, httr2::request)

# Perform http requests
na_resps = req_perform_sequential(na_reqs)
```

    ## Iterating ■■■■■■■■ 22% | ETA: 4sIterating ■■■■■■■■■ 28% | ETA: 4sIterating
    ## ■■■■■■■■■■■ 33% | ETA: 3sIterating ■■■■■■■■■■■■■ 39% | ETA: 3sIterating
    ## ■■■■■■■■■■■■■■ 44% | ETA: 3sIterating ■■■■■■■■■■■■■■■■ 50% | ETA: 2sIterating
    ## ■■■■■■■■■■■■■■■■■■ 56% | ETA: 2sIterating ■■■■■■■■■■■■■■■■■■■ 61% | ETA:
    ## 2sIterating ■■■■■■■■■■■■■■■■■■■■■ 67% | ETA: 2sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■ 72% | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■ 78% |
    ## ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■■■ 83% | ETA: 1sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 89% | ETA: 1sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 94% | ETA: 0s

``` r
itl_resps = req_perform_sequential(itl_reqs)
```

    ## Iterating ■■■■■■■■■ 28% | ETA: 3sIterating ■■■■■■■■■■■ 33% | ETA: 3sIterating
    ## ■■■■■■■■■■■■■ 39% | ETA: 3sIterating ■■■■■■■■■■■■■■ 44% | ETA: 2sIterating
    ## ■■■■■■■■■■■■■■■■ 50% | ETA: 2sIterating ■■■■■■■■■■■■■■■■■■ 56% | ETA:
    ## 2sIterating ■■■■■■■■■■■■■■■■■■■ 61% | ETA: 2sIterating ■■■■■■■■■■■■■■■■■■■■■
    ## 67% | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■ 72% | ETA: 1sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■ 78% | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■■■ 83%
    ## | ETA: 1sIterating ■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 89% | ETA: 0sIterating
    ## ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■ 94% | ETA: 0s

``` r
# Extract north american and international goalie prospects from requests
na_data = map(na_resps, function(resp) {
  resp_data = resp_body_json(resp)
  goalies = resp_data$rankings
  lapply(goalies, function(goalie) {
    goalie$draftYear = resp_data$draftYear
    goalie$midtermRank = ifelse(goalie$midtermRank, goalie$midtermRank, NA_integer_)
    goalie$finalRank = ifelse(goalie$finalRank, goalie$finalRank, NA_integer_)
    goalie$international = FALSE
    map(goalie, ~ifelse(is.null(.), NA, .))
  })
})
itl_data = map(itl_resps, function(resp) {
  resp_data = resp_body_json(resp)
  goalies = resp_data$rankings
  lapply(goalies, function(goalie) {
    goalie$draftYear = resp_data$draftYear
    goalie$midtermRank = ifelse(goalie$midtermRank, goalie$midtermRank, NA_integer_)
    goalie$finalRank = ifelse(goalie$finalRank, goalie$finalRank, NA_integer_)
    goalie$international = TRUE
    map(goalie, ~ifelse(is.null(.), NA, .))
  })
})

# Convert extracted data to tibbles
na_prospects = list_rbind(map(list_c(na_data), \(x) as_tibble(x) ))
itl_prospects = list_rbind(map(list_c(itl_data), \(x) as_tibble(x) ))

dim(na_prospects)
```

    ## [1] 640  16

``` r
head(na_prospects)
```

    ## # A tibble: 6 × 16
    ##   lastName   firstName positionCode shootsCatches heightInInches weightInPounds
    ##   <chr>      <chr>     <chr>        <chr>                  <int>          <int>
    ## 1 McCollum   Thomas    G            L                         74            220
    ## 2 Pickard    Chet      G            L                         74            215
    ## 3 Delmas     Peter     G            L                         74            185
    ## 4 Holtby     Braden    G            L                         73            200
    ## 5 Hutchinson Michael   G            R                         74            183
    ## 6 Deserres   Jacob     G            L                         74            186
    ## # ℹ 10 more variables: lastAmateurClub <chr>, lastAmateurLeague <chr>,
    ## #   birthDate <chr>, birthCity <chr>, birthStateProvince <chr>,
    ## #   birthCountry <chr>, midtermRank <int>, finalRank <int>, draftYear <int>,
    ## #   international <lgl>

``` r
tail(na_prospects)
```

    ## # A tibble: 6 × 16
    ##   lastName    firstName positionCode shootsCatches heightInInches weightInPounds
    ##   <chr>       <chr>     <chr>        <chr>                  <int>          <int>
    ## 1 Newlove     Michael   G            L                         74            172
    ## 2 Quinlan     Patrick   G            L                         73            185
    ## 3 Hendrickson Kambryn   G            R                         72            176
    ## 4 Lee-Stack   Dylan     G            L                         74            172
    ## 5 Egorov      David     G            L                         74            180
    ## 6 Langevin    Mathis    G            L                         76            182
    ## # ℹ 10 more variables: lastAmateurClub <chr>, lastAmateurLeague <chr>,
    ## #   birthDate <chr>, birthCity <chr>, birthStateProvince <chr>,
    ## #   birthCountry <chr>, midtermRank <int>, finalRank <int>, draftYear <int>,
    ## #   international <lgl>

``` r
dim(itl_prospects)
```

    ## [1] 229  16

``` r
head(itl_prospects)
```

    ## # A tibble: 6 × 16
    ##   lastName  firstName positionCode shootsCatches heightInInches weightInPounds
    ##   <chr>     <chr>     <chr>        <chr>                  <int>          <int>
    ## 1 Markstrom Jacob     G            L                         78            196
    ## 2 Sateri    Harri     G            L                         73            202
    ## 3 Lindback  Anders    G            L                         78            220
    ## 4 Bobrovsky Sergei    G            L                         74            190
    ## 5 Lehner    Robin     G            L                         75            220
    ## 6 Koskinen  Mikko     G            L                         79            202
    ## # ℹ 10 more variables: lastAmateurClub <chr>, lastAmateurLeague <chr>,
    ## #   birthDate <chr>, birthCity <chr>, birthCountry <chr>, midtermRank <int>,
    ## #   finalRank <int>, draftYear <int>, international <lgl>,
    ## #   birthStateProvince <chr>

``` r
tail(itl_prospects)
```

    ## # A tibble: 6 × 16
    ##   lastName    firstName positionCode shootsCatches heightInInches weightInPounds
    ##   <chr>       <chr>     <chr>        <chr>                  <int>          <int>
    ## 1 Sammalniemi Jooa      G            L                         72            183
    ## 2 Sorqvist    Isak      G            L                         72            172
    ## 3 Tkach-Tkac… Ivan      G            L                         75            185
    ## 4 Orsulak     Michal    G            R                         76            224
    ## 5 Birchler    Matia     G            L                         76            175
    ## 6 Carlsson    Simon     G            L                         74            183
    ## # ℹ 10 more variables: lastAmateurClub <chr>, lastAmateurLeague <chr>,
    ## #   birthDate <chr>, birthCity <chr>, birthCountry <chr>, midtermRank <int>,
    ## #   finalRank <int>, draftYear <int>, international <lgl>,
    ## #   birthStateProvince <chr>

``` r
# Join north american and international goalie prospects into one tibble
prospects = rbind(na_prospects, itl_prospects)

dim(prospects)
```

    ## [1] 869  16

``` r
# Tidy up some columns and sort
prospects = prospects |>
  mutate(
    fullName = paste(firstName, lastName)
  ) |>
  select(!c(firstName, lastName)) |>
  rename(
    catches = shootsCatches,
    height = heightInInches,
    weight = weightInPounds, 
    birthCountryCode = birthCountry
  ) |>
  arrange(draftYear, finalRank, midtermRank, fullName)

dim(prospects)
```

    ## [1] 869  15

``` r
head(prospects)
```

    ## # A tibble: 6 × 15
    ##   positionCode catches height weight lastAmateurClub lastAmateurLeague birthDate
    ##   <chr>        <chr>    <int>  <int> <chr>           <chr>             <chr>    
    ## 1 G            L           78    196 Brynas Jr.      SWEDEN-JR.        1990-01-…
    ## 2 G            L           74    220 Guelph          OHL               1989-12-…
    ## 3 G            L           74    215 Tri-City        WHL               1989-11-…
    ## 4 G            L           73    202 Tappara         FINLAND           1989-12-…
    ## 5 G            L           74    185 Lewiston        QMJHL             1990-02-…
    ## 6 G            L           78    220 Brynas          SWEDEN            1988-05-…
    ## # ℹ 8 more variables: birthCity <chr>, birthStateProvince <chr>,
    ## #   birthCountryCode <chr>, midtermRank <int>, finalRank <int>,
    ## #   draftYear <int>, international <lgl>, fullName <chr>

``` r
tail(prospects)
```

    ## # A tibble: 6 × 15
    ##   positionCode catches height weight lastAmateurClub lastAmateurLeague birthDate
    ##   <chr>        <chr>    <int>  <int> <chr>           <chr>             <chr>    
    ## 1 G            L           74    172 BURLINGTON      OJHL              2007-01-…
    ## 2 G            L           73    185 USA U-18        NTDP - USHL       2007-04-…
    ## 3 G            R           72    176 WATERLOO        USHL              2006-01-…
    ## 4 G            L           74    172 BRUNSWICK PREP  HIGH-CT           2007-04-…
    ## 5 G            L           74    180 BRANTFORD       OHL               2006-05-…
    ## 6 G            L           76    182 RIMOUSKI        QMJHL             2006-06-…
    ## # ℹ 8 more variables: birthCity <chr>, birthStateProvince <chr>,
    ## #   birthCountryCode <chr>, midtermRank <int>, finalRank <int>,
    ## #   draftYear <int>, international <lgl>, fullName <chr>

``` r
# Export NHL prospects data to a csv
write_csv(prospects, "../data/nhl_goalie_prospects_data.csv")
```
