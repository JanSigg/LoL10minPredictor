# LoL 10-Minute Predictor

Predikerer om det **blå laget vinner** en League of Legends-kamp, kun basert på statistikk fra
de **første 10 minuttene**. Bygget med logistisk regresjon på 9 879 rangerte Diamond-kamper.

> **Resultat:** 71,9 % accuracy på testsettet, mot en baseline på 50,1 %.

---

## 🇳🇴 Norsk

### Om prosjektet

Dette er besvarelsen på **Mandatory Assignment 2** (use case: Alternativ 4 – eget datasett).
Oppgaven er et binært klassifiseringsproblem: ut fra tallene etter 10 minutter skal modellen
svare på om blått lag ender opp med å vinne (`blueWins = 1`) eller tape (`blueWins = 0`).

Hele analysen ligger i notebooken [`MA2_LoL_Final.ipynb`](MA2_LoL_Final.ipynb) og går gjennom
hele løpet fra rådata til ferdig modell og demo-prediksjon.

### Resultater

| Metrikk | Verdi |
| --- | --- |
| Accuracy (test) | **71,9 %** |
| Baseline (alltid vanligste utfall) | 50,1 % |
| Precision / Recall / F1 | 0,72 / 0,72 / 0,72 |
| Riktige prediksjoner | 1 421 av 1 976 kamper |

Confusion matrix på testsettet:

|  | Predikert tap | Predikert seier |
| --- | --- | --- |
| **Faktisk tap** | 708 | 282 |
| **Faktisk seier** | 273 | 713 |

Feilene er jevnt fordelt mellom de to klassene, så modellen favoriserer ikke ett utfall.

### Datasett

