---
layout: page
title: Applying SSA to Lake Ice Phenology
description: Utilizing singular spectrum analysis (SSA) to analyze trends in the ice phenology of lakes in the Northern Hemisphere.
img: assets/img/oulujarvi3.jpg
importance: 1
category: work
related_publications: true
---

# About
Temporal trends in ice phenology of lakes are considered an important indicator of climate change. To study these trends, ice cover durations on lakes with over 100 years of data were pulled from the Global Lake and River Ice Phenology Database. After applying the symmetrizing logistic transformation on the ice cover duration data, singular spectrum analysis was used to estimate trends and periodic components from the data. Once these components were removed from the data, a similar process was repeated for the transformed absolute residuals, which can be treated as a proxy for volatility. Our analysis shows that lakes in Finland began experiencing a statistically significant change in ice cover duration starting around the year 2000. Patterns in ice cover duration and volatility are mainly dominated by 2- to 4-year periodic compoenents, yet the statistical significance of this trend is unknown. Temperature was explored as a possible variable for ice cover duration, showing strong correlation.

# Data
We pulled ice cover duration data on lakes and rivers from the Global Lake and River Ice Phenology Database maintained by the National Snow and Ice Data Center (NSIDC) {% cite nsidc2000 %}. In addition to ice cover duration data, this dataset also included freeze dates, thaw dates, latitude and longitude coordinates, a lake or river indicator, and country of origin. However, we primarily used ice cover duration data for our analysis. In order to get clean long run data, we applied a filter to remove lakes and rivers with under 100 years of data or with missing data of over 10 years in a row. This filtering resulted in 37 lakes from Finland, Canada, the United States, Sweden, and Russia. To symmetrize the residuals, we applied the logistic transformation to the ice cover duration data for each lake. The logistic transformation follows the equation

$$
y_{t} = \log\left( \frac{x_t - a}{b - x_t} \right),
$$

where $t$ is the year, $y_t$ is the transformed ice cover duration, $x_t$ is the empirical ice cover duration, and $a$ and $b$ are the lake specific lower and upper bounds of the ice cover duration, respectively. To account for the bounds of each dataset for the logistic transformation, we simply used 0.9 times the minimum ice cover duration as the lower bound and 1.1 times the maximum ice cover duration as the upper bound.

To check the correlation between temperature and ice cover duration, we pulled a global brightness temperature dataset of the lower troposphere from the National Oceanic and Atmospheric Administration (NOAA) {% cite noaa2015 %}.

# Methods
After applying the logistic transformation to the ice cover duration data, we passed the transformed time series through singular spectrum analysis. Singular spectrum analysis (SSA) is a statistical method for decomposing a time series into trend, periodic, and noise components without prior knowledge of the structure of the time series. Before utilizing SSA, the window length $L$ must be chosen. $L$ must take an integer value between $1$ and $N$, where $N$ is the length of the time series. If the length of the periodic component is known, $L$ should take on a value of some multiple of that period, not greater than $N/2$. When considering lake ice cover duration data, the window length value of $L$ was chosen to be $1$ as in yearly for each lake, as we had no prior intuition on the yearly periodicity of the data. 

Below is a brief explanation on the steps of SSA.

**Step 1: Embedding.** To perform embedding, map the time series $X = (x_1, \ldots, x_N)$ onto the trajectory matrix

$$
T = \begin{pmatrix}
        x_1 & x_2 & \cdots & x_K \\
        x_2 & x_3 & \cdots & x_{K+1} \\
        \vdots & \vdots &  & \vdots \\
        x_L & x_{L+1} & \cdots & x_N \\
    \end{pmatrix}
$$

where $K = N - L + 1$. This trajectory matrix has two important properties, being
1. the rows and columns of $T$ are both subseries of the original time series $X$, and
2. the antidiagonal values of $T$ are equal, making the trajectory matrix Hankel.

**Step 2: Singular value decomposition.** Performing singular value decomposition on $T$ yields $T = \sum_{i=1}^{L}\sqrt{\lambda_i}U_iV_i^T$, where $U_i$ are the left singular vectors of $T$, $V_i$ are the right singular vectors of $T$, and $\lambda_i$ are the eigenvalues of $TT^T$ such that $\lambda_1 \geq \cdots \geq \lambda_L \geq 0$. We also define $X_i = \sqrt{\lambda_i}U_iV_i$ for use in the next step.

**Step 3: Grouping.** Let $r' = \max(j:\lambda_j \neq 0 ) $. This $r'$ is determined from the data through use of the w-correlation matrix and the plot of the magnitudes of $\log(\lambda_i)$. The w-correlation matrix consists of the absolute weighted correlations between pairs of the elementary reconstructed components, $(X_i, X_j)$. As an example, the w-correlation matrix for Lake Oulujarvi of Finland has been provided below. Based on the noise in the plot, the $X_i$'s appear to be separable up to $i = 4$, implying $r'=4$ for the ice cover duration data for Lake Oulujarvi. This is corroborated by the plot of magnitudes of $\log(\lambda_i)$, where a change point at $i=5$ also suggests $r'=4$. We now consider all $\lambda_i$ where $i > r'$ to be equal to 0.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/oulujarvi4.jpg" title="Lake Oulujarvi W-Correlation Matrix" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The w-correlation matrix for Lake Oulujarvi.
</div>

