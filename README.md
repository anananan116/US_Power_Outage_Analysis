# Analysis of Power Outages in the US

## Data Import

```python
df = pd.read_excel('outage.xlsx').drop(['variables', 'OBS'], axis = 1)
```

### Introduction and Question Identification

This dataset provides very detailed information about the power outages within US from January 2000 to July 2016, recording 1534 reports of power outage. The dataset mainly consists of 4 parts: general information, regional climate information, outage events information, and regional electricity consumption information. I've considered many questions to be asked about this dataset, including the places and time that major outage happens, the relationship between electricity price and probability of having an outage... But I found most of them to be hard to answer with a few hypothesis testing. Thus, the question we will disscuss here is how factors like season, climate, and cause of outage affect the seriousness of power outage, and we will answer this question by a few hypothesis testing in the end of the project. This anlaysis could be very important for electricity companies to identify potential problems and avoid serious power outage in their service. To answer this question, we will focus on these variables in our analysis:
<table>
    <thead>
        <tr>
            <th>Variable Name</th>
            <th>Description</th>
        </tr>
    </thead>
        <tbody>
            <tr>
                <td>YEAR</td>
                <td>Indicates the year when the outage event occurred</td>
            </tr>
            <tr>
                <td>MONTH</td>
                <td>Indicates the month when the outage event occurred</td>
            </tr>
            <tr>
                <td>U.S._STATE</td>
                <td>Represents all the states in the continental U.S</td>
            </tr>
            <tr>
                <td>ANOMALY.LEVEL</td>
                <td>This represents the oceanic El Niño/La Niña (ONI) index referring to the cold and warm episodes by season. It is estimated as a 3-month running mean of ERSST.v4 SST anomalies in the Niño 3.4 region (5°N to 5°S, 120–170°W)</td>
            </tr>
            <tr>
                <td>CLIMATE.CATEGORY</td>
                <td>This represents the climate episodes corresponding to the years. The categories—“Warm”, “Cold” or “Normal” episodes of the climate are based on a threshold of ± 0.5 °C for the Oceanic Niño Index (ONI)</td>
            </tr>
            <tr>
                <td>CAUSE.CATEGORY</td>
                <td>Categories of all the events causing the major power outages</td>
            </tr>
            <tr>
                <td>CAUSE.CATEGORY.DETAIL</td>
                <td>Detailed description of the event categories causing the major power outages</td>
            </tr>
            <tr>
                <td>OUTAGE.DURATION</td>
                <td>Duration of outage events (in minutes)</td>
            </tr>
            <tr>
                <td>DEMAND.LOSS.MW</td>
                <td>Amount of peak demand lost during an outage event (in Megawatt)</td>
            </tr>
            <tr>
                <td>CUSTOMERS.AFFECTED</td>
                <td>Number of customers affected by the power outage event</td>
            </tr>
            <tr>
                <td>RES.PRICE</td>
                <td>Electricity price for residencial use</td>
            </tr>
    </tbody>
</table>

## Cleaning and EDA

### Data Cleaning

We first throw away cloumns that we don't need for the analysis as mentioned above. We also drop 9 rows that critical infomation are missing.(We will not do any analysis on them since almost all variables are missing in those rows and it's hard for us to determine which type of missingness they belong to) We also assign some new variables(Season) to the dataframe for further analysis.

```python
df_cleaned = df[variable_names]
df_cleaned = df_cleaned[pd.notna(df_cleaned['MONTH'])]
df_cleaned['SEASON'] = df_cleaned['MONTH'].apply(map_month_to_season)
df_cleaned.head()
```

|   YEAR |   MONTH | U.S._STATE   |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE | SEASON   |
|-------:|--------:|:-------------|----------------:|:-------------------|:-------------------|:------------------------|------------------:|-----------------:|---------------------:|------------:|:---------|
|   2011 |       7 | Minnesota    |            -0.3 | normal             | severe weather     | nan                     |              3060 |              nan |                70000 |       11.6  | Summer   |
|   2014 |       5 | Minnesota    |            -0.1 | normal             | intentional attack | vandalism               |                 1 |              nan |                  nan |       12.12 | Spring   |
|   2010 |      10 | Minnesota    |            -1.5 | cold               | severe weather     | heavy wind              |              3000 |              nan |                70000 |       10.87 | Fall     |
|   2012 |       6 | Minnesota    |            -0.1 | normal             | severe weather     | thunderstorm            |              2550 |              nan |                68200 |       11.79 | Summer   |
|   2015 |       7 | Minnesota    |             1.2 | warm               | severe weather     | nan                     |              1740 |              250 |               250000 |       13.07 | Summer   |

