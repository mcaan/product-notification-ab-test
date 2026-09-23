# A/B Testing New Product Category Notifications

A product experimentation case study evaluating whether an in-app promotional notification for a new product category increases purchasing behavior without creating an unacceptable increase in app uninstalls.

The analysis recreates and extends a course-provided A/B testing example in Python, with additional experiment-health checks, business-impact analysis, covariate-adjusted robustness checks, and exploratory treatment-effect heterogeneity.

## Project Overview

Decco, an online home-decor retailer, introduced a new **Lamps** category and tested an in-app promotional notification designed to increase awareness and purchase intent.

Users were randomly assigned to:

* **Control:** no promotional Lamps notification
* **Treatment:** received the promotional Lamps notification

The experiment included **100,000 users**, split evenly between treatment and control.

The primary question was whether the notification increased transaction rate. Because Decco also places significant value on retaining installed users, uninstall rate was treated as a key guardrail rather than evaluating the experiment on conversion alone.

### Experiment metrics

* **Primary metric:** Transaction rate
* **Funnel diagnostic:** Add-to-cart rate
* **Secondary metric:** Average purchase value among purchasers
* **Guardrail:** Uninstall rate
* **Business outcome:** Revenue per randomized user

The experiment was designed at a **5% significance level** and **80% power**, using a historical transaction rate of **10.1%** and a **20% relative minimum detectable effect (MDE)**. This implied a requirement of approximately **3,792 users per group (7,584 total)**, well below the realized sample of 50,000 users per group.

## Results Summary

The notification produced a substantial increase in purchasing behavior, but it also materially increased the uninstall guardrail.

| Metric                      | Control | Treatment |       Effect |
| --------------------------- | ------: | --------: | -----------: |
| Transaction rate            |  10.12% |    17.52% | **+7.40 pp** |
| Add-to-cart rate            |  23.57% |    27.70% | **+4.13 pp** |
| Uninstall rate              |   2.99% |     5.03% | **+2.03 pp** |
| Revenue per randomized user |  $27.51 |    $56.67 |  **+$29.16** |

The transaction effect represented approximately a **73% relative lift**, substantially exceeding the pre-specified 20% relative MDE. The 95% confidence interval for the absolute transaction-rate effect was approximately **+6.98 to +7.83 percentage points**.

At the observed treatment effects, exposing 10,000 users to the notification corresponds to approximately:

* **740 additional purchasers**
* **$291,581 in incremental revenue**
* **203 additional uninstalls**

The experiment therefore revealed a meaningful **conversion–retention trade-off**: the notification generated substantial commercial upside while also increasing behavior associated with user loss.

## Experiment Design and Validation

Before estimating treatment effects, the analysis checked whether the experiment appeared healthy.

Allocation was exactly balanced at **50,000 control and 50,000 treatment users**, with no sample ratio mismatch. Observed pre-treatment characteristics were also very similar between groups:

| Covariate                | Control | Treatment |
| ------------------------ | ------: | --------: |
| Active in prior 6 months |  0.7502 |    0.7498 |
| Days since activity      |  135.08 |    135.39 |

These checks do not independently prove successful randomization, but they provide no obvious evidence of an allocation or implementation problem in the observed data.

## Data

The analysis uses a course-provided dataset containing **100,000 app users** observed during the experiment. Variables include treatment allocation, recent user activity, days since activity, add-to-cart behavior, transactions, purchase value, and uninstall outcomes.

The dataset was provided as instructional material through Preeti Semwal's Udemy course **Complete Course on A/B Testing**. The original course analysis uses R; this project reworks the experiment in Python and extends the analysis.

The source dataset is **not redistributed in this repository**. Users with legitimate access to the course materials can place the source file at:

`data/_AB_Test_Data.csv`

```text
data/_AB_Test_Data.csv
```

See [`data/README.md`](data/README.md) for data-access instructions.

## Methodology

### Experiment health

The analysis first validates the intended 50/50 allocation, checks for sample ratio mismatch, examines missingness, and compares observed pre-treatment covariates across experiment groups.

### Treatment effects

Binary experiment outcomes are compared using differences in proportions, two-proportion z-tests, and 95% confidence intervals.

The analysis evaluates:

* transaction rate;
* add-to-cart rate; and
* uninstall rate.

Because treatment assignment was randomized, these **unadjusted treatment-control differences are the primary causal estimates**.

### Revenue analysis

Average purchase value among purchasers is examined descriptively. However, purchasing occurs after treatment assignment and is itself affected by treatment, so conditioning only on purchasers does not provide the cleanest overall causal measure of monetization.