Now that the nonzero eigenvalues have been identified, we can group eigenvalues that have multiple eigenvectors together. In the w-correlation matrix, the third and fourth eigenvalues appear to be highly correlated, while the first and second do not show any correlation with another eigenvalue. Let $I_s - \{s_1,s_2,\ldots,s_{n_s}\}$ be the set of indices in the $s$th group, and $[I_1,I_2,\ldots,I_r]$ be the $r$ groups, with $r \leq r'$. This suggests for Lake Oulujarvi, the grouping $[\{1\}. \{2\}, \{3,4\}]$ should suffice. This grouping is corroborated by the plot of the magnitudes of $\log(\lambda_i)$ by grouping singular values close together, which also group eigenvalues 3 and 4 together. The plot of paired eigenvalues can also be used to the same effect, where the formation of regular polygons or circles between two eigenvalues results in those eigenvalues being a pair. These shapes indicate periodicity in the data. For Lake Oulujarvi, paired eigenvalues 3 and 4 create a consistent shape between a square and a pentagon, indicating a periodicity between 4 and 5. The first and second eigenvalues do not form any sort of consistent shape, and thus are most likely effecting trend for the data. This grouping is consistent with that from the previous two plots. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/oulujarvi1.jpg" title="Oulujarvi Eigenvalues" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/oulujarvi3.jpg" title="Oulujarvi Paired Eigenvectors" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    More plots relating to Lake Oulujarvi. On the left is the plot of eigenvalues. On the right is the plot of paired eigenvectors.
</div>

Since the ice cover duration behaves differently for each lake, this grouping is specific to Lake Oulujarvi. This process is repeated to group the eigenvalues for each lake. It is important to note that some lakes had unpaired eigenvalues that showed sinusoidal character. These eigenvalues were omitted from the grouping to avoid confusion, as they brought a seasonal component to the trend that were not removed by SSA. 

**Step 4: Diagonal averaging.** After grouping, let $\tilde{X_s} = \sum_{i\in I_s}X_i$. For Lake Oulujarvi, this looks like $(\tilde{X_1},\tilde{X_2},\tilde{X_3}) := (X_1,X_2,X_3+X_4)$. For each $\tilde{X_s}$ for $s=1,\ldots,r$ created, take the mean of each of the antidiagonal entries. Now, create a new matrix $\bar{X_s}$ with the same dimensions as $\tilde{X_s}$ having the respective mean on each of the antidiagonal entries. Note that $\bar{X_s}$ is a Hankel matrix, which allows us to reconstruct the time series from $\tilde{X_s}$ to $\hat{X_s} = (x_{s,1},\ldots,x_{s,N})$ for each $s$. This $\hat{X_s}$ is either a trend or periodic component of the original time series, $X$. Also, by defining $\hat{X} := \sum_{s=1}^r \hat{X_s}$, we can approximate the original time series $X$ simply by summing up the component time series. 

These steps were done in the R statistical programming language using the Rssa package. The output from SSA is the trend, periodic, and error components for each lake. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/recon_oulujarvi.jpg" title="Reconstructed Series for Lake Oulujarvi" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A plot of the reconstructed time series of trend (F1), periodicity (F2), and residuals as output from performing SSA on Lake Oulujarvi.
</div>

It is possible that the output error from SSA could have its own respective trend and periodicity. To account for this, we again applied the logistic transformation to the output residual component for each lake. The method for obtaining the minimum and maximum parameters was the same for both iterations of the logistic transformation. The transformed residuals were then passed through a second iteration of SSA. The output from this second iteration of SSA did not seem to have any noticeable patterns in their trend or periodic components.

# Results

After obtaining the trend, periodic, and residual components from the first iteration of SSA on the transformed ice cover duration data, we analyzed the trend component in isolation to see if there were any notable patterns between the different lakes. We first grouped the lakes by region to see if each region of lakes had similar trends in their ice coverage duration. To achieve this grouping, we plotted the locations of the lakes using the longitude and latitude coordinates provided for each lake by the dataset obtained from the Global Lake and River Ice Phenology Database. We made two plots of this type, one for the lakes in North America, and one for the lakes in Europe. For the lakes in Europe, the grouping simply ended up being one group for each country. That is, there was a group of lakes in Finland, Sweden, and Russia. For the lakes in North America, there ended up being three separate groups, all around the area of the Great Lakes. 

After the lakes were grouped into 6 different regions, plots of the mean, median, and 95% quantiles for the trends for all the lakes in a region were made. Based on these plots, Finland was the only region with a visible trend across all lakes. Because of this, the lakes were regrouped into two groups, one including all the lakes in Finland, and one including all the lakes that are not in Finland. With these new groups, we made graphs of the mean, median, and 95% quantiles of the trends for the new not Finland group. From these graphs, it was visually apparent that the lakes in Finland uniformly follow a trend, while the rest of the regions do not.