### Univariate Analysis

We will use OUTAGE.DURATION, DEMAND.LOSS.MW, CUSTOMERS.AFFECTED to determine the seriousness of the power outage, so we will first look at their distribution.

```python
nan_count = df_cleaned[['OUTAGE.DURATION', 'DEMAND.LOSS.MW', 'CUSTOMERS.AFFECTED']].isna().sum()
print("NaN Counts:\n", nan_count)

```

|       Variable     |   Nan count |
|:-------------------|----:|
| OUTAGE.DURATION    |  49 |
| DEMAND.LOSS.MW     | 702 |
| CUSTOMERS.AFFECTED | 441 |

Since too many demand loss is missing, we will not consider this variable when determining the seriousness of the power outage. Let's take a look at the distribution of the distribution of the rest of the variables.

```python
fig_duration = px.histogram(df_cleaned, x='OUTAGE.DURATION', title='OUTAGE.DURATION Distribution', nbins=40)
fig_duration.show()

fig_customers_affected = px.histogram(df_cleaned, x='CUSTOMERS.AFFECTED', title='CUSTOMERS.AFFECTED Distribution', nbins=40)
fig_customers_affected.show()
```

<iframe src="assets/EDA-1.html" width=800 height=600 frameBorder=0></iframe>
<iframe src="assets/EDA-2.html" width=800 height=600 frameBorder=0></iframe>

We see some outliers that avoid us to see the details of the lower protion of the data, let's remove them for better representation.

```python
def filter_outliers(column, lower_quantile=0.01, upper_quantile=0.95):
    lower_bound, upper_bound = column.quantile([lower_quantile, upper_quantile])
    return column.clip(lower=lower_bound, upper=upper_bound)

# Apply filtering to each column
filtered_duration = filter_outliers(df['OUTAGE.DURATION'])
filtered_customers_affected = filter_outliers(df['CUSTOMERS.AFFECTED'])

# Plotting histograms with filtered data
fig_duration_filtered = px.histogram(filtered_duration, title='OUTAGE.DURATION Distribution (Filtered)', nbins=40)
fig_customers_affected_filtered = px.histogram(filtered_customers_affected, title='CUSTOMERS.AFFECTED Distribution (Filtered)', nbins=40)

# Displaying the plots
fig_duration_filtered.show()
fig_customers_affected_filtered.show()

fig_duration_filtered.write_html('EDA-3.html',include_plotlyjs='cdn')
fig_customers_affected_filtered.write_html('EDA-4.html',include_plotlyjs='cdn')
```

<iframe src="assets/EDA-3.html" width=800 height=600 frameBorder=0></iframe>
<iframe src="assets/EDA-4.html" width=800 height=600 frameBorder=0></iframe>

We can now see how power outage duration and customer affected distribute clearly. We can choose a threshold for each data such that any outage that has worse impact than the threshold should be called a serious power outage. But before we do that, we will need to deal with the missingness of both variables above.

## Assessment of Missingness

### Missingness of Duration

I believe the missingness of duration data is NMAR since it's more possible that the electricity companies would not publish a lot of details about the outage if the duration of it is too short. While there could be other reasons for the duration to be missing, such as some power companies never report the duration of the outage while others always report all the details, but since the main reason of missingness is still the value itself, we determine the missingness is NMAR (And there's hardly any extra data that could make it MAR), and we simply fill in 1 for all missingness(0 and NaN).

```python
df_cleaned['OUTAGE.DURATION'] = df_cleaned['OUTAGE.DURATION'].apply(lambda x: 1 if (x==0 or pd.isna(x)) else x)
df_cleaned.head()
```

|    |   YEAR |   MONTH | U.S._STATE   |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE | SEASON   |
|---:|-------:|--------:|:-------------|----------------:|:-------------------|:-------------------|:------------------------|------------------:|-----------------:|---------------------:|------------:|:---------|
|  0 |   2011 |       7 | Minnesota    |            -0.3 | normal             | severe weather     | nan                     |              3060 |              nan |                70000 |       11.6  | Summer   |
|  1 |   2014 |       5 | Minnesota    |            -0.1 | normal             | intentional attack | vandalism               |                 1 |              nan |                  nan |       12.12 | Spring   |
|  2 |   2010 |      10 | Minnesota    |            -1.5 | cold               | severe weather     | heavy wind              |              3000 |              nan |                70000 |       10.87 | Fall     |
|  3 |   2012 |       6 | Minnesota    |            -0.1 | normal             | severe weather     | thunderstorm            |              2550 |              nan |                68200 |       11.79 | Summer   |
|  4 |   2015 |       7 | Minnesota    |             1.2 | warm               | severe weather     | nan                     |              1740 |              250 |               250000 |       13.07 | Summer   |

