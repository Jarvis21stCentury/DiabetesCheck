# DiabetesCheck

**Eight survey questions that tell you whether a blood-sugar test is worth getting.**
Veritas AI International High School Hackathon 2026 · Track: AI + Health

**Try it:** https://jarvis21stcentury.github.io/DiabetesCheck/
**Notebook:** [`DiabetesCheck.ipynb`](DiabetesCheck.ipynb) (or open it in [Colab](https://colab.research.google.com/github/Jarvis21stCentury/DiabetesCheck/blob/main/DiabetesCheck.ipynb))

> This project is a prototype and is not medical advice.

## The problem

About 1 in 3 US adults has prediabetes, and most don't know it. The usual first step is the CDC's paper risk quiz, which adds up points and sends anyone at 5 or more for a blood test. We asked whether a small model trained on survey answers could pick who needs a test better than that quiz, using only 8 questions.

## Results (test set, 50,736 adults, run once after the model was frozen)

| Method | Cases caught | Missed | Tested for nothing | AUROC |
|---|---|---|---|---|
| **DiabetesCheck** (8 questions, cut-off 0.40) | **86.7%** | **943** | **17,148** | **0.820** |
| CDC paper quiz (partial), 5+ points | 81.2% | 1,331 | 18,450 | 0.757 |
| Simple rule: BMI 30+ or high blood pressure | 86.7% | 940 | 23,311 | n/a |

Compared with the paper quiz: **388 fewer missed cases (29% fewer) and 1,302 fewer unneeded tests.**
The trade-off is still real: about a third of everyone is sent for a test that comes back fine. That's the price of catching 87% of cases.

## How it works

- **Data:** [CDC Diabetes Health Indicators](https://archive.ics.uci.edu/dataset/891/cdc+diabetes+health+indicators) (BRFSS 2015, 253,680 adults), loaded with `ucimlrepo`.
- **Split:** 60 / 20 / 20, stratified, seed 42. The test set was used once, after the model and cut-off were chosen.
- **Baselines:** always "no", the BMI-or-blood-pressure rule, and a partial version of the CDC Prediabetes Risk Test.
- **Model:** scikit-learn `HistGradientBoostingClassifier` (76 trees, `class_weight="balanced"`) on 8 of the survey's 21 questions: BMI, age group, sex, high blood pressure, high cholesterol, general health, physical activity, smoking.
- **Cut-off:** 0.40, picked on validation data to catch most cases.
- **Honest scores:** class weighting makes raw scores run high, so we never show them as a percent chance. The app shows "about X in 100 people with a similar score had diabetes or prediabetes", counted from validation data.
- **Checked by group:** results are broken down by age, sex and income in the notebook.

## The web app

[`index.html`](index.html) is one self-contained page. Section 13b of the notebook exports the trained trees into it and checks that the page's JavaScript gives exactly the same score as scikit-learn on 2,000 validation people (largest difference: 0.0). Answers never leave the browser.

## Limitations

- It can't diagnose anything. Only a blood test can.
- It missed 13% of cases on the test set.
- The survey is self-reported, from US adults in 2015.
- The survey didn't ask about family history or gestational diabetes, so our CDC comparison uses a partial version of the quiz without those questions.

## Run it yourself

Open the notebook in Colab and choose **Runtime → Run all** (about 8 minutes; no API keys or uploads needed).

## Credits

Built by Hima Shreyas Nagumalla and [partner's full name].
Data: CDC BRFSS 2015 via the UCI Machine Learning Repository. Libraries: scikit-learn, pandas, NumPy, ucimlrepo.
AI use: Claude and ChatGPT helped with code, debugging and the web interface. We can explain every part of it. Details are in the notebook.
