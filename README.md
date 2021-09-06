# Estimators of Prediction Intervals for Statistical and Machine Learning Forecasts

**Presented in Informs Annual Meeting 2020:**


Confirmation:
https://www.abstractsonline.com/notify/notifyintro.asp?MKey={87C214B4-B728-4685-BC07-53814C78751F}&NKey={C8871E23-254B-4797-AE8F-99618FF2B98A}&userFrom=

The full project is written in R

Sometimes (well, maybe most of the times) point forecasts are not enough.
There is no magic sphere to predict the future. 
Nor a universally accepted model which produces extremely accurate predictions (not yet at least....)
The future is uncertain.

In turn, forecasts are also uncertain. This uncertainty, when we translate forecasts into actions, needs to be measured.
Take for example demand forecasting. As forecasts are not 100% accurate we do not have knowledge about the exact demand. 
Thus, we must be prepared for the worst scenario. In the context of demand forecasting, a stock-out or an overstocked inventory.

Prediction intervals capture these uncertainties.
They provide an upper and lower limit, where the unknown future value is expected to lie in between (with a specific probability) 

This work presents and describes techniques for estimating predictions intervals for statistical models (such as ETS)
Gives the advantages, the limitations, and suggestions for their implementations

Most importantly, it demonstrates how some of these techniques can be applied for predictions of machine learning models.
The results of an experiment show that some methods can successfully be transferred in the machine learning domain.

