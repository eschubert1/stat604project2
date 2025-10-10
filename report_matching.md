
To validate our findings through previous methods and visually confirm the age-related expression patterns identified through regression models, we implemented a propensity score matching approach.
This method serves two main purposes:
	1.	To check the robustness of our regression-based findings under a non-parametric framework.
	2.	To provide an intuitive visualization of gene expression changes that can be easily interpreted by collaborators with limited statistical background.

<!-- 
## Data Preparation 
	<!-- •	The dataset includes gene expression measurements across three brain regions:
	•	Anterior cingulate cortex (ANCg) 
	•	Dorsolateral prefrontal cortex (DLPFC)
	•	Cerebellum (CB)
	•	Each sample also has metadata: sex, lab, arrayversion, and age group (≥70 vs <70).
	•	Matching was conducted within each brain region to preserve biological context. --> 

### Matching Procedure

#### 1. Propensity score estimation

- A logistic regression model was used to estimate the probability of being in the older age group (≥70) as a function of the confounding variables:

$$
\operatorname{logit}\big(\Pr(\text{AgeGroup}=1)\big)
\;=\; \beta_0 
\;+
\; \beta_1\,\text{sex}
\;+
\; \beta_2\,\text{lab}
\;+
\; \beta_3\,\text{arrayversion}
$$

- This model produced a propensity score for each sample.

#### 2. 1:2 Nearest-Neighbor Matching

- Each older individual was matched to two younger individuals with the closest propensity scores (nearest neighbor).
- Matching was done without replacement to maintain balance.
- Caliper (maximum distance tolerance) was selected based on standardized mean differences to ensure acceptable covariate balance.

#### 3. Balance Assessment

- After matching, we evaluated covariate balance using standardized mean differences (SMD). A threshold of |SMD| < 0.1 was considered acceptable.

#### Balance plots (by region)


```{r, eval=T, fig.align='center', fig.cap="Covariate balance - ANCG"}
include_graphics("../figs/covariate-balance_ANCg.png")
```

```{r, eval=T, fig.align='center', fig.cap="Covariate balance - DLPFC"}
include_graphics("../figs/covariate-ballance_DLPFC.png")
```

```{r, eval=T, fig.align='center', fig.cap="Covariate balance - CB"}
include_graphics("../figs/covariate-balance_CB.png")
```

 These plots summarize standardized mean differences (SMD) after matching for `sex`, `lab`, and `arrayversion` separately in `ANCg`, `DLPFC` and `CB` brain regions. The balance plots show that after matching, the differences in sex, lab, and array version were completely eliminated (all standardized mean differences = 0). Although the overall dataset remains unbalanced and some information is inevitably lost during matching, the resulting covariate balance indicates that the matching procedure effectively created comparable groups. Therefore, the matching analysis can be viewed as a meaningful validation step and a useful complementary tool to support and interpret the regression-based findings.



Paired Comparison of Expression
	1.	Within each matched set (1 older sample + 2 younger samples), we computed pairwise differences in log-transformed expression levels.
\Delta_{ij} = \text{Expr}{\text{older},i} - \text{Expr}{\text{younger},j}
	2.	For each gene, the mean difference across matched pairs was summarized.
	3.	We visualized results through:
	•	Paired line plots, connecting each matched pair’s expression across age groups.
	•	Histograms of pairwise differences, to display the distribution and direction of age-related effects.

#### Example paired comparisons and distributions

##### ANCg (example genes — CD74)

```{r, eval=T, fig.align='center', fig.cap="ANCg — CD74 paired lines"}
include_graphics("../figs/ANCg_CD74_paired_lines.png")
```

_Analysis (paired lines)_: Each line connects matched samples (<70 vs ≥70). A consistent up/down slope across pairs indicates a coherent age-associated change; crossing or flat lines suggest heterogeneity or weak effects.

```{r, eval=T, fig.align='center', fig.cap="ANCg — CD74 differences per pair"}
include_graphics("../figs/ANCg_CD74_diffs_per_pair.png")
```

  _Analysis (pairwise differences)_: Distribution centered above/below zero indicates direction of the age effect; spread reflects variability across pairs. Narrow spread implies robust effect across matches.

  **Per-treatment mean for 1:2 matching**

  For each matched set i (one older: O_i; two younger: Y_{i1}, Y_{i2}), on the log-expression scale for gene g, define the per-set per-treatment mean difference as:

  $$
  d_{i,g} \,=\, \operatorname{Expr}_g(O_i) \, - \, \frac{\operatorname{Expr}_g(Y_{i1}) + \operatorname{Expr}_g(Y_{i2})}{2}.
  $$

  <!-- Average across the M matched sets within a region to obtain the gene-level summary:

  $$
  \bar d_g \,=\, \frac{1}{M} \sum_{i=1}^{M} d_{i,g}.
  $$ -->

  <!-- Intuition: first average the two younger expressions, then subtract this average from the older expression. The histogram shows the distribution of {d_{i,g}}; its center (positive/negative) indicates direction, and its spread indicates consistency across matched sets. -->

#### Region-level summaries

```{r, eval=T, fig.align='center', fig.cap="DLPFC — paired heatmap"}
include_graphics("../figs/DLPFC-paired-heatmap.png")
```

```{r, eval=T, fig.align='center', fig.cap="ANCg — top-genes heatmap"}
include_graphics("../figs/ANCg_heatmap_top12.png")
```
_Analysis_: Top `DLPFC` and `ANCg` genes post-matching. Heatmap highlights concordant up/down differences across pairs and genes. Block-like patterns indicate consistent signatures; noisy patterns imply modest effects. 

<!-- 
![ANCg — top 20 genes](src/plots_ANCg/ACNg_top20.png)

_Analysis_: Ranked `ANCg` signals for broader context. Use alongside FDR-regression results to confirm agreement in both sign and magnitude. -->

### Analysis and Interpretation

If regression results were reliable, matched-pair differences should display consistent directional trends (e.g., most pairs showing higher expression in the older group). Indeed, the paired plots showed upward trends for several top candidate genes (such as those identified with low FDR values). This agreement between regression and matching analyses supports the robustness of our findings. Furthermore, we include all the per treatment mean histogram plottings in the appendix as a reference for biological collaborators to check the results.