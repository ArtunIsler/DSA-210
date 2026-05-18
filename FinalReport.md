# The Impact of Pricing and Ratings on Video Game Popularity

## DSA210 Term Project - Final Report

**Student:** Artun İşler  
**Spring 2025–2026**

---

## 1. Motivation

Video games are now one of the biggest entertainment industries in the world. As someone who loves gaming and wants to work in this industry, I've always been curious about what actually makes a game popular. Is it the price? The ratings? Or something else entirely?

This project tries to answer that question using real data. I looked at how pricing and user ratings relate to a game's popularity on Steam, using review counts as a measure of player engagement. The goal is to find out what actually drives a game's success — not just what we assume.

---

## 2. Data Sources

Three complementary datasets were used to capture pricing, ratings, and popularity:

### Steam Games Dataset (Primary)

- **Source:** https://www.kaggle.com/datasets/fronkongames/steam-games-dataset
- **Size:** 122,611 games, 39 columns
- **Key variables:** Game title, price, positive reviews, negative reviews, genres, release date
- **Role:** Main source for popularity (review counts) and pricing data

### SteamSpy API Dataset (Secondary)

- **Source:** https://www.kaggle.com/datasets/muhammadaqeelkabir/steam-games-dataset-steamspy-api
- **Size:** 10,000 games, 17 columns
- **Key variables:** Ownership estimates, average playtime, user score
- **Role:** Enriches the Steam dataset with playtime and ownership data

### Video Game Sales Dataset (Secondary)

- **Source:** https://www.kaggle.com/datasets/gregorut/videogamesales
- **Size:** 16,598 games, 11 columns
- **Key variables:** Platform, genre, global sales figures
- **Role:** Provides a broader market perspective and additional genre information

The datasets were merged on game title (lowercased and stripped), resulting in a final dataset of **83,035 games** with **15 features**.

---

## 3. Data Collection & Preprocessing

Data collection and cleaning involved the following steps:

1. **Loading:** All three datasets were downloaded from Kaggle and uploaded to Google Colab.

2. **Column correction:** The Steam dataset had a column alignment issue — the `AppID` column contained game titles. This was corrected by renaming columns appropriately.

3. **Filtering:** Games with zero reviews were removed (no engagement signal). Prices above $200 were excluded as outliers (likely DLC bundles or errors).

4. **Feature engineering:**
   - `total_reviews` (positive + negative) were computed as the popularity proxy
   - `popularity_rank` was assigned using `rank(ascending=False, method="first")`
   - `is_free` binary flag was added (1 if price = 0)
   - `rating` column was derived from SteamSpy user scores

5. **Merging:** Steam data was merged with SteamSpy (left join) and VGSales (left join) on cleaned game titles.

**Final dataset:** 83,035 rows × 15 columns

---

## Summary Statistics

![Summary Statistics](Visualization/image1.png)

---

## Missing Values

![Missing Values](Visualization/image2.png)

---

## 4. Exploratory Data Analysis

### Dataset Composition

- **Free games:** 49,525 (59.6%)
- **Paid games:** 33,510 (40.4%)

### Price Distribution

Most games are free-to-play. Among paid games, prices are heavily right-skewed, with most priced under $20.

### Popularity Distribution

Review counts are extremely right-skewed. The top game (**Counter-Strike 2**) has **8,815,087 reviews**, while the median game has only **24 reviews**. Log transformation reveals a roughly normal distribution.

### Top 10 Most Popular Games

![Top 10 Most Popular Games](Visualization/image3.png)

### Genre Analysis (Median Reviews)

Free To Play games lead with a median of **69 reviews**, followed by:

- RPG (**36**)
- Simulation (**33**)
- Adventure (**27**)
- Strategy (**27**)

---

## 5. Hypothesis Testing

All tests used a significance level of **α = 0.05**. Non-parametric tests were chosen due to the heavily skewed distribution of review counts.

### H1: Free vs. Paid Games — Popularity

- **Test:** Mann-Whitney U (two-sided)
- **Free games:** n = 49,525, Median = 16, Mean = 1,101
- **Paid games:** n = 33,510, Median = 51, Mean = 2,838
- **U statistic:** 574,196,402 | **p-value:** 0.000000
- **Result:** REJECT H0 — Paid games have significantly higher median reviews than free games.

> Interestingly, while free games are more numerous and the top games by raw count are free, paid games have a higher median review count overall — suggesting that paid games that survive in the market tend to attract more reviews per title.

---

### H2: Price Category vs. Popularity

- **Test:** Kruskal-Wallis
- **Median reviews by category:**  
  Free = 16,  
  Low ($0.01–$9.99) = 37,  
  Mid ($10–$29.99) = 41,  
  High ($30–$59.99) = 72,  
  Premium ($60+) = 84
