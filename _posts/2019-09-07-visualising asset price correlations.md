---
title: "Visualising asset price correlations"
categories:
  - Blog
tags:
  - Python
  - Visualisations
  - Plotly
  - Quantitative Finance
  - Network Analytics
---


<img src='{{site.baseurl}}/assets/visualising_stock_correlations_files/stock_correlations_gif.gif' style="width:500px;height:500px;">


### Topics covered
- __Calculating correlations between asset classes__
- __Heatmap visualisation using seaborn__
- __Network analysis and visualisation using Networkx__
- __Minimum spanning trees__
- __Interactive visualisations using Plotly__

----------------------------------
All code and raw data is available on this
<a href="https://github.com/julian-west/asset_price_correlations" target="_blank">Github repository</a>

The final interactive Plotly visualisation is embedded at the bottom of this page (
<a href="#plotly graph"> here</a>
  )

---------------------------------

### Introduction

Network analysis is becoming an ever increasingly popular method to analyse and visualise complex relationships in data in an intuitive way.

One interesting application of network analytics is visualising relationships between asset classes in the finance industry. We are always told that equities and bonds tend to move in [opposite directions](https://vanguardblog.com/2018/07/09/exploring-the-relationship-between-stocks-and-bonds/) and certain assets such as gold and the Japanese Yen are 'safe-haven' assets and therefore behave in similar ways. But what does that actually _look_ like and are there any other interesting relationships between asset classes? Networks can be a great way to convey this information at a high level and help understand broad market dynamics.

Information on the correlations between asset classes can be particularly useful for investors wanting to diversify their portfolio. Most people understand that portfolio risk can be mitigated by diversification, however, this may not always be the case. For instance if you 'diversified' your portfolio by investing in two different stocks which are strongly correlated with each other (i.e. if stock A goes up 2% then stock B also goes up 2%) then the overall risk of the portfolio has not been reduced as if there is a negative factor affecting stock A then it is also likely to affect stock B and both prices will trend downwards. Therefore, diversification can only be realistically achieved by investing in assets which are uncorrelated with each other. Good explanations of this phenomenon can be found at [Quantdare](https://quantdare.com/correlation-prices-returns/) and the [Investors Chronicle](https://www.investorschronicle.co.uk/managing-your-money/2018/10/11/how-to-diversify-in-a-correlated-world/).

In this post I will explain how we can visualise asset correlations as an interactive network diagram which allows the user to gain a high level overview of the relationships between asset classes. From this information they can identify areas of risk but also opportunities for diversification of their portfolio. We will cover the basics of how to calculate correlations between asset price returns using numpy and pandas, using networkx to create a network graph and finally use Plotly to display the network as an interactive visualisation.


### Library imports

- __numpy__ and __pandas__ to read and manipulate the raw data
- __networkx__ to generate the network representation of the data and create some initial graph visualisations
- __matplotlib__ and __seaborn__ for general visualisations and plot formatting
- __Plotly__ to create the final interactive visualisation


<script src="https://gist.github.com/julian-west/8a604f321ba9f3e2cc8897795986340c.js"></script>



  <div markdown="0">
          <script type="text/javascript">
        window.PlotlyConfig = {MathJaxConfig: 'local'};
        if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
        if (typeof require !== 'undefined') {
        require.undef("plotly");
        requirejs.config({
            paths: {
                'plotly': ['https://cdn.plot.ly/plotly-latest.min']
            }
        });
        require(['plotly'], function(Plotly) {
            window._Plotly = Plotly;
        });
        }
        </script>

  </div>


--------------------------

## Load data

For this analysis I will use a dataset containing the daily adjusted closing prices of 39 major ETFs which represent different asset classes covering equities, bonds, currencies and commodities. The data covers a period of 4 years between November 2013 and November 2017.

<script src="https://gist.github.com/julian-west/5a37d92139eb3ab18b0d24534552d08c.js"></script>


  ```
  There are 1013 rows and 39 columns in the dataset
Data timeperiod covers: 2013-11-01 to 2017-11-08

  ```

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
      <th>EOD~BND.11</th>
      <th>EOD~DBC.11</th>
      <th>EOD~DIA.11</th>
      <th>EOD~EEM.11</th>
      <th>EOD~EFA.11</th>
      <th>EOD~EMB.11</th>
      <th>EOD~EPP.11</th>
      <th>EOD~EWG.11</th>
      <th>EOD~EWI.11</th>
      <th>EOD~EWJ.11</th>
      <th>...</th>
      <th>EOD~VGK.11</th>
      <th>EOD~VPL.11</th>
      <th>EOD~VXX.11</th>
      <th>EOD~XLB.11</th>
      <th>EOD~XLE.11</th>
      <th>EOD~XLF.11</th>
      <th>EOD~XLK.11</th>
      <th>EOD~XLU.11</th>
      <th>EOD~CSJ.11</th>
      <th>EOD~FXF.11</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-11-08</th>
      <td>81.83</td>
      <td>16.40</td>
      <td>235.46</td>
      <td>46.78</td>
      <td>69.87</td>
      <td>114.60</td>
      <td>47.69</td>
      <td>33.18</td>
      <td>30.95</td>
      <td>60.02</td>
      <td>...</td>
      <td>58.20</td>
      <td>72.77</td>
      <td>33.53</td>
      <td>58.70</td>
      <td>69.82</td>
      <td>26.25</td>
      <td>64.01</td>
      <td>55.70</td>
      <td>104.96</td>
      <td>94.5100</td>
    </tr>
    <tr>
      <th>2017-11-07</th>
      <td>81.89</td>
      <td>16.43</td>
      <td>235.42</td>
      <td>46.56</td>
      <td>69.64</td>
      <td>114.65</td>
      <td>47.22</td>
      <td>33.07</td>
      <td>31.09</td>
      <td>59.65</td>
      <td>...</td>
      <td>58.17</td>
      <td>72.20</td>
      <td>33.52</td>
      <td>58.64</td>
      <td>70.16</td>
      <td>26.38</td>
      <td>63.66</td>
      <td>55.66</td>
      <td>105.01</td>
      <td>94.5400</td>
    </tr>
    <tr>
      <th>2017-11-06</th>
      <td>81.86</td>
      <td>16.53</td>
      <td>235.41</td>
      <td>46.86</td>
      <td>69.90</td>
      <td>115.26</td>
      <td>47.20</td>
      <td>33.34</td>
      <td>31.22</td>
      <td>59.18</td>
      <td>...</td>
      <td>58.67</td>
      <td>71.98</td>
      <td>33.34</td>
      <td>58.58</td>
      <td>70.25</td>
      <td>26.75</td>
      <td>63.63</td>
      <td>55.00</td>
      <td>105.00</td>
      <td>94.7500</td>
    </tr>
    <tr>
      <th>2017-11-03</th>
      <td>81.80</td>
      <td>16.22</td>
      <td>235.18</td>
      <td>46.34</td>
      <td>69.80</td>
      <td>115.42</td>
      <td>47.09</td>
      <td>33.39</td>
      <td>31.22</td>
      <td>59.19</td>
      <td>...</td>
      <td>58.58</td>
      <td>71.88</td>
      <td>33.66</td>
      <td>58.83</td>
      <td>68.68</td>
      <td>26.78</td>
      <td>63.49</td>
      <td>55.21</td>
      <td>105.00</td>
      <td>94.4400</td>
    </tr>
    <tr>
      <th>2017-11-02</th>
      <td>81.73</td>
      <td>16.12</td>
      <td>234.96</td>
      <td>46.58</td>
      <td>69.91</td>
      <td>116.15</td>
      <td>47.31</td>
      <td>33.50</td>
      <td>31.43</td>
      <td>59.05</td>
      <td>...</td>
      <td>58.69</td>
      <td>71.89</td>
      <td>33.71</td>
      <td>58.86</td>
      <td>68.48</td>
      <td>26.89</td>
      <td>62.99</td>
      <td>55.01</td>
      <td>105.04</td>
      <td>94.6299</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 39 columns</p>
</div>
  </div>



We can see that the csv file of asset prices contains 39 different assets and 1013 records for each asset. The time-series is in reverse order (newest to oldest) and there are no missing datapoints in the dataset. Each ETF in the dataset is encoded by its abbreviated ticker which makes it difficult to understand what sector each ETF actually represents. To make the ETF codes more readable I created an alias for each one which is more descriptive. We will rename the columns with these aliases so the future analysis is more interpretable.


<script src="https://gist.github.com/julian-west/c8b29048202ee66f1130a4d52292aea5.js"></script>



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
      <th>Code</th>
      <th>ETF Alias</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>EOD~BND.11</td>
      <td>Bonds Global</td>
    </tr>
    <tr>
      <th>1</th>
      <td>EOD~DBC.11</td>
      <td>Commodities</td>
    </tr>
    <tr>
      <th>2</th>
      <td>EOD~DIA.11</td>
      <td>DOW</td>
    </tr>
    <tr>
      <th>3</th>
      <td>EOD~EEM.11</td>
      <td>Emerg Markets</td>
    </tr>
    <tr>
      <th>4</th>
      <td>EOD~EFA.11</td>
      <td>EAFE ETF</td>
    </tr>
    <tr>
      <th>5</th>
      <td>EOD~EMB.11</td>
      <td>Emerg Markets Bonds</td>
    </tr>
    <tr>
      <th>6</th>
      <td>EOD~EPP.11</td>
      <td>Pacifix ex Japan</td>
    </tr>
    <tr>
      <th>7</th>
      <td>EOD~EWG.11</td>
      <td>Germany</td>
    </tr>
    <tr>
      <th>8</th>
      <td>EOD~EWI.11</td>
      <td>Italy</td>
    </tr>
    <tr>
      <th>9</th>
      <td>EOD~EWJ.11</td>
      <td>Japan</td>
    </tr>
    <tr>
      <th>10</th>
      <td>EOD~EWQ.11</td>
      <td>France</td>
    </tr>
    <tr>
      <th>11</th>
      <td>EOD~EWU.11</td>
      <td>UK</td>
    </tr>
    <tr>
      <th>12</th>
      <td>EOD~FXB.11</td>
      <td>GBP</td>
    </tr>
    <tr>
      <th>13</th>
      <td>EOD~FXC.11</td>
      <td>China Large Cap</td>
    </tr>
    <tr>
      <th>14</th>
      <td>EOD~FXE.11</td>
      <td>Euro</td>
    </tr>
    <tr>
      <th>15</th>
      <td>EOD~FXI.11</td>
      <td>China</td>
    </tr>
    <tr>
      <th>16</th>
      <td>EOD~FXY.11</td>
      <td>Yen</td>
    </tr>
    <tr>
      <th>17</th>
      <td>EOD~GDX.11</td>
      <td>Gold Miners</td>
    </tr>
    <tr>
      <th>18</th>
      <td>EOD~GLD.11</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>19</th>
      <td>EOD~IEF.11</td>
      <td>10yr treasuries</td>
    </tr>
    <tr>
      <th>20</th>
      <td>EOD~IYR.11</td>
      <td>US real estate</td>
    </tr>
    <tr>
      <th>21</th>
      <td>EOD~JNK.11</td>
      <td>High yield Bond</td>
    </tr>
    <tr>
      <th>22</th>
      <td>EOD~LQD.11</td>
      <td>Corp Bond</td>
    </tr>
    <tr>
      <th>23</th>
      <td>EOD~SLV.11</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>24</th>
      <td>EOD~SPY.11</td>
      <td>SNP 500 ETF</td>
    </tr>
    <tr>
      <th>25</th>
      <td>EOD~TIP.11</td>
      <td>Bonds II</td>
    </tr>
    <tr>
      <th>26</th>
      <td>EOD~TLT.11</td>
      <td>20+ Treasuries</td>
    </tr>
    <tr>
      <th>27</th>
      <td>EOD~USO.11</td>
      <td>Oil</td>
    </tr>
    <tr>
      <th>28</th>
      <td>EOD~UUP.11</td>
      <td>USD</td>
    </tr>
    <tr>
      <th>29</th>
      <td>EOD~VGK.11</td>
      <td>Europe</td>
    </tr>
    <tr>
      <th>30</th>
      <td>EOD~VPL.11</td>
      <td>Pacific</td>
    </tr>
    <tr>
      <th>31</th>
      <td>EOD~VXX.11</td>
      <td>VXX</td>
    </tr>
    <tr>
      <th>32</th>
      <td>EOD~XLB.11</td>
      <td>Materials</td>
    </tr>
    <tr>
      <th>33</th>
      <td>EOD~XLE.11</td>
      <td>Energy</td>
    </tr>
    <tr>
      <th>34</th>
      <td>EOD~XLF.11</td>
      <td>Finance</td>
    </tr>
    <tr>
      <th>35</th>
      <td>EOD~XLK.11</td>
      <td>Tech</td>
    </tr>
    <tr>
      <th>36</th>
      <td>EOD~XLU.11</td>
      <td>Utilities</td>
    </tr>
    <tr>
      <th>37</th>
      <td>EOD~CSJ.11</td>
      <td>ST Corp Bond</td>
    </tr>
    <tr>
      <th>38</th>
      <td>EOD~FXF.11</td>
      <td>CHF</td>
    </tr>
  </tbody>
</table>
</div>
  </div>


## Calculate asset price correlations

#### Convert to log returns

Before calculating the correlation matrix, it is important to first normalise the dataset and convert the absolute asset prices into daily returns. In financial time-series it is common to make this transformation as investors are typically interested in returns on assets rather than their absolute prices. By normalising the data it allows us to compare the expected returns of two assets more easily.


<script src="https://gist.github.com/julian-west/c88bcf7639dbcc1cb1d147155e8d33e0.js"></script>


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
      <th>Bonds Global</th>
      <th>Commodities</th>
      <th>DOW</th>
      <th>Emerg Markets</th>
      <th>EAFE ETF</th>
      <th>Emerg Markets Bonds</th>
      <th>Pacifix ex Japan</th>
      <th>Germany</th>
      <th>Italy</th>
      <th>Japan</th>
      <th>...</th>
      <th>Europe</th>
      <th>Pacific</th>
      <th>VXX</th>
      <th>Materials</th>
      <th>Energy</th>
      <th>Finance</th>
      <th>Tech</th>
      <th>Utilities</th>
      <th>ST Corp Bond</th>
      <th>CHF</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-11-08</th>
      <td>-0.000733</td>
      <td>-0.001828</td>
      <td>0.000170</td>
      <td>0.004714</td>
      <td>0.003297</td>
      <td>-0.000436</td>
      <td>0.009904</td>
      <td>0.003321</td>
      <td>-0.004513</td>
      <td>0.006184</td>
      <td>...</td>
      <td>0.000516</td>
      <td>0.007864</td>
      <td>0.000298</td>
      <td>0.001023</td>
      <td>-0.004858</td>
      <td>-0.004940</td>
      <td>0.005483</td>
      <td>0.000718</td>
      <td>-0.000476</td>
      <td>-0.000317</td>
    </tr>
    <tr>
      <th>2017-11-07</th>
      <td>0.000366</td>
      <td>-0.006068</td>
      <td>0.000042</td>
      <td>-0.006423</td>
      <td>-0.003727</td>
      <td>-0.005306</td>
      <td>0.000424</td>
      <td>-0.008131</td>
      <td>-0.004173</td>
      <td>0.007911</td>
      <td>...</td>
      <td>-0.008559</td>
      <td>0.003052</td>
      <td>0.005384</td>
      <td>0.001024</td>
      <td>-0.001282</td>
      <td>-0.013928</td>
      <td>0.000471</td>
      <td>0.011929</td>
      <td>0.000095</td>
      <td>-0.002219</td>
    </tr>
    <tr>
      <th>2017-11-06</th>
      <td>0.000733</td>
      <td>0.018932</td>
      <td>0.000977</td>
      <td>0.011159</td>
      <td>0.001432</td>
      <td>-0.001387</td>
      <td>0.002333</td>
      <td>-0.001499</td>
      <td>0.000000</td>
      <td>-0.000169</td>
      <td>...</td>
      <td>0.001535</td>
      <td>0.001390</td>
      <td>-0.009552</td>
      <td>-0.004259</td>
      <td>0.022602</td>
      <td>-0.001121</td>
      <td>0.002203</td>
      <td>-0.003811</td>
      <td>0.000000</td>
      <td>0.003277</td>
    </tr>
    <tr>
      <th>2017-11-03</th>
      <td>0.000856</td>
      <td>0.006184</td>
      <td>0.000936</td>
      <td>-0.005166</td>
      <td>-0.001575</td>
      <td>-0.006305</td>
      <td>-0.004661</td>
      <td>-0.003289</td>
      <td>-0.006704</td>
      <td>0.002368</td>
      <td>...</td>
      <td>-0.001876</td>
      <td>-0.000139</td>
      <td>-0.001484</td>
      <td>-0.000510</td>
      <td>0.002916</td>
      <td>-0.004099</td>
      <td>0.007906</td>
      <td>0.003629</td>
      <td>-0.000381</td>
      <td>-0.002009</td>
    </tr>
    <tr>
      <th>2017-11-02</th>
      <td>0.000979</td>
      <td>0.006223</td>
      <td>0.003283</td>
      <td>0.001289</td>
      <td>0.003008</td>
      <td>0.002759</td>
      <td>0.005723</td>
      <td>0.004488</td>
      <td>0.009591</td>
      <td>0.001186</td>
      <td>...</td>
      <td>0.002559</td>
      <td>0.002089</td>
      <td>-0.011210</td>
      <td>-0.007279</td>
      <td>-0.002916</td>
      <td>0.009341</td>
      <td>0.000476</td>
      <td>0.003642</td>
      <td>0.000000</td>
      <td>0.004553</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 39 columns</p>
</div>
  </div>



#### Calculate correlations matrix

To calculate the pairwise correlations between assets we can simply use the inbuilt pandas _corr()_ function.


<script src="https://gist.github.com/julian-west/c8dc3e002e282e475d652feb48b900ff.js"></script>




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
      <th>Bonds Global</th>
      <th>Commodities</th>
      <th>DOW</th>
      <th>Emerg Markets</th>
      <th>EAFE ETF</th>
      <th>Emerg Markets Bonds</th>
      <th>Pacifix ex Japan</th>
      <th>Germany</th>
      <th>Italy</th>
      <th>Japan</th>
      <th>...</th>
      <th>Europe</th>
      <th>Pacific</th>
      <th>VXX</th>
      <th>Materials</th>
      <th>Energy</th>
      <th>Finance</th>
      <th>Tech</th>
      <th>Utilities</th>
      <th>ST Corp Bond</th>
      <th>CHF</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bonds Global</th>
      <td>1.000000</td>
      <td>-0.086234</td>
      <td>-0.279161</td>
      <td>-0.069623</td>
      <td>-0.177521</td>
      <td>0.296679</td>
      <td>-0.104739</td>
      <td>-0.188547</td>
      <td>-0.201014</td>
      <td>-0.159231</td>
      <td>...</td>
      <td>-0.181249</td>
      <td>-0.141707</td>
      <td>0.222886</td>
      <td>-0.220603</td>
      <td>-0.198536</td>
      <td>-0.420264</td>
      <td>-0.197672</td>
      <td>0.301774</td>
      <td>0.598105</td>
      <td>0.250142</td>
    </tr>
    <tr>
      <th>Commodities</th>
      <td>-0.086234</td>
      <td>1.000000</td>
      <td>0.305137</td>
      <td>0.428909</td>
      <td>0.369637</td>
      <td>0.313791</td>
      <td>0.400316</td>
      <td>0.283711</td>
      <td>0.331083</td>
      <td>0.250509</td>
      <td>...</td>
      <td>0.367019</td>
      <td>0.341089</td>
      <td>-0.224835</td>
      <td>0.429541</td>
      <td>0.677430</td>
      <td>0.257454</td>
      <td>0.225463</td>
      <td>0.089290</td>
      <td>0.028872</td>
      <td>0.042324</td>
    </tr>
    <tr>
      <th>DOW</th>
      <td>-0.279161</td>
      <td>0.305137</td>
      <td>1.000000</td>
      <td>0.719260</td>
      <td>0.793387</td>
      <td>0.343817</td>
      <td>0.688344</td>
      <td>0.722882</td>
      <td>0.666315</td>
      <td>0.691672</td>
      <td>...</td>
      <td>0.765251</td>
      <td>0.769604</td>
      <td>-0.797538</td>
      <td>0.817422</td>
      <td>0.655544</td>
      <td>0.874902</td>
      <td>0.849271</td>
      <td>0.395247</td>
      <td>-0.042967</td>
      <td>-0.172730</td>
    </tr>
    <tr>
      <th>Emerg Markets</th>
      <td>-0.069623</td>
      <td>0.428909</td>
      <td>0.719260</td>
      <td>1.000000</td>
      <td>0.796425</td>
      <td>0.580454</td>
      <td>0.815049</td>
      <td>0.697222</td>
      <td>0.660268</td>
      <td>0.638955</td>
      <td>...</td>
      <td>0.763111</td>
      <td>0.811155</td>
      <td>-0.668680</td>
      <td>0.674630</td>
      <td>0.607172</td>
      <td>0.606749</td>
      <td>0.686458</td>
      <td>0.365031</td>
      <td>0.123624</td>
      <td>-0.018461</td>
    </tr>
    <tr>
      <th>EAFE ETF</th>
      <td>-0.177521</td>
      <td>0.369637</td>
      <td>0.793387</td>
      <td>0.796425</td>
      <td>1.000000</td>
      <td>0.466883</td>
      <td>0.790699</td>
      <td>0.884385</td>
      <td>0.830052</td>
      <td>0.785198</td>
      <td>...</td>
      <td>0.949368</td>
      <td>0.872972</td>
      <td>-0.721559</td>
      <td>0.720030</td>
      <td>0.610561</td>
      <td>0.711714</td>
      <td>0.725415</td>
      <td>0.352270</td>
      <td>0.050121</td>
      <td>0.021760</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 39 columns</p>
</div>
  </div>



### Correlations Heatmap

The conventional way to visualise correlations is via a heatmap. Before developing the network visualisation, we will quickly create a heatmap of the correlation matrix to check the output of the correlation calculations and to gain a high level insight into some of the relationships present in the data.

Seaborn has a very useful function called _clustermap_ which visualises the matrix as a heatmap but also clusters the ETFs so that ETFs which behave similarly are next to each other. Clustered heatmaps can be a useful way of visualising correlations between attributes in a dataset, especially if the data is highly dimensional as it automatically reorders attributes which are similar to each other into clusters. This makes the heatmap more structured and readable so it is easier to identify relationships between ETFs and which asset classes behave similarly.


<script src="https://gist.github.com/julian-west/c59beefcf11765f21128520591cf781b.js"></script>


  <div markdown="0">
  <h3>Clustered Heatmap: Correlations between asset price returns</h3>
  </div>



![png](../assets/visualising_stock_correlations_files/visualising_stock_correlations_14_1.png)


The heatmap is colour coded using a divergent colourscale where strong positive correlations (correlation = 1) are dark green, uncorrelated assets are yellow (correlation = 0) and negatively correlated assets are red (correlation = -1).

The clustered heatmap visualisation already gives a good picture of the data and tells an interesting story:
1. Broadly speaking, there are two major clusters of assets. These appear to be separated into equities and "non-equity" assets (e.g. bonds, currencies and precious metals). The heatmap shows that these two categories are generally negatively or non-correlated with each other which is expected as 'safe haven' assets such as bonds, gold and currencies like the Japanese Yen tend to move in opposite directions to equities which are seen as a riskier asset class.
2. ETFs tracking Geographic regions which are close to each other are highly correlated with each other. For example, UK, Europe, Germany, Italy and France ETFs are highly correlated with each other, so are Japan, Pacific ex Japan and China ETFs
3. The VXX ETF is strongly negatively correlated with equities.  


## Network Visualisations

Heatmaps are useful, however, they can only convey one dimension of information (the magnitude of the correlation between two assets). As an investor wanting to make a decision on which asset classes to invest in, a heatmap still does not help answer important questions such as what the annualized returns and volatility of an asset class is.

We can use network graphs  to investigate the initial findings from the heatmap further and visualise them in a more accessible way which encodes more information.

### Networkx

[Networkx](https://networkx.github.io/documentation/stable/) is one of the most popular and useful Python libraries for analysing small/medium size networks.

In order to analyse the the correlations matrix as a network we first need to convert the correlations between assets to an edge list. This is a list containing information for each connection between each asset ETF in our data. This format requires the 'source' node (ETF), the 'target' node and the 'weight' (correlation) of the link between the two.

__Create edge list__


<script src="https://gist.github.com/julian-west/b66319a742d8b8a7c874c19d6ac49050.js"></script>



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
      <th>asset_1</th>
      <th>asset_2</th>
      <th>correlation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Bonds Global</td>
      <td>Commodities</td>
      <td>-0.086234</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bonds Global</td>
      <td>DOW</td>
      <td>-0.279161</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bonds Global</td>
      <td>Emerg Markets</td>
      <td>-0.069623</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bonds Global</td>
      <td>EAFE ETF</td>
      <td>-0.177521</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Bonds Global</td>
      <td>Emerg Markets Bonds</td>
      <td>0.296679</td>
    </tr>
  </tbody>
</table>
</div>
  </div>



__Create graph from edge list__

Now that we have an edge list we need to feed that into the networkx library to create a graph. Note that this network is undirected as the correlation between assets is the same in both directions.


<script src="https://gist.github.com/julian-west/2b3800b324eeeb2d339f097f6dd7f759.js"></script>

  ```
  Name:
Type: Graph
Number of nodes: 39
Number of edges: 741
Average degree:  38.0000

  ```

__Visualise the network__

To visualise the graph we have just created, we can use a number of 'out-of-the-box' layouts which can be drawn using networkx, for example:

- circular_layout - Position nodes on a circle.
- random_layout - Position nodes uniformly at random in the unit square.
- spectral_layout - Position nodes using the eigenvectors of the graph Laplacian.  
- spring_layout - Position nodes using Fruchterman-Reingold force-directed algorithm.   


<script src="https://gist.github.com/julian-west/9e1fce7882058e2200154723989886e9.js"></script>


![png](../assets/visualising_stock_correlations_files/visualising_stock_correlations_21_0.png)


Whilst these visualisations may look pretty, they are not very useful in their current form as all nodes have the same number of edges (links), and each edge looks the same so no useful information about the correlations between ETFs can be gained.

__Improved network visualisation__

The network visualisation can be improved in a number of ways by thinking about the sort of information which we are looking to uncover from this analysis. It is important to think about what questions we wish to answer from the data and then design the visualisation to gain the most insight for the question in hand. For the purposes of enhancing the visualisation, We will assume that the audience for this visualisation will be investors wanting to assess risk in their portfolio. Investors would want to identify which assets are correlated and uncorrelated with each other in order to assess the unsystematic risk in their portfolio. Therefore, from this visualisation the user would want to quickly understand:
- which assets show strong/meaningful correlations (i.e. >0.5) with each other
- are these correlations positive or negative
- which are the most/least 'connected' assets. (i.e. which assets share the most/least strong correlations with other assets in the dataset)
- which groups of assets behave similarly (i.e. which assets are correlated with the same type of other assets)

With this information, an investor could identify if they held a number of assets which behave the same (increased risk) and identify assets which show very few correlations with assets currently held in the portfolio and investigate these as a potential opportunity for diversification.

For a first iteration to improve the network visualisation we can take the circular layout graph from above and make the following changes:
- reduce the number of connections between nodes by adding a threshold value for the strength of correlation
- introduce colour to signify positive or negative correlations
- scale the edge thickness to indicate the magnitude of correlation
- scale the size of nodes to indicate which assets have the greatest number of strong correlations with the rest of the assets in the dataset

##### Remove edges below a threshold


<script src="https://gist.github.com/julian-west/dbc6b99d5fc5ab2cc9a77780450930f8.js"></script>

  ```
  530 edges removed

  ```

##### Create colour, edge thickness and node size features

<script src="https://gist.github.com/julian-west/29299749dd7591563e67ddef962ac355.js"></script>

##### Draw improved graph

<script src="https://gist.github.com/julian-west/69667e33e9e8ad2ebb881902539084fc.js"></script>


![png](../assets/visualising_stock_correlations_files/visualising_stock_correlations_28_0.png)


The network visualisation has been improved in four main ways:
- less cluttered: we have removed edges with weak correlations and kept only the edges with significant (actionable) correlations
- identify type of correlation: simple intuitive colour scheme to show positive (green) or negative (red) correlations
- identify strength of correlation: we now are able to assess the relative strength of correlations between nodes, with prominence given to the correlations with the greatest magnitude
- identify the most connected nodes: the size of the nodes has been adjusted to emphasise which nodes have the greatest number of strong correlations with other nodes in the network

Looking at this network we could now quickly identify which assets are highly correlated and therefore may pose increased risk on the portfolio. The viewer could also identify and investigate further the assets with low degree of connectivity (smaller node size) as these are weakly correlated with the other assets in the sample and may provide an opportunity to diversify the portfolio.

The circular layout, however, does not group the nodes in a meaningful order, it just orders the nodes in the order in which they were created, therefore it is difficult to gain insight as to which assets are most similar to each other in terms of correlations to other nodes. We can improve this with a spring based layout using the 'Fruchterman-Reingold' algorithm [2] which sets the positions of the nodes using a cost function which minimises the distances between strongly correlated nodes. This algorithm will therefore cluster the nodes which are strongly correlated with each other allowing the viewer to quickly identify groups of assets with similar properties.


<script src="https://gist.github.com/julian-west/09ef058bc60da14d838fa8c49ed5c0fc.js"></script>


![png](../assets/visualising_stock_correlations_files/visualising_stock_correlations_30_0.png)


The Fruchterman Reingold layout has successfully grouped the assets into clusters of strongly correlated assets. As seen before in the heatmap visualisation, there are distinct clusters of assets which behave similarly to each other. There is a cluster containing bond ETFs, a cluster of precious metal ETFs (silver, gold, goldminers), a cluster of currencies (CHF, USD, Euro) and a large cluster for equities.

However, the main cluster of equities ETFs is still very cluttered as the nodes are very tightly packed and the node sizes and labels are overlapping making it difficult to make out.

As the layout now positions the nodes which are strongly correlated in space, it is no longer necessary to keep every single edge as it is implied that assets closer to each other in space are more strongly correlated. We can also convert the node sizes back to a consistent (smaller) size as the degree of each node is now meaningless.

##### Minimum spanning tree

It is common in financial networks to use a minimum spanning tree [3,4,5] to visualise networks. A minimum spanning tree reduces the edges down to a subset of edges which connects all the nodes together, without any cycles and with the minimum possible sum of edge weights (correlation value in this case). This essentially provides a skeleton of the graph, minimising the number of edges and reducing the clutter in the network graph.

[Kruskal's algorithm](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm) is used to calculate the minimum spanning tree and is fairly intuitive. However, Networkx has an inbuilt function which calculates the minimum spanning tree for us.


<script src="https://gist.github.com/julian-west/3ce28ae7db651b2bb8fc9cd33b4f8d4d.js"></script>

![png](../assets/visualising_stock_correlations_files/visualising_stock_correlations_33_0.png)


The improved graph has made the clusters of nodes more readable by reducing the node size and reducing the number of edges in the graph. However, reducing the clutter was at the expense of conveying some information about the nodes such as nodes with the most strong correlations and their relative strengths.

### Interactive visualisation

Finally, I will explore the use of interactive plotting libraries to enhance the user experience and alleviate some of the missing information issues with the use of informative tooltips.

Currently the network is very 'static' and does not convey any information other than spacial relationships between strongly correlated assets. Although useful, this logically leads the viewer to ask more questions about the properties of each individual node in a cluster such as their historical returns and the nodes with which it is least correlated to.

One way to include such information about each node is to use an interactive graph with informative tooltips. This can be achieved using the Plotly[6] python library which offers a python api to create interactive javascript graphs built in d3.js. The following code uses the Plotly api to create a graph with the following features:
- interactivity, such as zoom, to focus on clusters of nodes
- tooltips which provides useful information about the node when the user hovers over it
- node sizes proportional to the annualised returns of each asset
- node colours to show asset positive or negative return over the dataset timeframe

#### Plotly

The Python code required for Ploty graphs can be a little bit more involved than out of the box matplotlib or seaborn charts but allows for much more customisation.

The main benefit of using Plotly in this example is the use of tooltips. In these tooltips we can store lots of additional information about each ETF which we can access by hovering over the respective node in the graph.

There are obviously many different bits of information of use which we could incorporate into the tooltip. For this example I will add:
- annualised returns of the asset class
- annualised volatility of the asset class
- the top and bottom 3 assets which the asset of interest is most/least strongly correlated with

##### Functions to get node information to populate tooltips

The following functions calculate the above quantities. It should be noted that Plotly tooltips are formatted using html. Therefore the input to the tooltip should be a string with the relevant html tags for any formatting that is required.


<script src="https://gist.github.com/julian-west/7d7ab41b49766ec2eadbee07f179c885.js"></script>

##### Function to generate coordinates

Plotly does not have an 'out-of-the-box' network graph chart, therefore we need to 'imitate' the network layout by plotting the data as a scatter plot which plots the graph nodes, and plot a 'line' chart on top which draws the lines which connect each point. To achieve this we need a function which converts the Fruchterman Reingold coordinates calculated using networtx into an x and y series to create the scatter plot. We also need to store the x and y coordinates of the start and end of each edge which will be used to draw the 'line' chart.

The function calculates the x and y coordinates for the scatter plot (Xnodes and Ynodes) and the coordinates of the starting and ending positions of the lines connecting nodes (Xedges, Yedges):

<script src="https://gist.github.com/julian-west/c2fd8efaae3f31563701010b631fbe3c.js"></script>

__Plotly Graph Code__

Now we can finally plot the network as an interactive Plotly visualisation.

First, we need to calculate all the quantities and concatenate them into a html string to be used as an input for the tooltip. Then we need to calculate the coordinates for the scatter and line plots and finally define the node color and size which depend on the direction and size of the annualised returns.

<script src="https://gist.github.com/julian-west/2ea09ab1a148481d124cf3a60d3c823b.js"></script>

To plot the network we define the scatter plot (tracer) and line plot (tracer_marker) series and define some cosmetic parameters which dictate the graph layout.

<script src="https://gist.github.com/julian-west/ca80206a4d1010b40a3cbf0382279ec1.js"></script>

  <div markdown="0">
  <p>Node sizes are proportional to the size of annualised returns.<br>
                Node colours signify positive or negative returns since beginning of the timeframe.</p>
  </div>


<a name="plotly graph"></a>
  <div markdown="0">


  <div class="input_area" markdown="1">


  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script><div id="62c6be38-feea-4e42-91b5-ad5cb5374f66" class="plotly-graph-div" style="height:800px; width:800px;"></div><script type="text/javascript">window.PLOTLYENV=window.PLOTLYENV || {};if (document.getElementById("62c6be38-feea-4e42-91b5-ad5cb5374f66")) {Plotly.newPlot('62c6be38-feea-4e42-91b5-ad5cb5374f66',[{"hoverinfo": "none", "line": {"color": "#DCDCDC", "width": 1}, "mode": "lines", "showlegend": false, "type": "scatter", "x": [0.08881567905654729, 0.14219446175739464, null, 0.08881567905654729, 0.14547910832673452, null, 0.08881567905654729, 0.0678910578142049, null, 0.08881567905654729, -0.025413207423628365, null, 0.08881567905654729, 0.0023048478936011627, null, 0.08881567905654729, 0.18200010974322967, null, -0.5141088853318235, -0.6248901845248148, null, -0.5141088853318235, -0.6507093149355977, null, -0.5141088853318235, -0.30962827455714015, null, -0.06560430455472269, 0.004967435573741214, null, -0.06560430455472269, -0.15897587208074193, null, -0.06560430455472269, -0.152091244431824, null, -0.06560430455472269, -0.023987185580284332, null, -0.06560430455472269, 0.061433212139347666, null, -0.06560430455472269, -0.19565741309762238, null, -0.06560430455472269, 0.06314606935460153, null, -0.06560430455472269, 0.07678330471253572, null, -0.06560430455472269, -0.16103248280595145, null, -0.06560430455472269, 0.1475946945725378, null, -0.06560430455472269, -0.0878407272587489, null, -0.06560430455472269, -0.015905324913862127, null, -0.06560430455472269, 0.04199124409275927, null, -0.06560430455472269, -0.21791128642209218, null, -0.06560430455472269, -0.08224213661789308, null, -0.06560430455472269, -0.22095314098935834, null, -0.06560430455472269, -0.30962827455714015, null, -0.06560430455472269, -0.11695572143942899, null, -0.06560430455472269, 0.04245841648186777, null, 0.004967435573741214, 0.04872444309148883, null, 0.07678330471253572, 0.20169301883961577, null, 0.20169301883961577, 0.3797501675336845, null, 0.20169301883961577, 0.17454679203554743, null, 0.3797501675336845, 0.5152890493613155, null, 0.14219446175739464, 0.23911154915758948, null, 0.14219446175739464, 0.17454679203554743, null, 0.3109037670374345, 0.23911154915758948, null, 0.3109037670374345, 0.39735120906100363, null, 0.1475946945725378, 0.2894770693287549, null], "y": [-0.860964223278597, -0.6354115041785166, null, -0.860964223278597, -0.988259749958403, null, -0.860964223278597, -1.0, null, -0.860964223278597, -0.8847811916559735, null, -0.860964223278597, -0.951994486104371, null, -0.860964223278597, -0.8987502639637319, null, 0.1624566102813492, 0.0836537523729241, null, 0.1624566102813492, 0.16572540268155744, null, 0.1624566102813492, 0.2478674847785123, null, 0.3585600158351537, 0.5759735854441056, null, 0.3585600158351537, 0.24455591399239043, null, 0.3585600158351537, 0.5159426114307397, null, 0.3585600158351537, 0.48286572177625564, null, 0.3585600158351537, 0.27570532897229894, null, 0.3585600158351537, 0.4640234699389884, null, 0.3585600158351537, 0.398566067636373, null, 0.3585600158351537, 0.09122092014699142, null, 0.3585600158351537, 0.3171429281756685, null, 0.3585600158351537, 0.4171763763961235, null, 0.3585600158351537, 0.22279938487211332, null, 0.3585600158351537, 0.24054768251713737, null, 0.3585600158351537, 0.46766845149729536, null, 0.3585600158351537, 0.40928495836498174, null, 0.3585600158351537, 0.5245744086157088, null, 0.3585600158351537, 0.343764091308348, null, 0.3585600158351537, 0.2478674847785123, null, 0.3585600158351537, 0.4491949434199497, null, 0.3585600158351537, 0.333406425123105, null, 0.5759735854441056, 0.7241879800922412, null, 0.09122092014699142, -0.15480587445020788, null, -0.15480587445020788, -0.17756878803351836, null, -0.15480587445020788, -0.3995745496548265, null, -0.17756878803351836, -0.19260079075727007, null, -0.6354115041785166, -0.6573426083823192, null, -0.6354115041785166, -0.3995745496548265, null, -0.600372270188565, -0.6573426083823192, null, -0.600372270188565, -0.5830862371070473, null, 0.4171763763961235, 0.46864802204303363, null]}, {"hoverinfo": "text", "hovertext": ["<b>Bonds Global</b><br>Annualized Returns: 2.8%<br>Annualized Volatility: 3.2%<br><br>Strongest correlations with: <br>10yr treasuries: 0.94<br>20+ Treasuries: 0.90<br>Corp Bond: 0.89<br><br>Weakest correlations with: <br>GBP: 0.03<br>High yield Bond: -0.06<br>Emerg Markets: -0.07<br>", "<b>Commodities</b><br>Annualized Returns: -10.9%<br>Annualized Volatility: 15.1%<br><br>Strongest correlations with: <br>Oil: 0.88<br>Energy: 0.68<br>China Large Cap: 0.53<br><br>Weakest correlations with: <br>ST Corp Bond: 0.03<br>Yen: -0.03<br>CHF: 0.04<br>", "<b>DOW</b><br>Annualized Returns: 12.6%<br>Annualized Volatility: 11.8%<br><br>Strongest correlations with: <br>SNP 500 ETF: 0.97<br>Finance: 0.87<br>Tech: 0.85<br><br>Weakest correlations with: <br>Silver: 0.03<br>Gold Miners: 0.04<br>ST Corp Bond: -0.04<br>", "<b>Emerg Markets</b><br>Annualized Returns: 4.3%<br>Annualized Volatility: 18.5%<br><br>Strongest correlations with: <br>China: 0.84<br>Pacifix ex Japan: 0.82<br>Pacific : 0.81<br><br>Weakest correlations with: <br>Bonds II: -0.00<br>Gold: -0.01<br>CHF: -0.02<br>", "<b>EAFE ETF</b><br>Annualized Returns: 4.4%<br>Annualized Volatility: 14.7%<br><br>Strongest correlations with: <br>Europe: 0.95<br>France: 0.91<br>UK: 0.91<br><br>Weakest correlations with: <br>CHF: 0.02<br>ST Corp Bond: 0.05<br>Corp Bond: -0.06<br>", "<b>Emerg Markets Bonds</b><br>Annualized Returns: 5.6%<br>Annualized Volatility: 6.4%<br><br>Strongest correlations with: <br>Emerg Markets: 0.58<br>High yield Bond: 0.58<br>Pacifix ex Japan: 0.48<br><br>Weakest correlations with: <br>Yen: 0.07<br>CHF: 0.07<br>Euro: 0.12<br>", "<b>Pacifix ex Japan</b><br>Annualized Returns: 3.1%<br>Annualized Volatility: 16.2%<br><br>Strongest correlations with: <br>Pacific : 0.84<br>Emerg Markets: 0.82<br>EAFE ETF: 0.79<br><br>Weakest correlations with: <br>Corp Bond: -0.01<br>Gold: -0.01<br>CHF: -0.01<br>", "<b>Germany</b><br>Annualized Returns: 5.8%<br>Annualized Volatility: 17.9%<br><br>Strongest correlations with: <br>Europe: 0.93<br>France: 0.93<br>EAFE ETF: 0.88<br><br>Weakest correlations with: <br>CHF: 0.02<br>ST Corp Bond: 0.02<br>Gold Miners: 0.06<br>", "<b>Italy</b><br>Annualized Returns: 2.6%<br>Annualized Volatility: 23.8%<br><br>Strongest correlations with: <br>Europe: 0.88<br>France: 0.88<br>Germany: 0.84<br><br>Weakest correlations with: <br>ST Corp Bond: -0.01<br>CHF: 0.01<br>Gold Miners: 0.07<br>", "<b>Japan</b><br>Annualized Returns: 7.5%<br>Annualized Volatility: 15.7%<br><br>Strongest correlations with: <br>Pacific : 0.93<br>EAFE ETF: 0.79<br>SNP 500 ETF: 0.71<br><br>Weakest correlations with: <br>Gold Miners: 0.03<br>ST Corp Bond: 0.04<br>Silver: 0.05<br>", "<b>France</b><br>Annualized Returns: 5.4%<br>Annualized Volatility: 17.8%<br><br>Strongest correlations with: <br>Europe: 0.96<br>Germany: 0.93<br>EAFE ETF: 0.91<br><br>Weakest correlations with: <br>CHF: 0.05<br>ST Corp Bond: 0.05<br>Corp Bond: -0.07<br>", "<b>UK</b><br>Annualized Returns: 0.6%<br>Annualized Volatility: 16.9%<br><br>Strongest correlations with: <br>Europe: 0.94<br>EAFE ETF: 0.91<br>France: 0.85<br><br>Weakest correlations with: <br>CHF: 0.03<br>ST Corp Bond: 0.03<br>Corp Bond: -0.07<br>", "<b>GBP</b><br>Annualized Returns: -5.2%<br>Annualized Volatility: 9.6%<br><br>Strongest correlations with: <br>USD: -0.61<br>UK: 0.55<br>Euro: 0.51<br><br>Weakest correlations with: <br>10yr treasuries: -0.01<br>Bonds Global: 0.03<br>Corp Bond: 0.06<br>", "<b>China Large Cap</b><br>Annualized Returns: -5.1%<br>Annualized Volatility: 8.0%<br><br>Strongest correlations with: <br>Commodities: 0.53<br>Oil: 0.51<br>USD: -0.47<br><br>Weakest correlations with: <br>10yr treasuries: 0.01<br>20+ Treasuries: -0.08<br>Bonds Global: 0.08<br>", "<b>Euro</b><br>Annualized Returns: -4.4%<br>Annualized Volatility: 8.8%<br><br>Strongest correlations with: <br>USD: -0.96<br>CHF: 0.53<br>GBP: 0.51<br><br>Weakest correlations with: <br>US real estate: -0.01<br>Energy: -0.01<br>Materials: -0.02<br>", "<b>China</b><br>Annualized Returns: 7.5%<br>Annualized Volatility: 22.9%<br><br>Strongest correlations with: <br>Emerg Markets: 0.84<br>Pacifix ex Japan: 0.73<br>Pacific : 0.72<br><br>Weakest correlations with: <br>Corp Bond: -0.06<br>ST Corp Bond: 0.06<br>USD: 0.07<br>", "<b>Yen</b><br>Annualized Returns: -4.0%<br>Annualized Volatility: 9.6%<br><br>Strongest correlations with: <br>10yr treasuries: 0.56<br>USD: -0.54<br>Finance: -0.54<br><br>Weakest correlations with: <br>Commodities: -0.03<br>Utilities: 0.04<br>Emerg Markets Bonds: 0.07<br>", "<b>Gold Miners</b><br>Annualized Returns: -0.6%<br>Annualized Volatility: 41.6%<br><br>Strongest correlations with: <br>Gold: 0.79<br>Silver: 0.70<br>China Large Cap: 0.36<br><br>Weakest correlations with: <br>VXX: -0.03<br>Japan: 0.03<br>DOW: 0.04<br>", "<b>Gold</b><br>Annualized Returns: -1.1%<br>Annualized Volatility: 14.2%<br><br>Strongest correlations with: <br>Gold Miners: 0.79<br>Silver: 0.79<br>Yen: 0.52<br><br>Weakest correlations with: <br>Pacifix ex Japan: -0.01<br>Emerg Markets: -0.01<br>Energy: -0.02<br>", "<b>10yr treasuries</b><br>Annualized Returns: 2.9%<br>Annualized Volatility: 5.4%<br><br>Strongest correlations with: <br>Bonds Global: 0.94<br>20+ Treasuries: 0.93<br>Corp Bond: 0.86<br><br>Weakest correlations with: <br>China Large Cap: 0.01<br>GBP: -0.01<br>US real estate: 0.09<br>", "<b>US real estate</b><br>Annualized Returns: 9.3%<br>Annualized Volatility: 13.4%<br><br>Strongest correlations with: <br>SNP 500 ETF: 0.63<br>Utilities: 0.62<br>DOW: 0.59<br><br>Weakest correlations with: <br>Euro: -0.01<br>USD: -0.02<br>CHF: -0.04<br>", "<b>High yield Bond</b><br>Annualized Returns: 3.5%<br>Annualized Volatility: 6.2%<br><br>Strongest correlations with: <br>SNP 500 ETF: 0.67<br>Materials: 0.66<br>Emerg Markets: 0.64<br><br>Weakest correlations with: <br>Bonds II: 0.01<br>USD: -0.01<br>Euro: -0.02<br>", "<b>Corp Bond</b><br>Annualized Returns: 4.7%<br>Annualized Volatility: 5.0%<br><br>Strongest correlations with: <br>Bonds Global: 0.89<br>10yr treasuries: 0.86<br>20+ Treasuries: 0.85<br><br>Weakest correlations with: <br>Pacifix ex Japan: -0.01<br>Emerg Markets: 0.02<br>Pacific : -0.03<br>", "<b>Silver</b><br>Annualized Returns: -6.8%<br>Annualized Volatility: 22.2%<br><br>Strongest correlations with: <br>Gold: 0.79<br>Gold Miners: 0.70<br>Commodities: 0.35<br><br>Weakest correlations with: <br>VXX: -0.02<br>DOW: 0.03<br>Tech: 0.03<br>", "<b>SNP 500 ETF</b><br>Annualized Returns: 11.6%<br>Annualized Volatility: 12.1%<br><br>Strongest correlations with: <br>DOW: 0.97<br>Tech: 0.90<br>Finance: 0.87<br><br>Weakest correlations with: <br>ST Corp Bond: -0.02<br>Silver: 0.05<br>Gold Miners: 0.08<br>", "<b>Bonds II</b><br>Annualized Returns: 1.7%<br>Annualized Volatility: 4.7%<br><br>Strongest correlations with: <br>10yr treasuries: 0.84<br>Bonds Global: 0.83<br>20+ Treasuries: 0.79<br><br>Weakest correlations with: <br>Emerg Markets: -0.00<br>Pacifix ex Japan: -0.01<br>High yield Bond: 0.01<br>", "<b>20+ Treasuries</b><br>Annualized Returns: 7.0%<br>Annualized Volatility: 12.5%<br><br>Strongest correlations with: <br>10yr treasuries: 0.93<br>Bonds Global: 0.90<br>Corp Bond: 0.85<br><br>Weakest correlations with: <br>US real estate: 0.07<br>GBP: -0.07<br>China Large Cap: -0.08<br>", "<b>Oil</b><br>Annualized Returns: -27.3%<br>Annualized Volatility: 34.1%<br><br>Strongest correlations with: <br>Commodities: 0.88<br>Energy: 0.71<br>China Large Cap: 0.51<br><br>Weakest correlations with: <br>CHF: 0.00<br>ST Corp Bond: 0.01<br>Bonds II: 0.03<br>", "<b>USD</b><br>Annualized Returns: 3.1%<br>Annualized Volatility: 7.4%<br><br>Strongest correlations with: <br>Euro: -0.96<br>GBP: -0.61<br>CHF: -0.58<br><br>Weakest correlations with: <br>Materials: -0.00<br>Pacific : -0.00<br>High yield Bond: -0.01<br>", "<b>Europe</b><br>Annualized Returns: 4.1%<br>Annualized Volatility: 16.2%<br><br>Strongest correlations with: <br>France: 0.96<br>EAFE ETF: 0.95<br>UK: 0.94<br><br>Weakest correlations with: <br>ST Corp Bond: 0.05<br>CHF: 0.07<br>Corp Bond: -0.07<br>", "<b>Pacific </b><br>Annualized Returns: 6.8%<br>Annualized Volatility: 14.1%<br><br>Strongest correlations with: <br>Japan: 0.93<br>EAFE ETF: 0.87<br>Pacifix ex Japan: 0.84<br><br>Weakest correlations with: <br>USD: -0.00<br>Euro: -0.02<br>Corp Bond: -0.03<br>", "<b>VXX</b><br>Annualized Returns: -79.6%<br>Annualized Volatility: 60.3%<br><br>Strongest correlations with: <br>SNP 500 ETF: -0.84<br>DOW: -0.80<br>Tech: -0.75<br><br>Weakest correlations with: <br>Silver: -0.02<br>Gold Miners: -0.03<br>ST Corp Bond: 0.05<br>", "<b>Materials</b><br>Annualized Returns: 9.4%<br>Annualized Volatility: 15.6%<br><br>Strongest correlations with: <br>SNP 500 ETF: 0.83<br>DOW: 0.82<br>Finance: 0.74<br><br>Weakest correlations with: <br>ST Corp Bond: 0.00<br>USD: -0.00<br>Euro: -0.02<br>", "<b>Energy</b><br>Annualized Returns: -2.6%<br>Annualized Volatility: 20.8%<br><br>Strongest correlations with: <br>Materials: 0.73<br>Oil: 0.71<br>SNP 500 ETF: 0.69<br><br>Weakest correlations with: <br>ST Corp Bond: -0.00<br>Euro: -0.01<br>Gold: -0.02<br>", "<b>Finance</b><br>Annualized Returns: 13.0%<br>Annualized Volatility: 15.7%<br><br>Strongest correlations with: <br>DOW: 0.87<br>SNP 500 ETF: 0.87<br>Materials: 0.74<br><br>Weakest correlations with: <br>Silver: -0.05<br>Gold Miners: -0.07<br>GBP: 0.12<br>", "<b>Tech</b><br>Annualized Returns: 17.8%<br>Annualized Volatility: 14.3%<br><br>Strongest correlations with: <br>SNP 500 ETF: 0.90<br>DOW: 0.85<br>VXX: -0.75<br><br>Weakest correlations with: <br>ST Corp Bond: 0.01<br>Silver: 0.03<br>Gold Miners: 0.06<br>", "<b>Utilities</b><br>Annualized Returns: 12.3%<br>Annualized Volatility: 14.3%<br><br>Strongest correlations with: <br>US real estate: 0.62<br>SNP 500 ETF: 0.42<br>DOW: 0.40<br><br>Weakest correlations with: <br>Yen: 0.04<br>Oil: 0.07<br>CHF: 0.07<br>", "<b>ST Corp Bond</b><br>Annualized Returns: 1.2%<br>Annualized Volatility: 0.9%<br><br>Strongest correlations with: <br>Bonds Global: 0.60<br>Corp Bond: 0.58<br>10yr treasuries: 0.58<br><br>Weakest correlations with: <br>Materials: 0.00<br>Energy: -0.00<br>Tech: 0.01<br>", "<b>CHF</b><br>Annualized Returns: -3.2%<br>Annualized Volatility: 11.8%<br><br>Strongest correlations with: <br>USD: -0.58<br>Euro: 0.53<br>Yen: 0.39<br><br>Weakest correlations with: <br>Oil: 0.00<br>Pacifix ex Japan: -0.01<br>Italy: 0.01<br>"], "marker": {"color": ["#9eccb7", "#ffa09b", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#ffa09b", "#ffa09b", "#ffa09b", "#9eccb7", "#ffa09b", "#ffa09b", "#ffa09b", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#ffa09b", "#9eccb7", "#9eccb7", "#9eccb7", "#ffa09b", "#9eccb7", "#9eccb7", "#9eccb7", "#ffa09b", "#9eccb7", "#ffa09b", "#9eccb7", "#9eccb7", "#9eccb7", "#9eccb7", "#ffa09b"], "line": {"width": 1}, "size": [8.409962668180937, 16.524876267989818, 17.7498353842169, 10.403416190734509, 10.53709598744569, 11.788905944056836, 8.74128497140116, 12.018545507968037, 8.086765277867375, 13.672562505704223, 11.597668231034088, 4.000588539017547, 11.373755562808794, 11.277544145227461, 10.478913401298094, 13.724484396832173, 9.974741121453453, 3.781399550268956, 5.16239106325131, 8.552093624728395, 15.226449669612531, 9.333975515026662, 10.816214686027195, 13.012444619200147, 17.038389976013647, 6.562382254765895, 13.190086272365527, 26.116645047013265, 8.841394865569645, 10.163491593442695, 12.997150911568347, 44.62053536030952, 15.307855776949758, 8.019867967649972, 17.99398503323159, 21.086276557424682, 17.527129395409723, 5.401409819263998, 8.931243120025815]}, "mode": "markers+text", "showlegend": false, "text": ["Bonds Global", "Commodities", "DOW", "Emerg Markets", "EAFE ETF", "Emerg Markets Bonds", "Pacifix ex Japan", "Germany", "Italy", "Japan", "France", "UK", "GBP", "China Large Cap", "Euro", "China", "Yen", "Gold Miners", "Gold", "10yr treasuries", "US real estate", "High yield Bond", "Corp Bond", "Silver", "SNP 500 ETF", "Bonds II", "20+ Treasuries", "Oil", "USD", "Europe", "Pacific ", "VXX", "Materials", "Energy", "Finance", "Tech", "Utilities", "ST Corp Bond", "CHF"], "textfont": {"size": 7}, "textposition": "top center", "type": "scatter", "x": [0.08881567905654729, -0.5141088853318235, -0.06560430455472269, 0.004967435573741214, -0.15897587208074193, 0.04872444309148883, -0.152091244431824, -0.023987185580284332, 0.061433212139347666, -0.19565741309762238, 0.06314606935460153, 0.07678330471253572, 0.20169301883961577, -0.6248901845248148, 0.3797501675336845, -0.16103248280595145, 0.14219446175739464, 0.3109037670374345, 0.23911154915758948, 0.14547910832673452, 0.1475946945725378, -0.0878407272587489, 0.0678910578142049, 0.39735120906100363, -0.015905324913862127, -0.025413207423628365, 0.0023048478936011627, -0.6507093149355977, 0.17454679203554743, 0.04199124409275927, -0.21791128642209218, -0.08224213661789308, -0.22095314098935834, -0.30962827455714015, -0.11695572143942899, 0.04245841648186777, 0.2894770693287549, 0.18200010974322967, 0.5152890493613155], "y": [-0.860964223278597, 0.1624566102813492, 0.3585600158351537, 0.5759735854441056, 0.24455591399239043, 0.7241879800922412, 0.5159426114307397, 0.48286572177625564, 0.27570532897229894, 0.4640234699389884, 0.398566067636373, 0.09122092014699142, -0.15480587445020788, 0.0836537523729241, -0.17756878803351836, 0.3171429281756685, -0.6354115041785166, -0.600372270188565, -0.6573426083823192, -0.988259749958403, 0.4171763763961235, 0.22279938487211332, -1.0, -0.5830862371070473, 0.24054768251713737, -0.8847811916559735, -0.951994486104371, 0.16572540268155744, -0.3995745496548265, 0.46766845149729536, 0.40928495836498174, 0.5245744086157088, 0.343764091308348, 0.2478674847785123, 0.4491949434199497, 0.333406425123105, 0.46864802204303363, -0.8987502639637319, -0.19260079075727007]}],{"autosize": false, "height": 800, "hovermode": "closest", "plot_bgcolor": "#fff", "showlegend": false, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Plotly - interactive minimum spanning tree"}, "width": 800, "xaxis": {"showgrid": false, "showline": false, "showticklabels": false, "ticks": "", "title": {"font": {"size": 20}, "text": ""}, "zeroline": false}, "yaxis": {"showgrid": false, "showline": false, "showticklabels": false, "ticks": "", "title": {"font": {"size": 20}, "text": ""}, "zeroline": false}},{"responsive": true})};</script>

With this final network layout, we have summarised the most important information about the relationships between assets in the dataset. Users can quickly identify clusters of assets which are strongly correlated with each other, gauge the performance of the asset over the timeframe by looking at the node sizes and colours and can get more detailed information about each node by hovering over it.

### Conclusion

In this post we have shown how to visualise a network of asset price correlations using the networkx python library. The 'out of the box' networkx visualisations were iteratively improved to help the audience identify assets which behave similarly to other assets in the dataset. This was achieved by reducing redundant information in the network and using intuitive colour schemes, edge thickness and node thickness to convey information in a qualitative way. The functionality of the visualisation was further improved by the use of Plotly, an interactive graphing library, which allowed us to use tooltips to store more information about each node in the network without cluttering the visualisation.


### Future Work

Further work could look at the rolling asset correlations over a shorter timeframe and see how these may have changed over time and how this then affects the network layout and clusters. This could help identify outliers or asset classes which are not behaving as 'normal'. There are also many more interesting calculations which can be carried out on the network, such as working out which are the most important or influential nodes in the networks by using centrality measures [7].

The post focused on how to visualise the network using networkx and Plotly, however, there are many other libraries and software which could have been used and are worth investigating:

__Open Source Python Libraries__
- Graphviz
- Pygraphviz

__Open Source Interactive Libraries__
- d3.js

__Network drawing Software__
- Gelphi Gelphi (https://gephi.org/)


If you are interested in investigating network analytics in the financial industry in more detail, I would highly recommend checking out [FNA](https://www.fnalab.com/) which is a commercial platform for financial network analytics. They have a free trial to their platform which has some very impressive network visualisations of the stock market but also financial transcations for fraud detection and many more applications.


## References

[1] Networkx documentation: https://networkx.github.io/documentation/stable/reference/drawing.html <br>
[2]  Fruchterman, T.M. and Reingold, E.M., 1991. Graph drawing by force-directed placement. 1991. Zitiert auf den, p.37. <br>
[3] Mantegna, R.N., 1999. Hierarchical structure in financial markets. The European Physical Journal B-Condensed Matter and Complex Systems, 11(1), pp.193-197. <br>
[4] Rešovský, M., Horváth, D., Gazda, V. and Siničáková, M., 2013. Minimum Spanning Tree Application in the Currency Market. Biatec, 21(7), pp.21-23. <br>
[5] Financial Network Analytics: https://www.fna.fi/ <br>
[6] Plotly: https://plot.ly/d3-js-for-python-and-pandas-charts/  
[7] Wenyue Sun, Chuan Tian, Guang Yang, 2015, [Network Analysis of the Stock Market](http://snap.stanford.edu/class/cs224w-2015/projects_2015/Network_Analysis_of_the_Stock_Market.pdf)
