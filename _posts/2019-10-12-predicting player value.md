---
title: "Predicting the value of football players using their Fifa statistics"
categories:
  - Blog
tags:
  - Python
  - Regression
  - Machine Learning
  - XGBoost
  - SHAP
description: "Ever thought some football players are seriously overvalued or that your favourite player in the lower leagues is underrated? Use regression modelling to find out"
excerpt: "Predicting the market values of football players"
header:
  teaser: assets/predicting player value_files/predicting player value_116_0.png.png
image: assets/predicting player value_files/predicting player value_116_0.png.png
---

<img src='{{site.baseurl}}/assets/predicting player value_files/fifa_img.png' style="width:200px;height:200px;">
<br>

### Introduction

>Ever thought some football players are seriously overvalued or that your favourite player in the lower leagues is underrated?

I recently came across a fantastically data-rich website called <a href="sofifa.com" target='\_blank'>sofifa.com</a>. It has data on every football player in each professional football league around the world, including their playing attributes, value, position, wage, preferred foot, team, country, contract length, weight..... the list is pretty endless.

I had been looking for some dataset to practice regression modelling techniques and this looked perfect. Most people use the famous <a href="https://www.kaggle.com/c/boston-housing" target='\_blank'>Boston housing data set</a> for this purpose, but I have lost count of the number of blog posts repeating the same analysis and repeating the same results (😴) so thought it would be more interesting to curate my own dataset - this also provided an excellent excuse to write a webscraper to collect the data (one of my favourite pastimes🙈).

After collecting the data (<a href='https://github.com/julian-west/sofifa-analysis/tree/master/data_collection' target='\_blank'>webscraper here</a>),I set about inspecting the dataset features and creating a model to predict the value of each player. There were three main questions I wanted to answer:


1. Can the value of a football player be predicted from their playing attributes and meta data?
2. Which factors are most important for determining a player's market value?
3. Which players are over or undervalued compared to the market?

This information could be beneficial for two main reasons:
- Football teams/scouts can identify whether a target player is currently over or undervalued and what price they should be willing to pay for a player of that quality (regardless of current form)
- Understanding which factors most affect value could inform up and coming players which skills are most in demand in the market and which skills they should focus on improving to increase their value.

Below is an executive summary of the findings of the regression analysis.

