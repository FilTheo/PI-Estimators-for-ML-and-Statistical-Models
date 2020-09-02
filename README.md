# Estimators of Prediction Intervals for Statistical and Machine Learning Forecasts

**Full Paper is available**

### Introduction 

Prediction intervals which usually accompany point forecasts(a single number) are typically used to capture the uncertainty of such forecasts as they provide an upper and lower limit, where the unknown future value is expected to lie in between, with a specific probability.  [Chatfield 1996] Despite the wide extend of the literature regarding point forecasts, methods for estimating Prediction Intervals are not as widely explored

Most commonly used methods, such as theoretical and simulation-based ones proposed by [Hyndman, 2008], [Hyndman et al, 2018] are heavily relied on unrealistic assumptions such as independent, identically and normally distributed errors. However, it should be noted that on real world applications, these assumptions are not viable and thus, using such methods can result in a higher risk of a stockout. [Trapero 2016]. Another alternative that overlaps the normality assumption, which is proven to be unrealistic [Trapero et al, 2019], is bootstrappingbased methods. In spite of the non normality dependence, assuming the errors are i.i.d. cannot be avoided.

A family of methods for measuring prediction intervals, where both the i.i.d. and the normality assumption is relaxed, is empirical methods. [Kourentzes 2019] [Isengildina et al, 2006] [Trapero 2016] . Despite their promising results and their development in areas such as prediction intervals theory and the financial risk management they are not as widely applied on safety stock calculations. [Trapero 2016]

**The aim of this work is to present an extensive review of the existing methods on computing prediction intervals for statistical models, along with offering
guidelines for applying empirical methods for machine learning approaches**

### Designing the Experiment

#### Models Used

###### State Space Models 
From the Statistical State Space Models, Exponential Smoothing(ETS) was selected, as it is the most popular statistical approach for demand forecasting. [Willemain et al., 2004]. It’s popularity has been because of its simple implementation and its relative good overall accuracy. [Trapero et al, 2019] For each time series used, the optimal parameters(Trend,Seasonality and type of Errors), in terms of corrected Akaike’s Information Criterion(AICc), were automatically picked. [Hyndman, 2008], [Hyndman et al, 2018]

###### Machine Learning Models
From the Machine Learning Model’s family, XGBoost(Extreme Gradient Boosting) was selected, as it has been dominating applied machine learning competitions, and on top of that, by using far fewer computational resources than other models [Chen et al, 2016]. Σuccessfully fitting and optimizing XGBoost for forecasting Time Series is not as simple as a state space model.

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

To  validate the performance of every method on the different time series, evaluation on a rolling forecasting origin with re-estimation was used, as it provides a more reliable assessment of each methods performance, since the evaluation is not depending entirely on a single h-step-ahead forecast, which might include outliers and irregularities.[Ord et al,2017].

