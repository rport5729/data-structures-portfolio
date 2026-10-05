# Do Psychological Factors and Game State Predict Match Outcomes in Deadlock?

## Core Design

- **Unit of analysis:** one player in one match.
- **Target:** win or loss, so this is binary classification.
- **Feature sets**, each added on top of the last:
  1. **Controls:** rank or MMR, hero, patch.
  2. **Psychological (pre-match):** loss streak, hero experience, party status, fatigue.
  3. **Game state at a time point:** soul and objective differences at 10, 20, and 30 minutes.
- **Models:** logistic regression, random forest, and gradient boosting (XGBoost or LightGBM).

---

## 1. Problem Definition

- **Main question:** Do psychological factors predict match outcome beyond skill? How does their predictive value change as the match goes on?
- **Hypotheses:**
  - Players on loss streaks win their next game less often.
  - More experience on a hero increases win probability.
  - Players in parties win more often than solo players.
  - Psychological features matter most before the match, and their predictive value shrinks as in-game stats accumulate.
- **Who benefits:**
  - Game designers: matchmaking and "take a break" prompts.
  - Professional coaches and players: managing tilt.
  - Researchers studying resilience, practice, and teamwork.

According to a study, flow experience, attitude, perceived enjoyment, perceived behavioral control, and subjective norm are direct effects that facilitate the customers' continued intention to pla[...]

- **Why it matters:** The relationship between online game experience and players' cognitive functions has become increasingly important in the literature. Unfortunately, the association between o[...]
- **Why a MOBA:** In contrast to such popularity and broad research interest in MOBA, the field of gaming expertise research is currently dominated by game genres such as puzzle games, action game[...]

---

## 2. Background and Context

- **Game Information:** Deadlock is Valve's 6v6 hybrid of a MOBA and third-person shooter. Players are placed on a large map with three primary lanes, two players each. Players aim to take objecti[...]