> Full code and notebook is available on [Github](https://github.com/julian-west/sofifa-analysis)  
To view the Jupyter notebook in nbviewer click [here](https://nbviewer.jupyter.org/github/julian-west/sofifa-analysis/blob/master/predicting%20player%20value.ipynb)


## Analysis Summary

Information about each professional football player, such as attribute scores (e.g. passing, accuracy, stamina, acceleration etc.) and additional information (e.g. age, position, potential etc.), was collected from the SoFIFA website for the 2019 season. This information was used to develop a model to predict the value of each player.

The distribution of player market values is very positively skewed with most players having relatively low valuations (< &euro;1M), but with a few exceptional players commanding a significant premium. This variable was log transformed to normalize the data and create the target variable for modelling - log(value).

Initially, six regression models were tested on the full set of features (linear regression, multiple linear regression, decision trees, random forests and XGBoost) using GridSearchCV to tune the hyper-parameters. XGBoost performed the best with a root mean squared error (RMSE) of 0.18. After interpreting the model output, individual playing attributes were found to be statistically significant but not economically significant. The playing attributes were combined into more general categories to reduce the model complexity but preserve some of the information contained in these features. Using the XGBoost algorithm on the new feature set, the performance of the model was marginally, yielding a RMSE of 0.17.

<div markdown="0">
  <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>rmse</th>
      <th>adj. r2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>XGBoost FE</th>
      <td>0.078654</td>
      <td>0.996748</td>
    </tr>
    <tr>
      <th>XGBoost</th>
      <td>0.079354</td>
      <td>0.996672</td>
    </tr>
    <tr>
      <th>Random Forest</th>
      <td>0.167902</td>
      <td>0.985099</td>
    </tr>
    <tr>
      <th>Decision Tree</th>
      <td>0.197168</td>
      <td>0.979451</td>
    </tr>
    <tr>
      <th>Multiple Linear Regression</th>
      <td>0.250560</td>
      <td>0.967403</td>
    </tr>
    <tr>
      <th>Multiple Linear Regression - Backward Elimination</th>
      <td>0.252326</td>
      <td>0.967028</td>
    </tr>
    <tr>
      <th>XGBoost w/PCA</th>
      <td>0.278596</td>
      <td>0.958974</td>
    </tr>
    <tr>
      <th>Random Forest w/PCA</th>
      <td>0.425746</td>
      <td>0.904190</td>
    </tr>
    <tr>
      <th>baseline model (linear regression)</th>
      <td>0.438973</td>
      <td>0.900383</td>
    </tr>
    <tr>
      <th>Decision Tree w/PCA</th>
      <td>0.533455</td>
      <td>0.849580</td>
    </tr>
  </tbody>
</table>
</div>
  </div>



<img src='{{site.baseurl}}/assets/predicting%20player%20value_files/predicting%20player%20value_114_2.png'>


SHAP values were used to interpret the XGBoost model with overall rating, player potential and age found to be the most important features. Age had a non-linear relationship with the target variable with older players having a significantly lower predicted market value. Of the skills features, attacking skills (Crossing, finishing, short passing etc.) were the most important for increasing the predicted value.


<img src='{{site.baseurl}}/assets/predicting%20player%20value_files/predicting%20player%20value_116_0.png'>



<img src='{{site.baseurl}}/assets/predicting%20player%20value_files/predicting%20player%20value_116_1.png'>

The model was used to identify under and overvalued English players. The overvalued players included relatively famous names and/or players who had played for big clubs in the past. This suggests that popularity/reputation, which was not included as a feature, could inflate the value of players above their fundamental skill.


  <div markdown="0">
  <h3>Top 10 overvalued players from England</h3>
  </div>





  <div markdown="0">
  <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>country</th>
      <th>age</th>
      <th>overall_rating</th>
      <th>Position</th>
      <th>value_clean</th>
      <th>predicted_value</th>
      <th>difference</th>
      <th>difference_pct</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6206</th>
      <td>Dean Marney</td>
      <td>England</td>
      <td>34</td>
      <td>67</td>
      <td>midfield</td>
      <td>375000.0</td>
      <td>2.809261e+05</td>
      <td>9.407391e+04</td>
      <td>25.086375</td>
    </tr>
    <tr>
      <th>12116</th>
      <td>David Perkins</td>
      <td>England</td>
      <td>36</td>
      <td>65</td>
      <td>midfield</td>
      <td>170000.0</td>
      <td>1.299228e+05</td>
      <td>4.007724e+04</td>
      <td>23.574848</td>
    </tr>
    <tr>
      <th>1600</th>
      <td>Adebayo Akinfenwa</td>
      <td>England</td>
      <td>36</td>
      <td>66</td>
      <td>attack</td>
      <td>230000.0</td>
      <td>1.880056e+05</td>
      <td>4.199441e+04</td>
      <td>18.258437</td>
    </tr>
    <tr>
      <th>9161</th>
      <td>Matt Bloomfield</td>
      <td>England</td>
      <td>34</td>
      <td>63</td>
      <td>midfield</td>
      <td>180000.0</td>
      <td>1.494351e+05</td>
      <td>3.056489e+04</td>
      <td>16.980495</td>
    </tr>
    <tr>
      <th>12245</th>
      <td>Christopher Hamilton</td>
      <td>England</td>
      <td>23</td>
      <td>66</td>
      <td>defence</td>
      <td>925000.0</td>
      <td>7.827488e+05</td>
      <td>1.422512e+05</td>
      <td>15.378507</td>
    </tr>
    <tr>
      <th>5745</th>
      <td>Jack Cork</td>
      <td>England</td>
      <td>29</td>
      <td>76</td>
      <td>midfield</td>
      <td>7500000.0</td>
      <td>6.386116e+06</td>
      <td>1.113884e+06</td>
      <td>14.851793</td>
    </tr>
    <tr>
      <th>12855</th>
      <td>Tommy Rowe</td>
      <td>England</td>
      <td>29</td>
      <td>67</td>
      <td>midfield</td>
      <td>750000.0</td>
      <td>6.433922e+05</td>
      <td>1.066078e+05</td>
      <td>14.214367</td>
    </tr>
    <tr>
      <th>9205</th>
      <td>Darren Pratley</td>
      <td>England</td>
      <td>33</td>
      <td>66</td>
      <td>midfield</td>
      <td>375000.0</td>
      <td>3.225915e+05</td>
      <td>5.240850e+04</td>
      <td>13.975600</td>
    </tr>
    <tr>
      <th>7652</th>
      <td>Jabo Ibehre</td>
      <td>England</td>
      <td>35</td>
      <td>59</td>
      <td>attack</td>
      <td>70000.0</td>
      <td>6.067787e+04</td>
      <td>9.322133e+03</td>
      <td>13.317333</td>
    </tr>
    <tr>
      <th>5331</th>
      <td>Levi Sutton</td>
      <td>England</td>
      <td>21</td>
      <td>60</td>
      <td>defence</td>
      <td>290000.0</td>
      <td>2.520315e+05</td>
      <td>3.796853e+04</td>
      <td>13.092597</td>
    </tr>
  </tbody>
</table>
</div>
  </div>


  <div markdown="0">
  <h3>Top 10 undervalued players from England</h3>
  </div>





  <div markdown="0">
  <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>country</th>
      <th>age</th>
      <th>overall_rating</th>
      <th>Position</th>
      <th>value_clean</th>
      <th>predicted_value</th>
      <th>difference</th>
      <th>difference_pct</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9168</th>
      <td>Gary O'Neil</td>
      <td>England</td>
      <td>35</td>
      <td>67</td>
      <td>midfield</td>
      <td>180000.0</td>
      <td>2.733090e+05</td>
      <td>-93308.96875</td>
      <td>-51.838316</td>
    </tr>
    <tr>
      <th>13103</th>
      <td>Matty Blair</td>
      <td>England</td>
      <td>29</td>
      <td>65</td>
      <td>midfield</td>
      <td>400000.0</td>
      <td>4.881799e+05</td>
      <td>-88179.87500</td>
      <td>-22.044969</td>
    </tr>
    <tr>
      <th>7650</th>
      <td>Karl Henry</td>
      <td>England</td>
      <td>35</td>
      <td>67</td>
      <td>midfield</td>
      <td>180000.0</td>
      <td>2.159892e+05</td>
      <td>-35989.21875</td>
      <td>-19.994010</td>
    </tr>
    <tr>
      <th>12202</th>
      <td>Gareth Barry</td>
      <td>England</td>
      <td>37</td>
      <td>72</td>
      <td>midfield</td>
      <td>425000.0</td>
      <td>5.048729e+05</td>
      <td>-79872.87500</td>
      <td>-18.793618</td>
    </tr>
    <tr>
      <th>2263</th>
      <td>Luke O'Nien</td>
      <td>England</td>
      <td>23</td>
      <td>66</td>
      <td>midfield</td>
      <td>750000.0</td>
      <td>8.740792e+05</td>
      <td>-124079.25000</td>
      <td>-16.543900</td>
    </tr>
    <tr>
      <th>11344</th>
      <td>Harry Pell</td>
      <td>England</td>
      <td>26</td>
      <td>66</td>
      <td>midfield</td>
      <td>625000.0</td>
      <td>7.240376e+05</td>
      <td>-99037.56250</td>
      <td>-15.846010</td>
    </tr>
    <tr>
      <th>8471</th>
      <td>Tom Hateley</td>
      <td>England</td>
      <td>28</td>
      <td>68</td>
      <td>midfield</td>
      <td>725000.0</td>
      <td>8.392409e+05</td>
      <td>-114240.87500</td>
      <td>-15.757362</td>
    </tr>
    <tr>
      <th>12891</th>
      <td>George Francomb</td>
      <td>England</td>
      <td>26</td>
      <td>63</td>
      <td>midfield</td>
      <td>325000.0</td>
      <td>3.742320e+05</td>
      <td>-49231.96875</td>
      <td>-15.148298</td>
    </tr>
    <tr>
      <th>6714</th>
      <td>Nathan Byrne</td>
      <td>England</td>
      <td>26</td>
      <td>69</td>
      <td>midfield</td>
      <td>1000000.0</td>
      <td>1.150460e+06</td>
      <td>-150459.75000</td>
      <td>-15.045975</td>
    </tr>
    <tr>
      <th>10568</th>
      <td>James Horsfield</td>
      <td>England</td>
      <td>22</td>
      <td>63</td>
      <td>midfield</td>
      <td>400000.0</td>
      <td>4.576239e+05</td>
      <td>-57623.90625</td>
      <td>-14.405977</td>
    </tr>
  </tbody>
</table>
</div>
  </div>


## Future Work

- expose the machine learning model to a user interface (e.g. webapp ([dash](https://plot.ly/dash/) or [streamlit](https://streamlit.io/)) so that the user can input their own data and get a prediction for the value of a player with those attributes
- incorporate different features into the model such as current league/country or commercial factors such as social media following and brand appeal
- link the value of each player to their on pitch performance (e.g. number of career goals/clean sheets, number of goals last season etc.)
- collect data from different time periods to predict which players are likely to increase in value and what factors are most predictive
    - from example, collect data from 2/3 years ago and compare how the values of each football player (still playing) has changed.


## Resources

**Data**
- Original data scraped from the [SOFIFA website](www.sofifa.com)
- see *data_collection* folder in repository for scraper - note that it may not work if the website layout has been changed


**Articles**

- [Managing machine learning workflows - Matthew Mayo, KDnuggets](https://www.kdnuggets.com/2018/01/managing-machine-learning-workflows-scikit-learn-pipelines-part-3.html)
- [Interpretable Machine Learning - Scott Lundberg, TowardsDataScience](https://slundberg.github.io/shap/notebooks/Census%20income%20classification%20with%20XGBoost.html)
- [Shap-values - Kaggle](https://www.kaggle.com/dansbecker/shap-values)
- [Decision tree regressor explained - George Drakos](https://gdcoder.com/decision-tree-regressor-explained-in-depth/)
- [Feature selection techniques - Gabriel Azevedo, TowardsDataScience
](https://towardsdatascience.com/feature-selection-techniques-for-classification-and-python-tips-for-their-application-10c0ddd7918b)
- [Machine Learning data pipelines - TowardsDataScience](https://towardsdatascience.com/custom-transformers-and-ml-data-pipelines-with-python-20ea2a7adb65)