The analysis therefore also calculates **revenue per randomized user**, assigning zero revenue to users who do not purchase. This retains all randomized users and provides a more appropriate overall experiment-level monetization measure.

Revenue per randomized user increased from approximately **$27.51 to $56.67**, an absolute difference of **$29.16 per user**.

### Practical impact

Statistical significance is translated into estimated business impact at a scale of 10,000 treated users.

The observed experiment effects imply approximately **3.6 incremental purchases per incremental uninstall** and approximately **$1,436 in incremental revenue per incremental uninstall**.

These quantities make the trade-off more concrete, but they are not a complete economic decision rule. A true break-even analysis would require information such as contribution margin and expected customer lifetime value lost when a user uninstalls.

### Covariate-adjusted robustness checks

Logistic regression models adjust the transaction, add-to-cart, and uninstall outcomes for:

* prior six-month activity; and
* days since activity.

The adjusted models were consistent with the unadjusted experiment results: treatment remained strongly associated with higher transaction, add-to-cart, and uninstall rates.

Because treatment was randomized but these baseline covariates were not, their coefficients are interpreted as conditional associations rather than causal effects.

### Exploratory treatment-effect heterogeneity

An exploratory interaction analysis examined whether the treatment's effect on uninstall behavior differed according to prior user activity.

The observed uninstall treatment effect was:

* **Previously inactive users:** +0.60 percentage points
* **Previously active users:** +2.51 percentage points

The treatment-by-activity interaction had **p = 0.009**, suggesting that the uninstall effect differed across these groups in this dataset.

This analysis was motivated after observing the experiment results rather than pre-specified. It is therefore treated as **hypothesis-generating**, not as sufficient evidence for a production targeting rule.

## Recommendation and Next Experiment

The results do not support a broad rollout of the tested notification **in its current form**.

The experiment demonstrates substantial commercial potential: transaction rate increased from **10.12% to 17.52%**, and the observed revenue effect corresponds to approximately **$291,581 in incremental revenue per 10,000 users exposed**.

At the same time, uninstall rate increased from **2.99% to 5.03%**, equivalent to approximately **203 additional uninstalls per 10,000 treated users**. Given the business objective of protecting long-term user value, that guardrail deterioration creates a material retention concern.

The appropriate next step is therefore to **iterate rather than abandon the underlying notification strategy**. A follow-up experiment should test changes to notification messaging, timing, frequency, or targeting with the objective of preserving as much of the transaction and revenue lift as possible while materially reducing uninstall risk.

Prior activity status should be incorporated into the next experiment as a **pre-specified dimension of analysis**. The exploratory heterogeneity result provides a useful hypothesis, but it should be independently validated before being used for targeting decisions.

## Repository Structure

```text
AB-Testing-New-Product-Category-Notifications/
│
├── data/
│   ├── README.md
│   └── _AB_Test_Data.csv       # local only; ignored by Git
│
├── AB Testing - Decco.ipynb
├── .gitignore
├── LICENSE
├── README.md
└── requirements.txt
```

The repository intentionally remains compact because the analysis is contained in a single end-to-end notebook and does not generate processed datasets, models, or external output files.

## Reproducing the Analysis

1. Clone the repository.

2. Create and activate a Python environment.

3. Install the required packages:

```bash
pip install -r requirements.txt
```

4. Obtain `_AB_Test_Data.csv` through the referenced Udemy course materials.

5. Place the file at:

```text
data/_AB_Test_Data.csv
```

6. Open `AB Testing - Decco.ipynb` from the repository root and run the notebook from top to bottom.

The notebook uses a repository-relative data path and will raise an informative error if the expected dataset is not present.

## Limitations

Several limitations should be considered when interpreting the results:

* The available data do not contain the contribution margins or customer lifetime value information needed to fully monetize the conversion–uninstall trade-off.
* Average purchase value among purchasers conditions on a post-treatment outcome and is therefore treated as descriptive rather than the primary causal revenue estimate.
* The exploratory activity-segment interaction was not pre-specified and should be validated in a subsequent experiment before informing targeting decisions.
* Observed covariate balance and the exact 50/50 allocation support experiment-health checks but do not independently establish that every aspect of randomization or experiment implementation was error-free.
* The dataset originates from instructional course material, so conclusions should be interpreted as a product experimentation case study rather than evidence about a currently operating retailer.

## Tools

* Python
* pandas
* NumPy
* SciPy
* statsmodels
* Matplotlib
* seaborn
* Jupyter Notebook

## License

The original code and analysis in this repository are available under the [MIT License](LICENSE).

The course-provided source dataset is not covered by this repository's license and is not redistributed here.
