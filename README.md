# Smokers Cost 3.8x More. The Affordable Care Act Caps Their Premium at 1.5x.

**What drives medical charges, and how well do the factors an insurer is legally allowed to price on track actual cost?**

An Excel and Tableau analysis of 1,338 insurance policyholders.

<img width="1577" height="873" alt="smoker_dashboard" src="https://github.com/user-attachments/assets/f70fe507-d0b1-4de8-8c75-c82620f8a766" />

---

## Key findings

- **Smoking is the dominant cost driver.** Smoking status alone explains about 62% of the variation in charges (r = 0.787). Age and BMI are a distant second and third.
- **Smokers incur 3.8x the average charges of non-smokers, but US law caps the tobacco surcharge at 1.5x.** Smokers are 20.5% of policyholders but account for 49.5% of all charges. Put in people: 5 smokers cost about the same as 19 non-smokers. Under the cap, non-smokers subsidize part of smokers' costs.
- **The smoking penalty grows sharply with obesity.** In the normal and overweight BMI ranges, smokers cost about 2.6x to 2.7x what non-smokers in the same range cost. At BMI 30 and above, that gap widens.

## The business question

**what can an insurer legally charge for, and how well do those factors line up with what people actually cost?**

Under the Affordable Care Act, individual-market premiums may vary only by:

| Allowed factor | Limit |
|---|---|
| Age | Up to 3:1 (oldest vs. youngest adult) |
| Tobacco use | Up to 1.5:1 |
| Geographic rating area | Set by state |
| Family size | Per-member |

Sex and health status cannot be used. BMI is not an allowed rating factor. So the strongest cost drivers in the data are not all legal pricing levers, and the one that is (tobacco) is capped well below its cost effect.