### Missingness of Customer Affected

Missingness of customer affected could be dependent on the duration of power outage since the electricity company may not publish detailed information if the duration of power outage is too short. We will do a permutation test to prove it. Note here that Missingness of Customer Affected is also NMAR, since the missingness is also realted to its own value, but it also has a very strong correlation with duration and time(year) of the power outage, we will only focus on its correlation with duration here.

Null Hypothesis: The distribution of 'OUTAGE.DURATION' is independent of the missingness in the 'CUSTOMERS.AFFECTED' column. This means that the duration of outages does not influence whether the number of affected customers is reported.

Alterative Hypothesis: The 'OUTAGE.DURATION' tends to be lower (shorter) for cases where 'CUSTOMERS.AFFECTED' data is missing. This suggests that shorter outages are more likely to have missing data on the number of customers affected.

We will use the difference in mean duration as test statistic.

```python
missing_affected = df_cleaned[['OUTAGE.DURATION', 'CUSTOMERS.AFFECTED']].copy()
missing_affected['missing_affected'] = pd.isna(missing_affected['CUSTOMERS.AFFECTED'])
observed = missing_affected.groupby('missing_affected')['OUTAGE.DURATION'].mean()
observed_stats = observed[False] - observed[True]
observed_stats
missing_affected_draw = missing_affected.copy()
missing_affected_draw['OUTAGE.DURATION'] = filter_outliers(missing_affected_draw['OUTAGE.DURATION'])
plot = px.histogram(missing_affected_draw, x='OUTAGE.DURATION', color='missing_affected', histnorm='probability', marginal='box',
             title="Outage Duration by Missingness of Customer Affected", barmode='overlay', opacity=0.6)
plot.show()
```

<iframe src="assets/MV-1.html" width=800 height=600 frameBorder=0></iframe>

```python
stats_test = []
for i in range(10000):
    test_permutation = np.random.permutation(missing_affected['missing_affected'])
    missing_affected['missing_affected'] = test_permutation
    test_observed = missing_affected.groupby('missing_affected')['OUTAGE.DURATION'].mean()
    test_stats = test_observed[False] - test_observed[True]
    stats_test.append(test_stats)
stats_test = np.array(stats_test)
print((stats_test > observed_stats).sum()/len(stats_test))
```

0.0028

We reject the null hypothesis with a alpha of 0.01, thus we can conclude that the missingness of CUSTOMERS.AFFECTED column is correlated with duration of the outage. Here's the empirical distribution of this test.

<iframe src="assets/MV-2.html" width=800 height=600 frameBorder=0></iframe>

This suggests that the customer affected variable is much more likely to be missing if the duration of the power outage is short. We should keep this in mind when we determine whether a power outage is serious with these two variables.

---

Then, let's see whether the missingness has to do with a less relevant variable like RES.PRICE.

Null Hypothesis: The distribution of 'RES.PRICE' is independent of the missingness in the 'CUSTOMERS.AFFECTED' column. This means that the residential electricity price does not influence whether the number of affected customers is reported.

Alterative Hypothesis: The 'RES.PRICE' tends to be lower for cases where 'CUSTOMERS.AFFECTED' data is missing. This suggests companies that provides cheaper residential electricity are more likely to have missing data on the number of customers affected.

We will use the same statistics for this test.

```python
missing_affected = df_cleaned[['RES.PRICE', 'CUSTOMERS.AFFECTED']].copy()
missing_affected['missing_affected'] = pd.isna(missing_affected['CUSTOMERS.AFFECTED'])
observed = missing_affected.groupby('missing_affected')['RES.PRICE'].mean()
observed_stats = observed[False] - observed[True]
plot = px.histogram(missing_affected, x='RES.PRICE', color='missing_affected', histnorm='probability', marginal='box',
             title="Residential Electricity Price by Missingness of Customer Affected", barmode='overlay', opacity=0.6)
plot.show()
plot.write_html('MV-3.html',include_plotlyjs='cdn')
```

<iframe src="assets/MV-3.html" width=800 height=600 frameBorder=0></iframe>