![Χωρίς τίτλο4](https://user-images.githubusercontent.com/70523417/91962463-3eaf6680-ed15-11ea-8564-97981d310466.png)

Every dataset is splitted into a training and a test set, where test set includes the two final periods of each time series. Initially, each model will be fitted into the
training set and the intervals for the next period, t + 1, t + 2, ...t + h will be forecasted and evaluated against the first h values of the test-set. On the next step, the model will be re-fitted on an updated training set, which will also include the first observation from the previously defined test set. A new h-steps-ahead forecast will take place and will be evaluated against an updated test set. The new forecast is equivalent to values t + 2 to t + (h + 1) of the test set. This procedure is repeated until a final forecast for t + h to t + 2h is performed. [Ord et al,2017]

It is important to highlight that naturally the different time series, which will be used, have by definition different scales and units. As a result, this would be reflected on the Interval Scores produced for each time series. To overcome this issue, Geometric Mean Relative Absolute Error(GMRAE) [Athanasopoulos and Kourentzes, 2020] will be used,
to produce non-scale depending errors. GMRAE is a method which takes the Geometric Mean of the absolute values of the Interval score produced by each method, divided by the score of the theoretical formula on the ETS model, which will be used as benchmark. Let IS be the interval score and N the total number of origins:

![Χωρίς τίτλο5](https://user-images.githubusercontent.com/70523417/91964900-67852b00-ed18-11ea-8142-0e4ba81940ad.png)

where ISBench is the score of the theoretical method applied on the ETS and ISA the score of every other method. A Relative errors lower than 1 indicates that method A outperformed the benchmark one, while scores bigger than 1 show that method A produced better results. \

#### Data Used

Applying every method on a single time-series, would not give trustworthy results, as some methods might be bias to the properties of the single-time series selected. As a result, different time series will be used and the results will be aggregated. In total, 76 different monthly time series were picked, with each having different properties.

Moreover, to test how each method performs on different horizons, 89 quarterly time series were also selected, in addition to 88 weekly ones. Results would be compared per horizon on each set of time series with similar frequency. Finally, Mean Interval Scores per-horizon of each method on the three different sets of time-series, will be compared

### Methods for Statistical Models

#### Algebric-Theoretical

In general, for most Statistical Models, Prediction Intervals(PIs) are estimated from, 

![Χωρίς τίτλο6](https://user-images.githubusercontent.com/70523417/91965284-f003cb80-ed18-11ea-9265-5298fb5887e1.png)

The main problem with this approach is the estimation of sigma. For some models, theoretical formulas do exists for either calculating or estimating it. However, this is not true for every model and on top of that, because of the complexity of some formulas, especially for models with multiplicative errors, results are estimations and not actual values. In addition, this method is solely available on the ETS model and is based on the following realistic(and unrealistic) assumptions : 
 
* There is no bias in the measurement process, and the true values of the process are reflected on the data used to produce the intervals [Ord et al,2017] [Hyndman, 2008]
* There is no correlation between errors, neither on the same horizon, nor across all horizons [Ord et al,2017]
* The variance of the errors is constant and conditional to horizon h. [Ord et al,2017]
* The errors are normally distributed [Ord et al,2017] [Hyndman, 2008]

These four assumptions could be summarized by stating that errors are identical, independently and normally distributed with zero means and a constant variance.  Lastly, assuming that there is no bias in picking the model’s parameters and hence no extra uncertainty exists, is also necessary. Under these assumptions, forecast mean and variance could be calculated, using formulas given by [Hyndman, 2008]. However, assuming reality away in order to overcome difficulties in estimations is not recommended on real-world
applications.

#### Simmulation Based Methods

Another simple approach to get the desired intervals,which has no restrictions on the model used for the estimations, is simulating M future paths and for
every h, from the generated distribution of predictions, the desired intervals could be extracted [Hyndman, 2008].

![Χωρίς τίτλο7](https://user-images.githubusercontent.com/70523417/91966750-f5621580-ed1a-11ea-8e73-ba67011eb6a6.png)

For this method to be implemented, assuming future errors will follow a behaviour similar with historical ones, and that there is no bias in selecting the
model’s parameters, are needed. In addition, as with the theoretical-approaches, assuming the errors are identical and independently distributed by summarizing the three first assumptions defined on section 3.1, along with them being normally distributed, are also necessary. [Hyndman, 2008]. For each t = n + 1, ..., n + h and for every i = 1, 2, ...M, an et value is picked from the assumed Gaussian distribution, using a random number generator and is used to estimate each prediction yt_i, used for the equivalent distribution.

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
From the generated distribution the desired intervals are extracted similarly to the simulation family of methods. However, a major difference is that there no need for any assumptions regarding the distribution of the errors and the only fundamental underlying assumptions needed are summarized by stating that the errors are identical and independently distributed. 

For every yt+h_(i), instead of randomly selecting an error factor from a Gaussian Distribution, errors are picked through re-sampling, from the in-sample distribution of historical errors.[Hyndman, 2008]. Consequently, assuming that future errors have a similar behaviour with past errors and that the process of picking sample Mi is uncorrelated to picking sample Mi−1 are necessary. Assumptions more relaxed than assuming normal distributed errors. As with simulation-based methods, when a significant number of future paths has been generated, more than one approaches could be applied for the extraction of the PI from the distribution of the predictions

#### Empirical Methods

Finally, the last set family of methods, which has not been widely applied, is Empirical methods[Trapero 2016] [Isengildina et al, 2006] [Chatfield, 2000]. They
are based on the hypothesis that PIs can be produced by estimating historical errors. The main assumption necessary is that future errors follow approximately
the same distribution with historical ones. [Smith et al, 1988] showed that this is a reasonable and afeasible assumption. As a result, one major advantage
of empirical methods over the rest, is that they are not based on assumptions which are far away from reality and they can be applied on any type of distribution. However in order for accurate PIs to be estimated, a good sample size is necessary [Isengildina et al, 2006]

Initially, the in-sample errors, the difference between the prediction of the fitted model on the training set and the actual values, are calculated for every time period and their distribution is generated. It should be pointed out that insample errors are usually lower than test-set errors[Barrow and Kourentzes, 2016] which would be estimated and this might affect the performance of such methods. A common method to overcome this limitation is using a validation set which has not been used for fitting the model. However,in this work, because of the relative small size of the time series used, no validation set to generate the errors was used.

From the distribution of in-sample errors, two non-parametric methods would be use to extract the upper and lower quantile.

1. The **Direct Method** described before, which directly extract the values which lie between the 5% and the 95% observations. Will be notated as **Direct Empirical**
2. The second method is calculating the probability density function of the set of errors by using KDE as described in the simmulation based methods. As proposed by [Trapero 2016] and [Kourentzes 2019], Silverman’s bandwidth and Epanechnikov kernel is used. When the quantiles are extracted,the fitted model is used to produce the point forecast for the forecast horizon h, and the intervals are given by: PI = (MeanForecast + LowerInterval , MeanForecast + UpperInterval). This method will be called **Empirical Mean-KDE**

### Methods for  XGBoost

As far as XGBoost models are concerned, there are no theoretical formulas for the estimation of PIs. In addition, as presented bellow, simulation based methods do not work accordingly as they produce extremely narrow intervals:

![Χωρίς τίτλο11](https://user-images.githubusercontent.com/70523417/91969271-81c20780-ed1e-11ea-8935-5853e9225731.png)

Usually such narrow intervals indicate that no all sources of uncertainty are taken into consideration[Hyndman,2014]. For a fitted and hypertuned XGBoost model, the only sources of uncertainty are its initial values. Despite adjusting the hyperparameteres responsible for slowing down the model from converging, the results were still disappointing.
As a result, the only methods which will be used for XGBoost will be Empirical methods 