* **Kilde:** [League of Legends Diamond Ranked Games (10 min)](https://www.kaggle.com/datasets/bobbyscience/league-of-legends-diamond-ranked-games-10-min) på Kaggle (bobbyscience)
* **Størrelse:** 9 879 kamper × 40 kolonner
* **Datakvalitet:** 0 manglende verdier, 0 dupliserte rader
* **Klassebalanse:** 4 949 tap / 4 930 seire – praktisk talt 50/50, så accuracy er en meningsfull metrikk

Filen `high_diamond_ranked_10min.csv` ligger i repoet, så notebooken kjører uten nedlasting.

### Framgangsmåte

1. **Utforsking** – form, datatyper, manglende verdier, duplikater og klassebalanse.
2. **Visualisering** – fordelingen av `blueGoldDiff` for vinnende og tapende lag. Gullforskjellen
   skiller tydelig (ca. ±1 500), men fordelingene overlapper rundt null – jevne kamper er vanskelige.
3. **Feature-valg** – fra 40 til 16 kolonner:
   * `gameId` droppet (ren ID, ingen prediktiv verdi).
   * Alle `red*`-kolonner droppet, fordi de er speilvendte av de blå (`redKills` = `blueDeaths`,
     `redGoldDiff` = −`blueGoldDiff`). Diff-kolonnene til blått inneholder allerede info om rødt lag.
   * Avledede kolonner droppet: `blueGoldPerMin`, `blueCSPerMin`, `blueEliteMonsters`
     (kun omregninger av kolonner vi allerede har).
4. **Modellvalg** – logistisk regresjon: målet er binært, alle features er numeriske, modellen gir
   en *sannsynlighet* i stedet for bare ja/nei, og koeffisientene er enkle å tolke.
5. **Splitt** – 80 % trening (7 903) / 20 % test (1 976), `stratify=y` og `random_state=42`.
6. **Trening** – `StandardScaler` + `LogisticRegression(max_iter=1000)` i en pipeline. Skalering er
   nødvendig fordi gull måles i tusener mens drager er 0 eller 1.
7. **Evaluering** – accuracy mot baseline, confusion matrix og classification report.
8. **Tolkning** – koeffisientplott som viser hvilke features som trekker mot seier og tap.
9. **Demo** – en medianKamp der `blueGoldDiff`, `blueExperienceDiff` og `blueDragons` justeres opp,
   gir 86 % sannsynlighet for blå seier.

### Features modellen bruker (16)

```
blueWardsPlaced          blueDragons              blueTotalExperience
blueWardsDestroyed       blueHeralds              blueTotalMinionsKilled
blueFirstBlood           blueTowersDestroyed      blueTotalJungleMinionsKilled
blueKills                blueTotalGold            blueGoldDiff
blueDeaths               blueAvgLevel             blueExperienceDiff
blueAssists
```

### Viktigste funn

* **`blueGoldDiff` og `blueExperienceDiff` betyr klart mest.** Et forsprang i gull og erfaring etter
  10 minutter er den sterkeste indikatoren på seier.
* Features som henger tett sammen (f.eks. `blueTotalGold` og `blueGoldDiff`) kan få uventede fortegn,
  fordi modellen fordeler effekten mellom dem – det er multikollinearitet, ikke en feil i dataene.
* 10 minutter er tidlig i en LoL-kamp. Comebacks og jevne kamper er i praksis ikke mulige å treffe,
  og det forklarer mye av de resterende 28 %.

### Kom i gang

```bash
git clone https://github.com/JanSigg/LoL10minPredictor.git
cd LoL10minPredictor

pip install pandas matplotlib scikit-learn jupyter
jupyter notebook MA2_LoL_Final.ipynb
```

Kjør cellene fra toppen (`Kernel → Restart & Run All`). Notebooken er kjørt med **Python 3.12.7**.

### Filer i repoet

| Fil | Beskrivelse |
| --- | --- |
| `MA2_LoL_Final.ipynb` | Hele analysen: utforsking, feature-valg, trening, evaluering og demo |
| `high_diamond_ranked_10min.csv` | Datasettet (9 879 kamper) |
| `README.md` | Denne filen |

### Videre arbeid

* Prøve Random Forest eller Gradient Boosting og sammenligne mot logistisk regresjon.
* Fjerne tett korrelerte features for å få renere og mer tolkbare koeffisienter.
* Kalibrere og vise sannsynlighetene, ikke bare klassen – mest interessant i jevne kamper.

### Gruppemedlemmer

Jan Sigurd Engh · Filip Floberg Jensen · Jonas Palmgren Hesmyr · Henrik Iversen

---

## 🇬🇧 English

### About

A binary classification project that predicts whether the **blue team wins** a League of Legends
match using **only the first 10 minutes** of game statistics. Built with logistic regression on
9,879 ranked Diamond games.

This is the submission for **Mandatory Assignment 2** (use case: option 4 – bring your own dataset).
The full analysis lives in [`MA2_LoL_Final.ipynb`](MA2_LoL_Final.ipynb).
**Note: the notebook itself is written in Norwegian.**

### Results

| Metric | Value |
| --- | --- |
| Accuracy (test) | **71.9%** |
| Baseline (always predict majority class) | 50.1% |
| Precision / Recall / F1 | 0.72 / 0.72 / 0.72 |
| Correct predictions | 1,421 of 1,976 games |

Errors are evenly split between the two classes (282 false wins, 273 false losses), so the model
is not biased toward either outcome.

### Dataset

* **Source:** [League of Legends Diamond Ranked Games (10 min)](https://www.kaggle.com/datasets/bobbyscience/league-of-legends-diamond-ranked-games-10-min) on Kaggle (bobbyscience)
* **Shape:** 9,879 games × 40 columns
* **Quality:** no missing values, no duplicate rows
* **Class balance:** 4,949 losses / 4,930 wins — essentially 50/50, which makes accuracy meaningful

The CSV is committed to the repo, so the notebook runs without any download.

### Approach

1. **Explore** the data: shape, dtypes, missing values, duplicates, class balance.
2. **Visualise** `blueGoldDiff` split by outcome. Gold difference separates the classes clearly
   (around ±1,500) but the distributions overlap near zero — close games are genuinely hard.
3. **Select features**, going from 40 columns to 16:
   * dropped `gameId` (an identifier with no predictive value);
   * dropped every `red*` column, since they mirror the blue ones (`redKills` = `blueDeaths`,
     `redGoldDiff` = −`blueGoldDiff`) and blue's diff columns already encode the red team;
   * dropped derived columns `blueGoldPerMin`, `blueCSPerMin`, `blueEliteMonsters`.
4. **Choose the algorithm** — logistic regression: the target is binary, all features are numeric,
   it returns a *probability* rather than a bare label, and its coefficients are easy to interpret.
5. **Split** 80% train (7,903) / 20% test (1,976), stratified, `random_state=42`.
6. **Train** a `StandardScaler` + `LogisticRegression(max_iter=1000)` pipeline. Scaling matters
   because gold is in the thousands while dragons are 0 or 1.
7. **Evaluate** with accuracy vs. baseline, a confusion matrix and a classification report.
8. **Interpret** the scaled coefficients to see which features push toward a win or a loss.
9. **Demo** a prediction: taking a median game and raising `blueGoldDiff`, `blueExperienceDiff`
   and `blueDragons` yields an 86% win probability for blue.

### Key findings

* **`blueGoldDiff` and `blueExperienceDiff` dominate.** A gold and experience lead at 10 minutes is
  the strongest signal for the eventual winner.
* Correlated features (e.g. `blueTotalGold` and `blueGoldDiff`) can take surprising signs because
  the model splits the effect between them — multicollinearity, not a data problem.
* Ten minutes is early. Comebacks and even matchups are effectively unpredictable from this data,
  which accounts for much of the remaining 28%.

### Getting started

```bash
git clone https://github.com/JanSigg/LoL10minPredictor.git
cd LoL10minPredictor

pip install pandas matplotlib scikit-learn jupyter
jupyter notebook MA2_LoL_Final.ipynb
```

Run all cells from the top (`Kernel → Restart & Run All`). Developed on **Python 3.12.7**.

### Repository contents

| File | Description |
| --- | --- |
| `MA2_LoL_Final.ipynb` | The full analysis: exploration, feature selection, training, evaluation, demo |
| `high_diamond_ranked_10min.csv` | The dataset (9,879 games) |
| `README.md` | This file |

### Future work

* Try Random Forest or gradient boosting and compare against logistic regression.
* Remove tightly correlated features for cleaner, more interpretable coefficients.
* Calibrate and surface the predicted probabilities, which are most interesting in close games.

### Authors

Jan Sigurd Engh · Filip Floberg Jensen · Jonas Palmgren Hesmyr · Henrik Iversen
