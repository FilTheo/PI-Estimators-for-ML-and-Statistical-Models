# Estimators of Prediction Intervals for Statistical and Machine Learning Forecasts

**Full Paper is available**

### Introduction 

Prediction intervals which usually accompany point forecasts(forecasting a single number) are typically used to capture the uncertainty of these forecasts as they provide an upper and lower limit, where the unknown future value is expected to lie in between, with a specific probability.  [Chatfield 1996] Despite the wide extend of the literature regarding point forecasts, methods for estimating Prediction Intervals are not as widely explored

Most commonly used methods, such as theoretical and simulation-based ones proposed by [Hyndman, 2008], [Hyndman et al, 2018] are heavily relied on unrealistic assumptions such as independent, identically and normally distributed errors. However, it should be noted that on real world applications, these assumptions are not viable and thus, using such methods can result in a higher risk of a stockout. [Trapero 2016]. Another alternative that overlaps the normality assumption, which is proven to be unrealistic [Trapero et al, 2019], is bootstrapping-based methods. In spite of the non normality dependence, assuming the errors are i.i.d. cannot be avoided.

A family of methods for measuring prediction intervals, where both the i.i.d. and the normality assumption is relaxed, is empirical methods. [Kourentzes 2019] [Isengildina et al, 2006] [Trapero 2016] . Despite their promising results and their development in areas such as prediction intervals theory and the financial risk management they are not as widely applied on safety stock calculations. [Trapero 2016]

**The aim of this work is to present an extensive review of the existing methods on computing prediction intervals for statistical models, along with offering
guidelines for applying empirical methods for machine learning approaches**

### Designing the Experiment

#### Models Used

##### State Space Models 
From the Statistical State Space Models, Exponential Smoothing(ETS) was selected, as it is the most popular statistical approach for demand forecasting. [Willemain et al., 2004]. It’s popularity has been because of its simple implementation and its relative good overall accuracy. [Trapero et al, 2019] For each time series used, the optimal parameters(Trend,Seasonality and type of Errors), in terms of corrected Akaike’s Information Criterion(AICc), were automatically picked. [Hyndman, 2008], [Hyndman et al, 2018]

##### Machine Learning Models
From the Machine Learning Model’s family, XGBoost(Extreme Gradient Boosting) was selected, as it has been dominating applied machine learning competitions, and on top of that, by using far fewer computational resources than other models [Chen et al, 2016]. Successfully fitting and optimizing XGBoost for forecasting Time Series is not as simple as a state space model.