## T-test

To determine whether the trends in the ice cover durations of the lakes were statistically significant or not, we performed t-tests on the trends of each lake. To calculate the t-statistic, $\text{T}_t$, we divided the transformed ice cover duration trend for each lake, $\mu_t$, by its standard deviation, $\sigma_t$, for each year, $t$, as shown by

$$
    \theta_t = \frac{\mu_t}{\sigma_t}.
$$

Note that this test statistic, $\theta_t$, has an approximate standard normal distribution. The transformed trend of the ice cover duration, $\mu_t$, comes from the output of running SSA on the logistic transformed ice cover duration data, but the standard deviation, $\sigma_t$, must be calculated. To estimate the standard deviation, we utilized the median absolute deviation (MAD) via the equation

$$
    \sigma_t = c \times \text{MAD}_t,
$$

where $c$ is a scaled constant based on the distribution of the data. Since the data has an approximate standard normal distribution, we use the value of $c$ appropriate for normally distributed data, $c = 1.4826$. Finally, the MAD was calculated by squaring the inverse logistic function of the logistic transformed square-rooted absolute residuals. This calculation was done for each lake at each time, $t$, as shown by

$$
    \text{MAD}_t = \left( \frac{b\exp{(|r_t|)}+a}{1+\exp{(|r_t|)}} \right) ^2,
$$

where $a$ and $b$ are the minimum and maximum bounds previously calculated for the logistic transformation of the ice cover duration data for each lake, and $\text{abs}(r_t)$ is the absolute residual output from running SSA on the transformed ice cover duration data.

Based on the t-test, the trend of the Finnish lakes changed enough to be statistically significant around the year 2000, where the lakes in the rest of the regions did not end up having a statistically significant trend. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/finland_trend_tstat.jpg" title="Finland Trend T-Statistics" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/notfinland_trend_tstat.jpg" title="Finland Trend T-Statistics" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    T-statistics for the ice cover duration of Finland (left) and non-Finnish (right) lakes. 
</div>

# Discussion

## Temperature and Other Potential Factors

Surface air temperature has been shown to be highly correlated with lake ice cover duration, as well as ice thickness and general lake ice phenomena {% cite skowron2023diversity %} {% cite pociask2022long %}. As we were unable to find a suitable dataset of surface air temperature above all the lakes, we instead opted for a monthly global dataset of brightness temperature of the lower troposphere from the National Oceanic and Atmospheric Administration (NOAA) {% cite noaa2015 %}. Brightness temperature is defined as the measure of all electromagnetic energy emitting from an object, and is calculated via the inverse Planck function on the measured radiation. An important property of brightness temperature is that it is highly correlated to kinetic, or true temperature, so it can be used as a proxy to show change in heat. To show a correlation between temperature and lake ice cover duration, we created a scatter plot of the lake ice cover duration and the yearly average brightness temperature. A visible downward trend corroborates the negative correlation between lake ice cover duration and temperature. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tbicd.jpg" title="Brightness Temperature vs Ice Cover Duration" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A plot of the average yearly brightness temperature versus yearly ice cover duration. Surface area of each lake was also plotted in square kilometers.
</div>

To check if there was a relation between lake surface area and ice cover duration, we included a measure of the surface area for each lake in our plot of brightness temperature and ice cover duration. From this plot, no discernible pattern was could be found, leading to the conclusion that there is no correlation between lake surface area and ice cover duration. In a 2004 study by Williams, Layman, and Stefan, consistent correlation between ice events and lake surface area were found via single variable linear regression and factor analysis, but in comparison to the correlation with air temperature it was less significant {% cite williams2004dependence %}.

Besides surface air temperature, other climate forces such as specific humidity, short-wave, and long-wave radiation contribute to over half of lake water warming {% cite tong2023global %}. Lake ice, however, works as to shield the water from incoming radiation, as well as increasing the lake's albedo {% cite prowse1986relationship %}. With ice cover duration decreasing, the open-water season is extended, and lake water is subject to increased short-wave radiation, increasing in turn the water temperature. It has been shown that the change in water temperature during the months of ice break off are on average 1.4 times that during the open-water season due to earlier ice break off dates. {% cite li2022earlier %}. Surface lake water on Lake Superior has increased in temperature more than that of the surface air temperature by a significant amount due to this effect {% cite austin2007lake %}.

However, this warming of Lake Superior's surface water is not consistent with other lakes in the Northern Hemisphere. Due to accelerated evaporation rates, lake water has overall been warming at a slower rate than that of the surface air temperature {% cite tong2023global %}.

We were also curious whether salinity could be a factor in determining lake ice cover duration. However, most of if not all of the lakes studied in this dataset are freshwater lakes, so no connection between salinity and lake ice cover duration could be measured through our dataset. The importance of salinity in determining lake ice phenomena may be low, as it has been found that more important factors relate more to climate and air temperature {% cite kropavcek2013analysis %}. 