```python
stats_test = []
for i in range(10000):
    test_permutation = np.random.permutation(missing_affected['missing_affected'])
    missing_affected['missing_affected'] = test_permutation
    test_observed = missing_affected.groupby('missing_affected')['RES.PRICE'].mean()
    test_stats = test_observed[False] - test_observed[True]
    stats_test.append(test_stats)
stats_test = np.array(stats_test)
print((stats_test < observed_stats).sum()/len(stats_test))
```

0.284

We fail to reject the null hypothesis. Here's the distribution for this test.

<iframe src="assets/MV-4.html" width=800 height=600 frameBorder=0></iframe>

Obviously the missingness of customer affected does not has strong correlation with the price of residential electricity price. This suggests we do not need to consider this variable when working with rows that has missing custormer affected.

## Back to EDA

Since we already know the missingness of customer affected is highly correlated with the duration of power outage, we can now consider how to define whether a outage is serious and thus we can do bivariate analysis to provide some insight for hypothesis testing later. We decide to call a outage serious if its duration is more than 2000 minutes and customer affected is more than 100,000. Since we know there's correlation between missingness of customer affected and duration, we will just ignore the condition on customer affected if it is missing.(We will say it is serious if the duration is more than 2000 minutes alone.)

```python
df_cleaned['SERIOUSNESS'] = ((df_cleaned['OUTAGE.DURATION'] > 2000) & ((pd.isna(df_cleaned['CUSTOMERS.AFFECTED'])) | (df_cleaned['CUSTOMERS.AFFECTED'] > 100000)))
df_cleaned.head(10)
```

|    |   YEAR |   MONTH | U.S._STATE   |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY     | CAUSE.CATEGORY.DETAIL   |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE | SEASON   | SERIOUSNESS   |
|---:|-------:|--------:|:-------------|----------------:|:-------------------|:-------------------|:------------------------|------------------:|-----------------:|---------------------:|------------:|:---------|:--------------|
|  0 |   2011 |       7 | Minnesota    |            -0.3 | normal             | severe weather     | nan                     |              3060 |              nan |                70000 |       11.6  | Summer   | False         |
|  1 |   2014 |       5 | Minnesota    |            -0.1 | normal             | intentional attack | vandalism               |                 1 |              nan |                  nan |       12.12 | Spring   | False         |
|  2 |   2010 |      10 | Minnesota    |            -1.5 | cold               | severe weather     | heavy wind              |              3000 |              nan |                70000 |       10.87 | Fall     | False         |
|  3 |   2012 |       6 | Minnesota    |            -0.1 | normal             | severe weather     | thunderstorm            |              2550 |              nan |                68200 |       11.79 | Summer   | False         |
|  4 |   2015 |       7 | Minnesota    |             1.2 | warm               | severe weather     | nan                     |              1740 |              250 |               250000 |       13.07 | Summer   | False         |
|  5 |   2010 |      11 | Minnesota    |            -1.4 | cold               | severe weather     | winter storm            |              1860 |              nan |                60000 |       10.63 | Fall     | False         |
|  6 |   2010 |       7 | Minnesota    |            -0.9 | cold               | severe weather     | tornadoes               |              2970 |              nan |                63000 |       11.41 | Summer   | False         |
|  7 |   2005 |       6 | Minnesota    |             0.2 | normal             | severe weather     | thunderstorm            |              3960 |               75 |               300000 |        9.1  | Summer   | True          |
|  8 |   2015 |       3 | Minnesota    |             0.6 | warm               | intentional attack | sabotage                |               155 |               20 |                 5941 |       11.53 | Spring   | False         |
|  9 |   2013 |       6 | Minnesota    |            -0.2 | normal             | severe weather     | hailstorm               |              3621 |              nan |               400000 |       12.71 | Summer   | True          |

### Bivariate Analysis

We now can see the proportion of serious power outage with different conditions set.

```python
plot = px.bar(df_cleaned.groupby('CAUSE.CATEGORY')['SERIOUSNESS'].mean(), title = "Proportion of Serious Power Outage for Difference Outgae Causes")
plot.show()
```

<iframe src="assets/EDA-5.html" width=800 height=600 frameBorder=0></iframe>

We can see severe weather conditions and fuel supply emergency tend to results in serious power outage. We will compare public appeal and severe weather in hypothesis testing later sicen they have more obvious difference.

```python
plot = px.bar(df_cleaned.groupby('SEASON')['SERIOUSNESS'].mean(), title = "Proportion of Serious Power Outage for Difference Seasons")
plot.show()
```

<iframe src="assets/EDA-6.html" width=800 height=600 frameBorder=0></iframe>