- **Statistic:** 6219.68 | **p-value:** 0.000000
- **Result:** REJECT H0 — Popularity differs significantly across price categories.

---

### H3: Playtime vs. Popularity

- **Test:** Spearman correlation (one-tailed)
- **Valid samples:** 7,896 games
- **Median playtime:** 360 minutes
- **Median reviews:** 2,258
- **ρ = 0.4919** | **p-value:** 0.000000
- **Result:** REJECT H0 — Significant positive correlation between playtime and popularity.

---

### H4: Game Genre vs. Popularity

- **Test:** Kruskal-Wallis
- **Statistic:** 3542.97 | **p-value:** 0.000000
- **Result:** REJECT H0 — Genre significantly influences popularity.

---

### H5: User Score vs. Popularity

- **Test:** Spearman correlation (one-tailed)
- **Valid samples with score > 0:** only 3 games
- **Result:** INCONCLUSIVE — Sample size critically too small (n = 3) for reliable inference.

## 6. Machine Learning

Three models were trained to predict game popularity using:

`log(total_reviews + 1)`

The models used the following features:

- Price
- Is free
- Genre encoded
- `avg_playtime_minutes`
- Rating

### Regression Model Results

![Machine Learning Results 1](media/image4.png)

![Machine Learning Results 2](media/image5.png)

Random Forest significantly outperformed Linear Regression, achieving **R² = 0.4449** compared to **0.1276**.

Average playtime dominated feature importance with **86.2%**, suggesting that how long players engage with a game is the strongest predictor of its popularity.

---

### K-Means Clustering

![K-Means Clustering](media/image6.png)

- **Cluster 0:** Nearly all free games with lower popularity
- **Clusters 1–3:** Paid games at increasing price points with increasing popularity
- **Cluster 3:** Premium-priced games (average price ≈ $79) with the highest average popularity

## 7. Key Findings

1. Paid games have higher median review counts than free games, even though free games dominate the top of the raw popularity leaderboard. This suggests that paid games may attract more consistent engagement per title.

2. Higher price categories are associated with higher median popularity. Premium games ($60+) have a median of 84 reviews, compared to 16 for free games. This challenges the assumption that free games are always more popular.

3. Playtime is the strongest predictor of popularity in this analysis (**Spearman ρ = 0.49**, **Random Forest feature importance = 86%**). Games that keep players engaged for longer tend to accumulate more reviews.

4. Genre has a significant relationship with popularity. Free To Play and RPG games show higher median review counts compared to many other genres.

5. The machine learning results show that the selected features explain about **44% of the variance** in log-transformed popularity (**Random Forest R² = 0.4449**). Among these features, average playtime is by far the most important predictor. The remaining unexplained variance is likely related to factors not included in the dataset, such as marketing, franchise recognition, community effects, and streamer coverage.

6. User score data was insufficient for reliable analysis. Only 3 games had valid SteamSpy user scores above 0, making H5 inconclusive.

---

## 8. Limitations and Future Work

### Limitations

- **Steam Spy coverage is limited:** Only ~9,000 of 83,000 games had playtime and ownership data. Only 3 games had usable user scores, making rating-based analysis hard to observe.

- **Steam-centric dataset:** The analysis covers only PC/Steam games and does not capture consoles or mobile markets.

- **No temporal analysis:** Release timing, seasonal effects, and major updates were not considered.

- **Unmeasured factors:** Marketing budgets, influencer/streamer coverage, and franchise recognition likely affect popularity that cannot be captured in these datasets.

- **Column alignment issue in Steam dataset:** Required manual correction, which may introduce minor errors.

### Future Work

- Incorporate Twitch/YouTube viewership data as an additional popularity signal.

- Analyze the effect of Steam discount events on review counts over time.

- Expand the analysis to console and mobile platforms using APIs such as IGDB or RAWG.

- Apply NLP sentiment analysis to Steam review text to create a richer rating variable.

- Build a recommendation model using game features and player engagement patterns.

---

## 9. AI Usage

I used AI for:

- **Claude:** Coding support, providing information about hypothesis testing, data cleaning, machine learning, and generating code outputs.

- **ChatGPT:** Used for organizing and enhancing report documentation, README, and the final report.

---

## 10. References

- Steam Games Dataset:  
  https://www.kaggle.com/datasets/fronkongames/steam-games-dataset

- Video Game Sales with Ratings:  
  https://www.kaggle.com/datasets/gregorut/videogamesales

- Steam Spy API Dataset:  
  https://www.kaggle.com/datasets/muhammadaqeelkabir/steam-games-dataset-steamspy-apiales
Steam Spy API Dataset: https://www.kaggle.com/datasets/muhammadaqeelkabir/steam-games-dataset-steamspy-api