Sources: [45 CFR 147.102](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-147/section-147.102), [HealthCare.gov: How insurance companies set health premiums](https://www.healthcare.gov/how-plans-set-your-premiums/). This is general context, not legal advice.

## Data and cleaning

**Source:** [Medical Cost Personal Datasets (Kaggle)](https://www.kaggle.com/datasets/mirichoi0218/insurance). 1,338 rows, 7 columns: age, sex, BMI, children, smoker, region, charges.

- **Record ID.** Assigned as a primary key before any sorting, then pasted as values so it can't shift. Used to trace rows and find duplicates.
- **Encodings.** Smoker coded 0/1 through a dimension table and pulled in with XLOOKUP. Sex coded 0/1. Region is nominal, so it was split into one 0/1 dummy per region, with one region dropped as the baseline in any regression. BMI was binned into WHO categories through a lookup table (XLOOKUP with approximate match on the lower bound).
- **Duplicates.** Screened with a sum of the numeric columns plus conditional formatting, checked by eye, then confirmed with a TEXTJOIN key across every column except the record ID (a sum alone can collide and ignores text columns). All three methods agree: exactly one duplicate pair, records 196 and 582.
- **Duplicate decision: kept and footnoted.** Nine other records share the same profile (age 19, male, BMI 30-35), the charge is low and unremarkable, and with no patient ID, identical values can't prove an error. In a real engagement, I'd check a significant duplicate with the data owner.

## Method

1. **Correlation ranking.** Each variable's correlation with charges. Smoker is point-biserial (0/1 vs. charges). Each region dummy is checked separately, because correlations can't be summed into one "region" number.
2. **Multiple regression** (Excel Data Analysis ToolPak). Started with smoker, age, BMI, children and sex, then dropped sex (not significant).
3. **Smoker cost ratio.** Average charges for smokers vs. non-smokers, compared with the 1.5x legal cap.
4. **Mean vs. median.** Charges are right-skewed (a small number of very large claims), so both were compared within each smoker group.
5. **Smoker x BMI.** Average charges by WHO BMI category, split by smoker status, to check for an interaction the additive regression can't show.
6. **Age.** Charges vs. age, paneled by BMI category and colored by smoker status.

## Results

### 1. Correlation with charges

| Variable | r | r² |
|---|---|---|
| Smoker | 0.787 | 0.62 |
| Age | 0.299 | 0.09 |
| BMI | 0.198 | 0.04 |
| Southeast | 0.074 | |
| Children | 0.068 | |
| Sex | 0.057 | |
| Southwest | -0.043 | |
| Northwest | -0.040 | |
| Northeast | 0.006 | |

Smoker dominates. Age and BMI are secondary. Everything else is close to zero on its own.

### 2. Regression

Final model: **charges ~ smoker + age + BMI + children** (n = 1,338).

- **R² = 0.75.** Four factors explain three quarters of the variation in charges.
- **Smoker** adds about 2.8x a typical non-smoker's average charges, holding age, BMI and children fixed (95% CI 2.7x to 2.9x).
- **Age:** each additional year adds about 2% of the overall average charge.
- **BMI:** moving up one WHO category (5 BMI points) adds about 12% of the overall average charge, with age and smoking held fixed.
- **Children:** each child adds about 4% of the overall average charge. Children showed almost no correlation alone (r = 0.068) but became significant once age and smoking were controlled for, because those removed the noise that hid it. Small, but it maps to family size, an allowed rating factor.
- **Sex** was not significant (p = 0.70) and was dropped. Adjusted R² stayed at 0.749 and the other coefficients held steady.

A lesson from an earlier refit: accidentally dropping age as well cut R² by about 9 points and inflated the BMI and children coefficients. That's omitted-variable bias in action, and it's why age stays in the model.

### 3. Smokers vs. non-smokers

| | Share of policyholders | Share of charges |
|---|---|---|
| Smokers | 20.5% | 49.5% |
| Non-smokers | 79.5% | 50.5% |

- **Mean:** smokers average **3.8x** the charges of non-smokers.
- **Median:** smokers' typical charge is **4.7x** the non-smoker typical charge.

The mean is the headline because it's what drives total cost and premiums. It's also the conservative number: the non-smoker mean is pulled above its median by a few large one-off claims, which narrows the gap. Either way, the cost ratio is far above the 1.5x a smoker can legally be charged.

### 4. Smoking and BMI

Average charges by BMI category, smokers vs. non-smokers. The ratio column is the one to read:

| BMI category | Non-smoker avg | Smoker avg | Smoker / non-smoker | Smoker n |
|---|---|---|---|---|
| Underweight (<18.5) | $5,533 | $18,810 | 3.4x | 5* |
| Normal (18.5-25) | $7,600 | $19,942 | 2.6x | 50 |
| Overweight (25-30) | $8,348 | $22,379 | 2.7x | 72 |
| Obesity 1 (30-35) | $8,493 | $39,204 | 4.6x | 74 |
| Obesity 2 (35-40) | $9,621 | $42,757 | 4.4x | 52 |
| Obesity 3 (40+) | $8,268 | $45,468 | 5.5x | 21 |

\* Only 5 underweight smokers; treat that bin as unreliable.

- **Non-smokers** stay roughly flat across BMI categories.
- **Smokers** are roughly flat below BMI 30, then jump: average charges in the obesity categories are roughly double those of smokers below BMI 30.

Smoking and obesity compound rather than simply add. The regression treats them as separate, additive effects, so this interaction is something it misses.

### 5. Age

Charges rise steadily with age for everyone. Colored by smoker status, smokers form two separate bands, split at BMI 30: smokers under 30 sit in a middle band, and smokers at 30 and above sit in the top band. Paneling by BMI category makes that split visible. Age is a legal rating factor (capped at 3:1), and it does track cost.

## Limitations

- **Simulated data.** This dataset is widely understood to be simulated from US demographic statistics for Brett Lantz's *Machine Learning with R*, not real claims. The patterns are illustrative. The legal framing is real.
- **Correlation is not causation.** The regression controls only for the variables in the data.
- **BMI is a blurry proxy.** It uses the same cutoffs for both sexes, while healthy body-fat ranges are higher for women.
- **Right skew.** Charges have a long right tail, so the mean sits above the median. Both are reported where it matters.
- **Small bins.** Only 5 underweight smokers and 21 smokers at BMI 40+.
- **Duplicate kept** without proof either way (no patient ID).
- **No premium data.** The comparison is cost ratio vs. the legal premium cap, not vs. actual premiums charged. It also ignores admin costs and the medical loss ratio.
- **Legal context** is general knowledge from public sources, not legal advice.

## Tools

- **Excel:** XLOOKUP, TEXTJOIN, AVERAGEIFS, PivotTables, conditional formatting, Data Analysis ToolPak (correlation, regression).
- **Tableau:** Data Visualization

All analysis was done in Excel then visualized in Tableau.

## Next steps

- **Bootstrap confidence intervals** (Excel RANDBETWEEN resampling with a Data Table), since normal-theory intervals are shaky on skewed charges.
- **Part 2: insurer financials.** Do insurer margins and medical cost ratios from 10-K filings reflect the pricing limits found here?