Obviously more serious power outage happens in Fall, probably related to some extreme weather conditions in that season. We will also do a hypothesis test comparing Fall and Spring in the later section.

```python
plot = px.bar(df_cleaned.groupby('CLIMATE.CATEGORY')['SERIOUSNESS'].mean(), title = "Proportion of Serious Power Outage for Difference Climate Categories")
plot.show()
```

<iframe src="assets/EDA-7.html" width=800 height=600 frameBorder=0></iframe>

Climate conditions of this type seems to be able to give us too much insight about serious power outages. We will skip this question in the hypothesis testing.

### Interesting Aggregates

Though some of the cause categories are obviously not relevant to season, there's still some important insights we can get from this pivot table. We can see fuel supply emergency usually cause much more trouble in Spring and Winter. We can also confirm that the reason why more serious power outage happens in Fall is because of severe weather.

```python
df_cleaned.pivot_table(index = "SEASON", columns= "CAUSE.CATEGORY", values = "SERIOUSNESS", aggfunc="mean")
```

| SEASON   |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:---------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Fall     |           0         |                0        |            0.0253165 |           0 |        0        |         0.436709 |                       0         |
| Spring   |           0.0588235 |                0.5      |            0.0458015 |           0 |        0        |         0.328    |                       0.0263158 |
| Summer   |           0         |                0.333333 |            0.0212766 |           0 |        0.166667 |         0.282686 |                       0.133333  |
| Winter   |           0         |                0.529412 |            0.0175439 |           0 |        0.181818 |         0.362694 |                       0.0357143 |

## Hypothesis Testing

---

For the first hypothesis test, we will take a look at how season affects the seriousness of power outages.

Null hypothesis: Power outages happens in Spring and Fall are equally serious. (measured by the proportion of serious power outages out of all power outages.)
Alternative hypothesis: Power outages happens in Fall are more serious than in Spring. (measured by the proportion of serious power outages out of all power outages.)

I choose the difference in mean to be the test statistic, and significance level to be 0.01.

```python
HT_test = df_cleaned[['SEASON', 'SERIOUSNESS']].copy()
HT_test = HT_test[(HT_test['SEASON'] == 'Fall') | (HT_test['SEASON'] == 'Spring')]
observed = HT_test.groupby('SEASON')['SERIOUSNESS'].mean()
observed_stats = observed['Fall'] - observed['Spring']
stats_test = []
for i in range(100000):
    test_permutation = np.random.permutation(HT_test['SERIOUSNESS'])
    HT_test['SERIOUSNESS'] = test_permutation
    test_observed = HT_test.groupby('SEASON')['SERIOUSNESS'].mean()
    test_stats = test_observed['Fall'] - test_observed['Spring']
    stats_test.append(test_stats)
stats_test = np.array(stats_test)
print((stats_test > observed_stats).sum()/len(stats_test))
```

0.00103

The resulting p-value is 0.001, thus we can reject the null hypothesis under the significance level of 0.01 and conclude that Power outages happens in Fall are generally more serious than in Spring.

---

For the second hypothesis test, we will take a look at how causes of outage affects the seriousness of power outages.

Null hypothesis: Power outages due to severe weather and public appeal are equally severe. (measured by the proportion of serious power outages out of all power outages.)
Alternative hypothesis: Power outages due to severe weather are more severe than outages due to public appeal. (measured by the proportion of serious power outages out of all power outages.)

I choose the difference in mean to be the test statistic, and significance level to be 0.01.

```python
HT_test = df_cleaned[['CAUSE.CATEGORY', 'SERIOUSNESS']].copy()
HT_test = HT_test[(HT_test['CAUSE.CATEGORY'] == 'severe weather') | (HT_test['CAUSE.CATEGORY'] == 'public appeal')]
observed = HT_test.groupby('CAUSE.CATEGORY')['SERIOUSNESS'].mean()
observed_stats = observed['severe weather'] - observed['public appeal']
stats_test = []
for i in range(100000):
    test_permutation = np.random.permutation(HT_test['SERIOUSNESS'])
    HT_test['SERIOUSNESS'] = test_permutation
    test_observed = HT_test.groupby('CAUSE.CATEGORY')['SERIOUSNESS'].mean()
    test_stats = test_observed['severe weather'] - test_observed['public appeal']
    stats_test.append(test_stats)
stats_test = np.array(stats_test)
print((stats_test > observed_stats).sum()/len(stats_test))
```

0.00015

The resulting p-value is 0.00015, thus we can reject the null hypothesis under the significance level of 0.01 and conclude that Power outages due to severe weather are generally more severe than outages due to public appeal.