While for ETS the whole time series is given as input, with y being the actual value, for XGBoost a selection of the past values which will be used as features
is necessary. 

  ![Χωρίς τίτλο1](https://user-images.githubusercontent.com/70523417/91960142-330e7080-ed12-11ea-9c39-94cd206e13bf.png)


For every value yt, lag1 = yt−1, lag2 = yt−2, ..., lagn = yt−n, where n is the frequency of the given series. As with all Machine Learning Algorithms, an appropriate feature selection is necessary. For every time series used, Partial Autocorrelations Coefficients where calculated and the Partial Autocorrelations Function(PACF), which shows which
lags have the most significant correlation with the actual values yt was estimated [Hyndman et al, 2018]

![Χωρίς τίτλο2](https://user-images.githubusercontent.com/70523417/91960732-ee370980-ed12-11ea-8465-baf2b99a4854.png)

**Lags with Coefficients over 0.1 or lower than -0.1 were automatically selected as the features for each model fitted on a particular time series**

Another challenge of fitting an XGBoost model for forecasting Time Series is ”forcing” the model to ”learn” both the trend and the seasonality of the series, while understanding the variations of the seasonality per year, with a relative small amount of training observations, without overfitting. **To overcome this issue, time series were transformed into stationary and seasonally stationary by taking the appropriate number of differences and seasonal differences**. The Augmented Dickey-Fuller test [Hyndman et al, 2018] is used to determine the appropriate number of differences for each time series. When forecasting has been completed, the predictions are reversed back into their original form.

Finally to optimize the performance of XGBoost, successfully tuning its hyperparameters is fundamental.**An automatic Random Search on the generated hyperparameter’s grid, validated by a 5-fold Cross validation, was implemented for every fitted model on each given time series** [Probst et al., 2018]. The optimal hyperparameter’s combinations was selected in terms of Mean Squared Error(MSE).

#### Evaluation Method

To evaluate the PIs produced by the different methods, Interval Score was used [Kourentzes and Athanasopoulos, 2020],[Gneiting et al,2004]. With Interval Score, a narrow interval is rewarded, while a penalty relative to a is given, if the true value, is not included on the interval produced [Gneiting et al,2004].
Interval Score(IS) is given by: 

![Χωρίς τίτλο3](https://user-images.githubusercontent.com/70523417/91961992-a0230580-ed14-11ea-8f2f-cf87219d4d37.png)

Where x is the true value, u and l stand for the upper and lower interval, while ID(a,b) returns 1 if a > b, or in other words, when the true value lies outside
the forecasted interval, and 0 otherwise.

To  validate the performance of every method on the different time series, **evaluation on a rolling forecasting origin with re-estimation** was used, as it provides a more reliable assessment of each methods performance, since the evaluation is not depending entirely on a single h-step-ahead forecast, which might include outliers and irregularities.[Ord et al,2017].

![Χωρίς τίτλο4](https://user-images.githubusercontent.com/70523417/91962463-3eaf6680-ed15-11ea-8564-97981d310466.png)

Every dataset is splitted into a training and a test set, where test set includes the two final periods of each time series. Initially, each model will be fitted into the
training set and the intervals for the next period, t + 1, t + 2, ...t + h will be forecasted and evaluated against the first h values of the test-set. On the next step, the model will be re-fitted on an updated training set, which will also include the first observation from the previously defined test set. A new h-steps-ahead forecast will take place and will be evaluated against an updated test set. The new forecast is equivalent to values t + 2 to t + (h + 1) of the test set. This procedure is repeated until a final forecast for t + h to t + 2h is performed. [Ord et al,2017]

It is important to highlight that naturally the different time series, which will be used, have by definition different scales and units. As a result, this would be reflected on the Interval Scores produced for each time series. **To overcome this issue, Geometric Mean Relative Absolute Error(GMRAE) [Athanasopoulos and Kourentzes, 2020] will be used,
to produce non-scale depending errors.** GMRAE is a method which takes the Geometric Mean of the absolute values of the Interval score produced by each method, divided by the score of the theoretical formula on the ETS model, which will be used as benchmark. Let IS be the interval score and N the total number of origins:

![Χωρίς τίτλο5](https://user-images.githubusercontent.com/70523417/91964900-67852b00-ed18-11ea-8142-0e4ba81940ad.png)

where ISBench is the score of the theoretical method applied on the ETS and ISA the score of every other method. A Relative errors lower than 1 indicates that method A outperformed the benchmark one, while scores bigger than 1 show that method A produced better results. \

#### Data Used

Applying every method on a single time-series, would not give trustworthy results, as some methods might be bias to the properties of the single-time series selected. As a result, different time series will be used and the results will be aggregated. **In total, 76 different monthly time series were picked**, with each having different properties.

Moreover, to test how each method performs on different horizons, **89 quarterly time series were also selected, in addition to 88 weekly ones**. Results would be compared per horizon on each set of time series with similar frequency. Finally, Mean Interval Scores per-horizon of each method on the three different sets of time-series, will be compared

### Methods for Statistical Models

#### Algebric-Theoretical

In general, for most Statistical Models, Prediction Intervals(PIs) are estimated from, 

![Χωρίς τίτλο6](https://user-images.githubusercontent.com/70523417/91965284-f003cb80-ed18-11ea-9265-5298fb5887e1.png)

The main problem with this approach is the estimation of sigma. For some models, theoretical formulas do exists for either calculating or estimating it. However, this is not true for every model and on top of that, because of the complexity of some formulas, especially for models with multiplicative errors, results are estimations and not actual values. In addition, this method is solely available on the ETS model and is based on the following realistic(and unrealistic) assumptions : 
 
* There is no bias in the measurement process, and the true values of the process are reflected on the data used to produce the intervals [Ord et al,2017] [Hyndman, 2008]
* There is no correlation between errors, neither on the same horizon, nor across all horizons [Ord et al,2017]
* The variance of the errors is constant and conditional to horizon h. [Ord et al,2017]
* The errors are normally distributed [Ord et al,2017] [Hyndman, 2008]

**These four assumptions could be summarized by stating that errors are identical, independently and normally distributed with zero means and a constant variance.**  Lastly, assuming that there is no bias in picking the model’s parameters and hence no extra uncertainty exists, is also necessary. Under these assumptions, forecast mean and variance could be calculated, using formulas given by [Hyndman, 2008]. However, assuming reality away in order to overcome difficulties in estimations is not recommended on real-world
applications.

#### Simmulation Based Methods

Another simple approach to get the desired intervals,which has no restrictions on the model used for the estimations, is simulating M future paths and for
every h, from the generated distribution of predictions, the desired intervals could be extracted [Hyndman, 2008].

![Χωρίς τίτλο7](https://user-images.githubusercontent.com/70523417/91966750-f5621580-ed1a-11ea-8e73-ba67011eb6a6.png)

**For this method to be implemented, assuming future errors will follow a behaviour similar with historical ones, and that there is no bias in selecting the
model’s parameters, are needed. In addition, as with the theoretical-approaches, assuming the errors are identical and independently distributed by summarizing the three first assumptions defined on section 3.1, along with them being normally distributed, are also necessary. [Hyndman, 2008].** For each t = n + 1, ..., n + h and for every i = 1, 2, ...M, an et value is picked from the assumed Gaussian distribution, using a random number generator and is used to estimate each prediction yt_i, used for the equivalent distribution.

![Χωρίς τίτλο8](https://user-images.githubusercontent.com/70523417/91966986-44a84600-ed1b-11ea-84fd-d4f43db1ee06.png)

Out of the simmulated distributions, extracting the desired PI could be achieved by more than one methods. 

1. The 100(1−a)% PI for the forecast horizon h, could be directly extracted by approximating the a/2 and the 1−a/2 quantiles of the simulated set  by picking the values which lie between the 5% and the 95% observation. For example, for the hypothetical set of predictions, yt+h|t={1, 2, 3, ...100} the 95% PI is [5,95]. This method is will be called: **Direct Quant**
2. For every forecast horizon h, the mean forecast, which stands for the mean value of the distribution, is calculated. Afterwards, the spread of errors, in terms of their standard deviation, around the mean is multiplied by the conditional probability c and the PI is estimated by: PI(c) = (m - sc , m + sc). The method will be called **Mean-Sigma**
3. The third approach is similar to the first one, but instead of extracting the quantiles directly for the distribution of predictions, they are extracted from the
errors approximated around the distribution’s mean and then they are summed with the mean forecast as follows: PI(c) = (m+ l, m + u), where l is the lower and u the upper quantile. The method will be called **Mean-Direct**
4. Lastly, in the fourth method, the full prediction density is estimated with a kernel density estimator(KDE) applied on the set of errors around the mean forecast.[Silverman B., 1986]. From the estimated distribution the lower and upper quantile are approximated and are added to the mean forecast as described in the methods above. This will be the **Mean-KDE** method

Most of the methods are not explored when the estimation of the PI using simmulation method is needed. However, as each method has advantages and dissadvantages, exploring all 4 methods should be considered. Results of the application of the 4 methods on the simple AirPassengers timeseries:

![Χωρίς τίτλο9](https://user-images.githubusercontent.com/70523417/91968321-2b07fe00-ed1d-11ea-8d3b-0ad129842c9c.png)

![Χωρίς τίτλο10](https://user-images.githubusercontent.com/70523417/91968407-4bd05380-ed1d-11ea-83de-44af0d69e11d.png)

#### Bootstrap - Based Methods:

A family of methods which is based on the same approach described before, as by using simulations a distribution of predictions is estimated. [Hyndman, 2008].
From the generated distribution the desired intervals are extracted similarly to the simulation family of methods. **However, a major difference is that there no need for any assumptions regarding the distribution of the errors and the only fundamental underlying assumptions needed are summarized by stating that the errors are identical and independently distributed.**

For every yt+h_(i), instead of randomly selecting an error factor from a Gaussian Distribution, errors are picked through re-sampling, from the in-sample distribution of historical errors.[Hyndman, 2008]. Consequently, assuming that future errors have a similar behaviour with past errors and that the process of picking sample Mi is uncorrelated to picking sample Mi−1 are necessary. Assumptions more relaxed than assuming normal distributed errors. As with simulation-based methods, when a significant number of future paths has been generated, more than one approaches could be applied for the extraction of the PI from the distribution of the predictions

#### Empirical Methods

Finally, the last set family of methods, which has not been widely applied, is Empirical methods[Trapero 2016] [Isengildina et al, 2006] [Chatfield, 2000]. **They
are based on the hypothesis that PIs can be produced by estimating historical errors. The main assumption necessary is that future errors follow approximately
the same distribution with historical ones. [Smith et al, 1988] showed that this is a reasonable and afeasible assumption. As a result, one major advantage
of empirical methods over the rest, is that they are not based on assumptions which are far away from reality and they can be applied on any type of distribution. However in order for accurate PIs to be estimated, a good sample size is necessary [Isengildina et al, 2006]**

Initially, the in-sample errors, the difference between the prediction of the fitted model on the training set and the actual values, are calculated for every time period and their distribution is generated. It should be pointed out that insample errors are usually lower than test-set errors[Barrow and Kourentzes, 2016] which would be estimated and this might affect the performance of such methods. A common method to overcome this limitation is using a validation set which has not been used for fitting the model. However,in this work, because of the relative small size of the time series used, no validation set to generate the errors was used.

From the distribution of in-sample errors, two non-parametric methods would be use to extract the upper and lower quantile.

1. The **Direct Method** described before, which directly extract the values which lie between the 5% and the 95% observations. Will be notated as **Direct Empirical**
2. The second method is calculating the probability density function of the set of errors by using KDE as described in the simmulation based methods. As proposed by [Trapero 2016] and [Kourentzes 2019], Silverman’s bandwidth and Epanechnikov kernel is used. When the quantiles are extracted,the fitted model is used to produce the point forecast for the forecast horizon h, and the intervals are given by: PI = (MeanForecast + LowerInterval , MeanForecast + UpperInterval). This method will be called **Empirical Mean-KDE**

### Methods for  XGBoost

**As far as XGBoost models are concerned, there are no theoretical formulas for the estimation of PIs. In addition, as presented bellow, simulation based methods do not work accordingly as they produce extremely narrow intervals:**

![Χωρίς τίτλο11](https://user-images.githubusercontent.com/70523417/91969271-81c20780-ed1e-11ea-8935-5853e9225731.png)

Usually such narrow intervals indicate that no all sources of uncertainty are taken into consideration[Hyndman,2014]. For a fitted and hypertuned XGBoost model, the only sources of uncertainty are its initial values. Despite adjusting the hyperparameteres responsible for slowing down the model from converging, the results were still disappointing.
As a result, the only methods which will be used for XGBoost will be Empirical methods 

## Results

Methods are initially compared for every set of monthly, quarterly and weekly time-series, so aggregated results per forecast horizon h, can bepresented. Next, Mean-Interval scores of every method, along with the mean absolute scaled point-forecast error of both ETS and XGBoost model are presented.

### Results for Monthly series.

The mean values of the 76 monthly series(per horizon h) are presented bellow: 

![Χωρίς τίτλο12](https://user-images.githubusercontent.com/70523417/92088793-058afb00-edd6-11ea-9fe2-0c31475e689f.png)

**As it can be observed bootstrapping with a direct extraction of the PI produced the best scores across all methods. On the other hand,Empirical methods outperformed the algebric one and had similar scores with simulation-direct method.**

Βy comparing the scores between BootstrapDirect and DirectEmpirical :

![Χωρίς τίτλο13](https://user-images.githubusercontent.com/70523417/92089152-716d6380-edd6-11ea-97cb-17163005a574.png)

it can be pointed out that there is not a significant difference between the two and their difference is mainly because of some outliers series were in-sample errors were much smaller than out-sample. This would not be an issue if more observations were present and instead of in-sample errors, a validation-set errors was used.

Furthermore, despite XGBoost producing good results with the Direct extraction of the interval for smaller horizons, for the later months of the year it gets heavily outperformed. This is natural as the furthest ahead one is trying to forecast, the more uncertain he gets [Hyndman et al, 2018].

**A possible explanation for XGboost’s poor performance is that the automatic procedure used for feature selection and hypeparameters tuning, might not have worked for some time-series.** A manual optimization might have produced better results as presented bellow on figure 9 , but due to time-limitations this was not feasible.

![Χωρίς τίτλο14](https://user-images.githubusercontent.com/70523417/92089357-b8f3ef80-edd6-11ea-9bcd-c3a647657db2.png)

### Results for Weekly series.

A similar pattern seems to take place with the weekly time series, as presented on figure 10. **Bootstrap and simulation methods with a direct extraction of the PI produced the best overall results, with Empirical Direct method being pretty close.** What is important to highlight on both weekly and monthly data is that KDE methods are outperformed. On the other hand, XGBoost performed poorly for the later forecasted horizons as with the monthly time series.

![Χωρίς τίτλο15](https://user-images.githubusercontent.com/70523417/92089494-e3de4380-edd6-11ea-8ec2-cf47cca95be8.png)

### Results for Quartely series.

As presented bellow, theoretical methods outperform Empirical ones applied on the XGBoost model, while simulation with Direct extraction outperforms every other method. A different pattern than the previous set of time-series takes place. **The main reason for these differences is the smaller sample size of the quarterly series(204 less observations than monthly time-series and 137 less than weekly ones).** For such a smaller sample size, simulation based methods seem to produce the best results, while the theoretical method outperforms some of the approaches.

![Χωρίς τίτλο16](https://user-images.githubusercontent.com/70523417/92089659-2142d100-edd7-11ea-9927-e8b968d076ae.png)

## Discussion.

### On the Results

**In general, theoretical and simulations-based methods, which are relied on strong underlying assumptions did not produce the best results and their wide use is questioned. A Bootstrap method which overlaps the normality assumption produced the best results on the two sets of time-series. However, it should be reminded that bootstrap-methods require the i.i.d assumption and their implementation might lead to an increase in the stock-out risk. As a result, and despite their promising results, bootstrap-methods should be used with cautious.**

For time series with fewer observations, methods relying on the normality assumption outperformed empirical ones.However, despite their better results, such methods should be avoided, as this assumption is not close to reality. Instead, Bootstrap methods, which overlap the normality assumption could be used as their performance was not significantly outperformed.

**On monthly and weekly series, empirical methods which are based on realistic assumptions did not produce significantly worse results than the rest of the methods. In general, by considering the nature of Interval score as an evaluation function , slightly bigger scores are results of slightly wider intervals. When it comes to real-world application, where issues such as stock-out are extremely important to be avoided, having marginally bigger PIs which might be closer to reality, should be preferred instead of unrealistically tighter intervals produced by the other methods.**

Another advantage of empirical methods over simulation and bootstrap ones is computational expense. As simulations/bootstrap methods require M (which for purposes of this work,is set to 10.000) forecasts for every rolling-origin on Cross Validation, to generate the prediction distribution, they require twice the time of empirical methods. This might be an issue on relative bigger time-series

A limitation of Empirical methods, as pointed out by [Isengildina et al, 2006] and as proven by the results on the quarterly set of series, is that Empirical
methods require a relative bigger sample size in comparison with the rest of the methods.

**As far as XGBoost is concerned,as presented on Table 3 bellow, different explanations exist for the various set of time-series. On quarterly data, despite XGBoost having a smaller MASE than ETS and as mentioned before, the relative small data size is the reason empirical methods do not work properly. However, as it stands out, the effect of data size is more clear on XGBoost than ETS as the model has more difficulties in fitting on the given series**

![Χωρίς τίτλο17](https://user-images.githubusercontent.com/70523417/92090263-e42b0e80-edd7-11ea-81d0-9b3184de06f3.png)

For monthly and weekly series, and as it will be presented later on this section, a higher point forecast error contributes to the higher interval score of
XGBoost over ETS. For weekly series, both a smaller data size and a bigger MASE are responsible for XGBoost not producing optimal results.A possible explanation, as mentioned, might be the automatic procedure used to fit the XGBoost model.

**All in all, on the monthly time-series Empirical methods produced the best results from the different sets of series as data size was significantly bigger.
Generally, empirical methods do not try to assume reality away, as the rest of the methods, to produce optimal results and by considering their performance, their relative limited research and usage are questioned. For an accurate estimation of PI on real-world applications, where data-size is not an issue, empirical methods are recommended.**

### On the difference Between Direct Methods and KDE estimation.

Direct method, gets rid of the extreme observations on the two tails of the distribution, while KDE tries to smoothly include all the distributed values. This issue is highlighted on the figure bellow.

![Χωρίς τίτλο18](https://user-images.githubusercontent.com/70523417/92090608-4a179600-edd8-11ea-913b-b7d93f303155.png)

As it can pointed out from the figure above, density function tries to smoothly include all the observations of the distribution. Due to some asymmetries, some gaps on the distribution, it fails to do so and as a result, wider intervals are produced. As it can be concluded and as pointed out by [Isengildina et al, 2006] a bigger error sample size would result in a better fitted density function, as no such “gaps” would be present. KDE would then fit the distribution more accurately and according to literature’s suggestions, it would outperform the Direct Empirical method.

**To sum things up, using various methods to extract the PI from either the prediction or the error distribution is highly recommended and more than one methods should be considered as no method works perfectly for all time series.**

### Correlation Between Intervals and Point Forecasts

For each set of monthly,quarterly and weekly time series, the correlation between cross-validated interval scores and the equivalent absolute error has been explored

![Χωρίς τίτλο19](https://user-images.githubusercontent.com/70523417/92091082-e6419d00-edd8-11ea-8eda-d27efb8ee65c.png)
![Χωρίς τίτλο20](https://user-images.githubusercontent.com/70523417/92091089-e8a3f700-edd8-11ea-86df-e1b42e23bb1e.png)
![Χωρίς τίτλο21](https://user-images.githubusercontent.com/70523417/92091092-e9d52400-edd8-11ea-8d05-ab09ebd7faa5.png)

**As it can be pointed out, there is some correlation between interval scores and absolute errors. The results suggest that for an accurate interval to be produced, a model which produces the optimal point forecast should be used as bad point forecasts result in non optimal Intervals.**

An example of the most well-suited methods not producing the best results is presented bellow, where the interval scores of two ETS models are compared with the optimal model, in terms of AICc on a single time-series

![Χωρίς τίτλο22](https://user-images.githubusercontent.com/70523417/92091439-6f58d400-edd9-11ea-9319-8460eb4278e1.png)
![Χωρίς τίτλο23](https://user-images.githubusercontent.com/70523417/92091448-71229780-edd9-11ea-99ba-479198e3a9e1.png)

As described by the figures above and from the following table, every method has a Mean Relative Error of over 1.6 and no method on any model, for any forecast horizon, has outperformed the optimal model. A Mean Relative Error of over 1.75 for both is calculated across all methods.

![Χωρίς τίτλο 24png](https://user-images.githubusercontent.com/70523417/92091673-bd6dd780-edd9-11ea-9ec4-60de31296346.png)

**Picking a wrong model for the given series would not result in the most accurate point forecast, which is proven to be correlated with the estimation of PIs.**

## Conclusions

As this paper suggests, empirical methods which are based on more relaxed assumptions than approaches proposed by literature and don’t have a significant
difference on the results produced, should be preferred. Producing slightly bigger interval scores than bootstrap methods is a result of estimating realistically
wider intervals. Bootstrap methods which produced better and hence,tighter intervals, require assuming i.i.d errors, which on real-world demand forecasting, might not be reasonable. Making decisions based on intervals produced by methods requiring unrealistic assumptions may result in an over or under-stocking a company’s inventory

Another aspect of estimating prediction intervals which has been investigated is their correlation with the goodness of point forecasts. For an accurate PI to be produced, picking a model with high point forecast accuracy should be a priority.

Furthermore, a standardized approach to extract the desired quantiles from an estimated distribution should be avoided. As it has been discussed, a direct extraction of the quantiles gave the best results, but for a bigger sample size, using a kernel density estimator would be a better choice. As a result, more than one methods should be considered.

## References

* [Armstrong, 2017] Armstrong, J.S., 2017. Demand Forecasting II: EvidenceBased Methods and Checklists 36.
* [Athanasopoulos and Kourentzes, 2020] Athanasopoulos, G., Kourentzes, N.,2020. On the evaluation of hierarchical forecasts 23.
* [Barrow and Kourentzes, 2016] Barrow, D., Kourentzes, N., 2016. Distributions of forecasting errors of forecast combinations: implications for inventory management. International Journal of Production Economics 177, 24–33.
* [Chatfield 1996] Chatfield, C. 1996a, The Analysis of Time Series, 5th edn. London: Chapman and Hall.
* [Chatfield 1998] Chatfield C. , 1998. Prediction Intervals, Department of Mathematical Sciences, University of Bath
* [Chatfield, 2000] Chatfield, C., 2000. Time-series forecasting. CRC Press
* [Chen et al, 2016] Chen, T., Guestrin, C., 2016. XGBoost: A Scalable Tree Boosting System, in: Proceedings of the 22nd ACM SIGKDD Inter27 national Conference on Knowledge Discovery and Data Mining. Presented at the KDD ’16: San Francisco California USA, pp. 785–794. https://doi.org/10.1145/2939672.2939785
* [Gneiting et al,2004] Gneiting, T., Raftery, A.E., 2004. Strictly Proper Scoring Rules, Prediction and Estimation 30.
* [Hyndman, 2008] Hyndman, RJ 2008, Forecasting with Exponential Smoothing: The State Space Approach, Springer Series in Statistics, Springer, Berlin, viewed 13 May 2020
* [Hyndman,2014] Hyndman R.j , 2014, Prediction intervals too narrow[Blog] Available at:https://robjhyndman.com/hyndsight/narrow-pi/
* [Hyndman et al, 2018] Hyndman, R. J. , Athanasopoulos G. (2018) : Forecasting: Principles and Practice .
* [Isengildina et al, 2006] Isengildina, O., Irwin, S.H., Good, D.L., 2006. Empirical Confidence Intervals for WASDE Forecasts of Corn, Soybean and Wheat Prices (No. 18995), 2006 Conference, April 17-18, 2006, St. Louis, Missouri. NCR-134 Conference on Applied Commodity Price Analysis, Forecasting, and Market Risk Management.
* [Kourentzes 2019] Kourentzes, N., Athanasopoulos, G., 2019. Elucidate structure in intermittent demand series 38.
* [Kourentzes and Athanasopoulos, 2020] Kourentzes, N., Athanasopoulos, G., 2020. Elucidate structure in intermittent demand series. European Journal of Operational Research S0377221720304926. https://doi.org/10.1016/j.ejor.2020.05.046
* [Mishina et al., 2014] Mishina, Y., Tsuchiya, M., Fujiyoshi, H., 2014. Boosted random forest, in: 2014 International Conference on Computer Vision Theory and Applications (VISAPP). Presented at the 2014 International Conference on Computer Vision Theory and Applications (VISAPP), pp. 594–598.
[Morde, 2019] Morde, V., 2019. XGBoost Algorithm: Long May She Reign![WWW Document]. Medium. URL https://towardsdatascience.com/httpsmedium-com-vishalmorde-xgboost-algorithm-long-she-may-reinedd9f99be63d (accessed 6.25.20).
* [Ord et al,1997] Ord K, Kroehler B, Snyder D. , (1997), Estimation and prediction for a class of dynamic nonlinear statistical models, Journal of the American Statistical Association, 92,1621-1629
* [Ord et al,2017] Ord, K., Fildes, R., Kourentzes, N., 2017. Principles of Business Forecasting–2nd ed. wessex, inc., New York, NY.
* [Probst et al., 2018] Probst, P., Bischl, B., Boulesteix, A.-L., 2018. Tunability: Importance of Hyperparameters of Machine Learning Algorithms. arXiv:1802.09596 [stat].
* [Silverman B., 1986] Silverman B., 1986,Density estimation for statistics and data analysis,Chapman and Hall, London[78]
* [Smith et al, 1988] Smith, S.K., and Sincich, T.,1988 “Stability over Time in the Distribution of Population Forecast Errors.” Demography, 25, 3 : 461-474.
* [Trapero 2016] Trapero, J. (2016). Empirical safety stock estimation based on Kernel and GARCH models. working paper.
* [Trapero et al, 2019] Trapero, J.R., Card´os, M., Kourentzes, N., 2019. Quantile forecast optimal combination to enhance safety stock estimation. International Journal of Forecasting 35, 239–250. https://doi.org/10.1016/j.ijforecast.2018.05.009
* [Willemain et al., 2004] Willemain, T.R., Smart, C.N., Schwarz, H.F., 2004. A new approach to forecasting intermittent demand for service parts inventories. International Journal of Forecasting 20, 375–387. https://doi.org/10.1016/S0169-2070(03)00013-X
* [Williams and GoodMan, 1971] Williams, W. H., Goodman, M. L., 1971. A simple method for the construction of empirical confidence limits for economic forecasts. Journal of the American Statistical Association 66 (336), 752–754.