Psychology sources are listed in [References](#references).

---

## 3. Data Description

- **Source:** [deadlock-api.com](https://deadlock-api.com), a community API built on Valve's game client data.
- **Pulling data:**
  - Collect match histories for a sample of players across ranks, which gives you streak and experience data.
  - Pull match metadata to get team stats at specific times. Check early whether the metadata includes stats at timestamps. If it only has end-of-match totals, drop the 20 and 30-minute snapshots [...]
- **Report:**
  - Rows, number of players and matches, date range, patch, and game mode (ranked only is cleanest).
  - Missing values per column.
- **Limitations:**
  - Third-party data that may lean toward dedicated players.
  - Frequent balance patches.
  - Only public profiles are included.

---

## 4. Data Understanding and Exploration

- **Summary statistics:**
  - The summary statistics show that streaks do not happen often, as we go from almost 72,000 matches with no streak to around 2,500 matches on just a two-streak (within the observed pages and dat[...]
  - Missing values are found on the second chart in terms of later games having low or no sample size. The longer the game goes on, the more comeback mechanics are in play. This forces games to gr[...]
- **Target distribution:** When considering non-streaked matches, it sits at a near 50%, which matches the target. Even with 1 and 2 wins or losses in a row, it still sits near the target. This he[...]
- **Patterns and outliers:** The heatmap suggests that greater relative gold advantage is associated with higher win probability, especially at 15 minutes. Later checkpoints have fewer matches and[...]
- **Visualizations:** I use a win/loss count plot for target balance, the streak bar chart for the psychological proxy, and the heatmap for relative gold advantage by checkpoint. Boxplots can help[...]
- **Feature choices:** Compute streaks only from matches before the current one.

![Streak Win Rate](streakwinrate.png)

---

## 5. Data Preparation and Feature Selection

- **Feature engineering:** sort each player's matches by time, then compute these using only earlier matches:
  - Loss streak.
  - Previous result.
- **Game-state features:** build each snapshot only from stats up to that minute, as differences from the player's team to the enemy team (souls, kills, objectives). Compare it to whether the team[...]
- **Cleaning:**
  - Remove duplicates, abandoned or very short matches, and players with too few matches.
  - Exclude volatile games that do not show a good representation of team leads.
- **Transformations:**
  - Log-transform or cap skewed counts.
- **Excluded features:** end-of-match totals (final kills, final souls, damage) because they leak the result, plus player IDs.

---

## 6. Baseline and Model Development

- **Baseline:** A majority-class classifier, which always predicts the most common outcome. It provides a simple benchmark for whether the trained models add predictive value.
- **Models:** Logistic regression and random forest, predicting whether a team wins from checkpoint minute and its relative soul lead.
- **Why these models:** Logistic regression tests whether these features have a useful, relatively simple relationship with wins. Random forest can capture more complex, nonlinear patterns.
- **Tuning:** Logistic regression tested `C` values of 0.1, 1, and 10. Random forest tested maximum depths of unlimited or 10 and minimum leaf sizes of 1 or 5, using 100 trees. Settings were compa[...]
- **Fair comparison:** The data was split by match, not by individual team observations. Both teams and all checkpoints from a match stayed together, preventing match information from appearing in[...]

---

## 7. Model Evaluation and Selection

- **Metrics:** Accuracy measures correct predictions; precision, recall, and F1 summarize positive-win predictions; ROC-AUC measures how well the model ranks winners above non-winners across thres[...]
- **Test results:**

| Model | Accuracy | F1 | ROC-AUC |
|---|---|---|---|
| Baseline (majority class) | 0.500 | — | 0.500 |
| Logistic Regression | **0.694** | **0.694** | **0.762** |
| Random Forest | 0.681 | 0.682 | 0.757 |

- **Final model:** Logistic regression.
- **Evidence:** It had the highest grouped cross-validation ROC-AUC (0.775 versus 0.769 for random forest) and also performed best on the held-out test set.
- **Tradeoffs:** Random forest can capture more complex patterns, but it did not outperform logistic regression here. Logistic regression was slightly better on the reported test metrics and is s[...]

---

## 8. Model Interpretation and Insights

- **What the model learned:** A team's relative soul lead is strongly related to whether it wins. A larger lead corresponds to higher odds of winning. Match minute had very little influence in th[...]
- **Most influential features:** Soul lead was the strongest feature. The logistic regression coefficient was 1.852 per standard deviation of soul lead, which corresponds to about 6.375 times the[...]

![Soul Lead Win Rate](soulleadwinrate.png)

- **Where it performs well or poorly:** It performed best at 15 and 25 minutes, with 70.8% accuracy and ROC-AUC scores of 0.793 and 0.764. Performance was weaker at 35 minutes (62.1% accuracy) an[...]
- **What the confusion matrix shows:** On the held-out data, logistic regression correctly predicted 2,119 losses and 2,119 wins. It incorrectly predicted 934 wins as losses and 934 losses as win[...]

|  | Predicted Loss | Predicted Win |
|---|---|---|
| **Actual Loss** | 2,119 | 934 |
| **Actual Win** | 934 | 2,119 |

- **What the coefficients and feature importance show:** Both methods point to soul lead as the main predictor. The logistic regression's minute coefficient rounds to 0.000, and the random forest[...]
- **What the probability errors and examples show:** The average absolute difference between predicted win probability and actual outcome was 0.364 at 15 minutes, increasing to 0.469 at 45 minute[...]
- **What we can conclude:** Soul lead is a useful predictor of a team's eventual win in this sample, especially earlier in the match.
- **What we cannot conclude:** These results do not show that gaining souls causes a win, or that the same performance will hold for other patches, dates, or samples. The model uses only checkpoi[...]

---

## 9. Limitations, Ethics, and Reflection

- **Bias:**
  - Sampling bias toward tracked players.
  - Disregarding the patch: different patches have altered the amplitude of comeback mechanics.
  - Possible matchmaking effects, since the system may give losing players easier opponents and hide or reverse tilt effects. However, this is often a myth and can only be truly confirmed if the [...]
  - Treating team outcomes as individual rows.
- **Prediction errors:** In matchmaking or professional play, a false "likely to lose" label could demotivate players and unfairly alter their mindset.
- **Real-world use:** Appropriate for research about mental wellness in competitive fields or games.
- **Privacy:** No specific player data is given or shown. All player data is publicly available, but for the purpose of this research study, no player IDs or references will be shown.
- **Transfer to real life:** Gaming behavior may not fully carry over to work or school settings.
- **Next steps:**
  - Track psychological effects across patches.
  - Add chat or behavior data if it becomes available.
  - Try adding in rank information to determine if rank can affect any factors.

---

## 10. Code and Transparency

- **Repository structure:**
  - `data-structures-portfolio/research/2026/deadlock`
- **Data citations:** [deadlock-api.com](https://deadlock-api.com) and its API documentation.
- **AI disclosure:** AI tools were used adhering to course policy. Claude was used in producing and refining an outline as well as helping analyze the data, and GitHub Copilot was used in support[...]

---

## References

Chang, Y., Liu, D., Chen, Y., & Hsieh, S. (2017). The relationship between online game experience and multitasking ability in a virtual environment. *Applied Cognitive Psychology, 31*(6), 653–6[...]

Ding, Y., Hu, X., Li, J., Ye, J., Wang, F., & Zhang, D. (2018). What makes a champion: The behavioral and neural correlates of expertise in multiplayer online battle arena games. *International J[...]

Lee, M.-C., & Tsai, T.-R. (2010). What drives people to continue to play online games? An extension of technology model and theory of planned behavior. *International Journal of Human-Computer In[...]
