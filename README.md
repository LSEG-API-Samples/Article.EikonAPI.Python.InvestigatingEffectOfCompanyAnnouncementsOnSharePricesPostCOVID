# Investigating the effect of Company Announcements on their Share Price following COVID-19 (using the S&P 500)

A lot of company valuation speculation has come about since the C0rona-VIrus-Disease-2019 (COVID-19 or COVID for short) started to impact the stock market (estimated on the 20th of February 2020, 2020-02-20). Many investors tried to estimate the impact of the outbreak on businesses and trade accordingly as fast as possible. In this haste, it is possible that they miss-priced the effect of COVID on certain stocks. \
This article lays out a framework to investigate whether the Announcement of Financial Statements after COVID (*id est* (*i.e.*): after 2020-02-20) impacted the price of stocks in any specific industry sector. It will proceed simply by producing a graph of the **movement in average daily close prices for each industry - averaged from the time each company produced a Post COVID Announcement** (i.e.: after they first produced a Financial Statement after 2020-02-20). \
From there, one may stipulate that a profitable investment strategy could consist in going long in stocks of companies (i) that did not release an announcement since COVID yet (ii) within a sector that the framework bellow suggest will probably increase in price following from such an announcement.

## Pre-requisites:
Thomson Reuters Eikon with access to new Eikon Data APIs. \
Required Python Packages: [Refinitiv Eikon Python API](https://developers.refinitiv.com/eikon-apis/eikon-data-api), [Numpy](https://numpy.org/), [Pandas](https://pandas.pydata.org/) and [Matplotlib](https://matplotlib.org/). The Python built in modules [datetime](https://docs.python.org/3/library/datetime.html) and [dateutil](https://dateutil.readthedocs.io/en/stable/) are also required.

### Suplimentary:
[pickle](https://docs.python.org/3/library/pickle.html): If one wishes to copy and manipulate this code, 'pickling' data along the way should aid in making sure no data is lost when / in case there are kernel issues.


## Import libraries

First we can use the library ' platform ' to show which version of Python we are using


```python
# The ' from ... import ' structure here allows us to only import the module ' python_version ' from the library ' platform ':
from platform import python_version
print("This code runs on Python version " + python_version())
```

    This code runs on Python version 3.7.7
    

 
We use **Refinitiv's [Eikon Python Application Programming Interface (API)](https://developers.refinitiv.com/eikon-apis/eikon-data-api)** to access financial data. We can access it via the Python library "eikon" that can be installed simply by using pip install.


```python
import eikon as ek

# The key is placed in a text file so that it may be used in this code without showing it itself:
eikon_key = open("eikon.txt","r")
ek.set_app_key(str(eikon_key.read()))
# It is best to close the files we opened in order to make sure that we don't stop any other services/programs from accessing them if they need to:
eikon_key.close()
```

 
The following are Python-built-in modules/librarys, therefore they do not have specific version numbers.


```python
# datetime will allow us to manipulate Western World dates
import datetime

# dateutil will allow us to manipulate dates in equations
import dateutil
```

 
numpy is needed for datasets' statistical and mathematical manipulations


```python
import numpy
print("The numpy library imported in this code is version: " + numpy.__version__)
```

    The numpy library imported in this code is version: 1.18.2
    

 
pandas will be needed to manipulate data sets


```python
import pandas
# This line will ensure that all columns of our dataframes are always shown:
pandas.set_option('display.max_columns', None)
print("The pandas library imported in this code is version: " + pandas.__version__)
```

    The pandas library imported in this code is version: 1.0.3
    

 
matplotlib is needed to plot graphs of all kinds


```python
import matplotlib
# the use of ' as ... ' (specifically here: ' as plt ') allows us to create a shorthand for a module (here: ' matplotlib.pyplot ')
import matplotlib.pyplot as plt
print("The matplotlib library imported in this code is version: " + matplotlib.__version__)
```

    The matplotlib library imported in this code is version: 3.2.1
    

 
## Defining Functions

 
The cell below defines a function to plot data on one y axis (as opposed to two, one on the right and one on the left).


```python
# Using an implicitly registered datetime converter for a matplotlib plotting method is no longer supported by matplotlib. Current versions of pandas requires explicitly registering matplotlib converters:
pandas.plotting.register_matplotlib_converters()

def plot1ax(dataset, ylabel = "", title = "", xlabel = "Year",
            datasubset = [0], # datasubset needs to be a list of the number of each column within the dtaset that needs to be labelled on the left
            datarange = False, # If wanting to plot graph from and to a specific point, make datarange a list of start and end date
            linescolor = False, # This needs to be a list of the color of each vector to be plotted, in order they are shown in their dataframe from left to right
            figuresize = (12,4), # This can be changed to give graphs of different proportions. It is defaulted to a 12 by 4 (ratioed) graph
            facecolor="0.25",# This allows the user to change the background color as needed
            grid = True,  # This allows us to decide whether or not to include a grid in our graphs
            time_index = [], time_index_step = 48, # These two variables allow us to dictate the frequency of the ticks on the x-axis of our graph
            legend = True): 
    
    # The if statement bellow allows for manipulation of the date range that we would like to graph:
    if datarange == False:
        start_date = str(dataset.iloc[:,datasubset].index[0])
        end_date = str(dataset.iloc[:,datasubset].index[-1])
    else:
        start_date = str(datarange[0])
        
        # The if statement bellow allows us to graph to the end of the dataframe if wanted, whatever date that may be:
        if datarange[-1] == -1:
            end_date = str(dataset.iloc[:,datasubset].index[-1])
        else:
            end_date = str(datarange[-1])
    
    fig, ax1 = plt.subplots(figsize=figuresize, facecolor=facecolor)
    ax1.tick_params(axis = 'both', colors = 'w')
    ax1.set_facecolor(facecolor)
    fig.autofmt_xdate()
    plt.ylabel(ylabel, color ='w')
    ax1.set_xlabel(str(xlabel), color = 'w')
    
    if linescolor == False:
        for i in datasubset: # This is to label all the lines in order to allow matplot lib to create a legend
            ax1.plot(dataset.iloc[:, i].loc[start_date : end_date],
                     label = str(dataset.columns[i]))
    else:
        for i in datasubset: # This is to label all the lines in order to allow matplot lib to create a legend
            ax1.plot(dataset.iloc[:, i].loc[start_date : end_date],
                     label = str(dataset.columns[i]),
                     color = linescolor)
    
    ax1.tick_params(axis='y')
    
    if grid == True:
        ax1.grid()
    else:
        pass
    
    if len(time_index) != 0:
        # locs, labels = plt.xticks()
        plt.xticks(numpy.arange(len(dataset.iloc[:,datasubset]), step = time_index_step), [i for i in time_index[0::time_index_step]])
    else:
        pass
    
    ax1.set_title(str(title) + " \n", color='w')

    
    if legend == True:
        plt.legend()
    elif legend == "underneath":
        ax1.legend(loc = 'upper center', bbox_to_anchor = (0.5, -0.3), fancybox = True, shadow = True, ncol = 5)
    elif legend != False:
        plt.legend().get_texts()[0].set_text(legend)
    
    
    plt.show()
```

 
The cell bellow defines a function that adds a series of daily close prices to the dataframe named 'daily_df' and plots it.


```python
# Defining the ' daily_df ' variable before the ' Get_Daily_Close ' function
daily_df = pandas.DataFrame()

def Get_Daily_Close(instrument, # Name of the instrument in a list.
                    days_back, # Number of days from which to collect the data.
                    plot_title = False, # If ' = True ', then a graph of the data will be shown.
                    plot_time_index_step = 30 * 3, # This line dictates the index frequency on the graph/plot's x axis.
                    col = ""): # This can be changed to name the column of the merged dataframe.
    
    # This instructs the function to use a pre-defined ' daily_df ' variable:
    global daily_df
    
    if col == "":
        # If ' col ' is not defined, then the column name of the data will be replaced with its instrument abbreviated name followed by " Close Price".
        col = str(instrument) + " Close Price"
    else:
        pass
    
    # This allows for the function to programmatically ensure that all instruments' data are collected - regardless of potential server Timeout Errors.
    worked = False
    while worked != True:
        try:
            instrument, err = ek.get_data(instruments = instrument,
                                          fields = [str("TR.CLOSEPRICE(SDate=-" + str(days_back) + ",EDate=0,Frq=D,CALCMETHOD=CLOSE).timestamp"),
                                                    str("TR.CLOSEPRICE(SDate=-" + str(days_back) + ",EDate=0,Frq=D,CALCMETHOD=CLOSE)")])
            instrument.dropna()
            worked = True
        except:
            # Note that this ' except ' is necessary
            pass
            
    instrument = pandas.DataFrame(list(instrument.iloc[:,2]), index = list(instrument.iloc[:,1]), columns = [col])
    instrument.index = pandas.to_datetime(instrument.index, format = "%Y-%m-%d")
    
    if plot_title != False:
            plot1ax(dataset = instrument.dropna(), ylabel = "Close Price", title = str(plot_title), xlabel = "Year", # legend ="Close Price",
                    linescolor = "#ff9900", time_index_step = plot_time_index_step, time_index = instrument.dropna().index)
    
    daily_df = pandas.merge(daily_df, instrument, how = "outer", left_index = True, right_index = True)
```

 
The cell bellow sets up a function that gets Eikon recorded Company Announcement Data through time for any index (or instrument)


```python
def Get_Announcement_For_Index(index_instrument, periods_back, show_df = False, show_list = False):
    
    # This allows the function to collect a list of all constituents of the index
    index_issuer_rating, err = ek.get_data(index_instrument, ["TR.IssuerRating"])
    index_Announcement_list = []
    
    for i in range(len(index_issuer_rating)):
        
        # This allows for the function to programmatically ensure that all instruments' data are collected - regardless of potential server Timeout Errors.
        worked = False
        while worked != True:
            try: # The ' u ' in ' index_issuer_rating_u ' is for 'unique' as it will be for each unique instrument
                index_Announcement_u, err = ek.get_data(index_issuer_rating.iloc[i,0],
                                                        ["TR.JPINCOriginalAnnouncementDate(SDate=-" + str(periods_back) + ",EDate=0,,Period=FI0,Frq=FI)",
                                                         "TR.JPCASOriginalAnnouncementDate(SDate=-" + str(periods_back) + ",EDate=0,,Period=FI0,Frq=FI)",
                                                         "TR.JPBALOriginalAnnouncementDate(SDate=-" + str(periods_back) + ",EDate=0,,Period=FI0,Frq=FI)"])
                worked = True
            except:
                # Note that this ' except ' is necessary
                pass
        
        index_Announcement_list.append(index_Announcement_u)
    
    index_Instrument = []
    index_Income_Announcement = []
    index_Cash_Announcement = []
    index_Balance_Announcement = []
    for i in range(len(index_Announcement_list)):
        for j in range(len(index_Announcement_list[i])):
            index_Instrument.append(index_Announcement_list[i].iloc[j,0])
            index_Income_Announcement.append(index_Announcement_list[i].iloc[j,1])
            index_Cash_Announcement.append(index_Announcement_list[i].iloc[j,2])
            index_Balance_Announcement.append(index_Announcement_list[i].iloc[j,3])
            
    index_Announcement_df = pandas.DataFrame(columns = ["Instrument",
                                                        "Income Statement Announcement Date",
                                                        "Cash Flos Statement Announcement Date",
                                                        "Balance Sheet Announcement Date"])
    index_Announcement_df.iloc[:,0] = index_Instrument
    index_Announcement_df.iloc[:,1] = pandas.to_datetime(index_Income_Announcement)
    index_Announcement_df.iloc[:,2] = pandas.to_datetime(index_Cash_Announcement)
    index_Announcement_df.iloc[:,3] = pandas.to_datetime(index_Balance_Announcement)

    if show_df == True:
        display(index_Announcement_df)
    else:
        pass
    
    if show_list == True:
        for i in range(len(index_Announcement_list)):
            display(index_Announcement_list[i])
    else:
        pass
    
    return index_Announcement_df, index_Announcement_list
```

 
## Setting Up Dates

Before starting to investigate data pre- or post-COVID, we need to define the specific time when COVID affected stock markets: In this instance we chose "2020-02-20"


```python
COVID_start_date = datetime.datetime.strptime("2020-02-20", '%Y-%m-%d').date()
days_since_COVID = (datetime.date.today() - COVID_start_date).days
```

 
## Announcements

The bellow collects announcements of companies within the index of choice for the past 3 financial periods. In this article, the Standard & Poor's 500 Index (S&P500 or SPX for short) is used as an example. It can be used with indices such as FTSE or DJI instead of the SPX.


```python
index_Announcement_df, index_Announcement_list = Get_Announcement_For_Index(index_instrument = ["0#.SPX"],
                                                                            periods_back = 3,
                                                                            show_df = False,
                                                                            show_list = False)

    

Now we can choose only announcements post COVID.


```python
Announcement_COVID_date = []
for k in (1,2,3):
    index_Instruments_COVID_date = []
    index_Announcement_post_COVID_list = []
    for i in range(len(index_Announcement_list)):
        index_Instrument_COVID_date = []
        for j in reversed(index_Announcement_list[i].iloc[:,1]):
            try: # Note that ' if (index_Announcement_list[i].iloc[1,1] - COVID_start_date).days >= 0: ' would not work
                if (datetime.datetime.strptime(index_Announcement_list[i].iloc[:,1].iloc[-1], '%Y-%m-%d').date() - COVID_start_date).days >= 0:
                    while len(index_Instrument_COVID_date) == 0:
                        if (datetime.datetime.strptime(j, '%Y-%m-%d').date() - datetime.datetime.strptime("2020-02-20", '%Y-%m-%d').date()).days >= 0:
                            index_Instrument_COVID_date.append(j)
                else:
                    index_Instrument_COVID_date.append("NaT")
            except:
                index_Instrument_COVID_date.append("NaT")
        index_Instruments_COVID_date.append(index_Instrument_COVID_date[0])
    Instruments_Announcement_COVID_date = pandas.DataFrame(index_Instruments_COVID_date, index = index_Announcement_df.Instrument.unique(), columns = ["Date"])
    Instruments_Announcement_COVID_date.Date = pandas.to_datetime(Instruments_Announcement_COVID_date.Date)
    Announcement_COVID_date.append(Instruments_Announcement_COVID_date)
```


```python
Instruments_Income_Statement_Announcement_COVID_date = Announcement_COVID_date[0]
Instruments_Income_Statement_Announcement_COVID_date.columns = ["Date of the First Income Statement Announced after COVID"]
```


```python
Instruments_Cash_Flow_Statement_Announcement_COVID_date = Announcement_COVID_date[1]
Instruments_Cash_Flow_Statement_Announcement_COVID_date.columns = ["Date of the First Cash Flow Statement Announced after COVID"]
```


```python
Instruments_Balance_Sheet_COVID_date = Announcement_COVID_date[2]
Instruments_Balance_Sheet_COVID_date.columns = ["Date of the First Balance Sheet Announced after COVID"]
```

 
## Daily Price

### Post COVID

The cell bellow collects Daily Close Prices for all relevant instruments in the index chosen.


```python
for i in index_Announcement_df.iloc[:,0].unique():
    Get_Daily_Close(i, days_back = days_since_COVID)
```

    2020-05-12 18:36:15,285 P[22472] [MainThread 2468] Backend error. 400 Bad Request
    

Some instruments might have been added to the index midway during out time period of choice. They are the ones bellow:


```python
removing = [i.split()[0]  + " Close Price" for i in daily_df.iloc[0,:][daily_df.iloc[0,:].isna() == True].index]
print("We will be removing " + removing + " from our dataframe")
```

    ['CARR.N Close Price', 'OTIS.N Close Price']
    

The cell bellow will remove them to make sure that the do not skew our statistics later on in the code.


```python
# This line removes instruments that wera added midway to the index
daily_df_no_na = daily_df.drop(removing, axis = 1).dropna()
```

Now we can focus on stock price movements alone.


```python
daily_df_trend = pandas.DataFrame(columns = daily_df_no_na.columns)
for i in range(len(pandas.DataFrame.transpose(daily_df_no_na))):
    daily_df_trend.iloc[:,i] = daily_df_no_na.iloc[:,i] - daily_df_no_na.iloc[0,i]
```

The following 3 cells display plots to visualise our data this far.


```python
datasubset_list = []
for i in range(len(daily_df_no_na.columns)):
    datasubset_list.append(i)
```


```python
plot1ax(dataset = daily_df_no_na,
        ylabel = "Close Price",
        title = "Index Constituents' Close Prices",
        xlabel = "Date",
        legend = False,
        datasubset = datasubset_list)
```


![png44](images/output_44_0.png "Data item browser")



```python
plot1ax(dataset = daily_df_trend, legend = False,
        ylabel = "Normalised Close Price",
        title = "Index Constituents' Change in Close Prices",
        datasubset = datasubset_list, xlabel = "Date",)
```


![png45](images/output_45_0.png "Data item browser")


The graph above shows the change in constituent companies' close prices since COVID.


## Saving our data

The cell bellow saves variables to a 'pickle' file to quicken subsequent runs of this code if they are seen as necessary.


```python
# pip install pickle-mixin
import pickle

pickle_out = open("SPX.pickle","wb")
pickl = (COVID_start_date, days_since_COVID,
         index_Announcement_df, index_Announcement_list,
         Announcement_COVID_date,
         Instruments_Income_Statement_Announcement_COVID_date,
         Instruments_Cash_Flow_Statement_Announcement_COVID_date,
         Instruments_Balance_Sheet_COVID_date,
         daily_df, daily_df_no_na,
         daily_df_trend, datasubset_list)
pickle.dump(pickl, pickle_out)
pickle_out.close()
```

The cell bellow can be run to load these variables back into the kernel


```python
# pickle_in = open("pickl.pickle","rb")
# COVID_start_date, days_since_COVID, index_Announcement_df, index_Announcement_list, Announcement_COVID_date, Instruments_Income_Statement_Announcement_COVID_date, Instruments_Cash_Flow_Statement_Announcement_COVID_date, Instruments_Balance_Sheet_COVID_date, daily_df, daily_df_no_na, daily_df_trend, datasubset_list = pickle.load(pickle_in)
```

 
## Post-COVID-Announcement Price Insight

Now we can start investigating price changes after the first Post-COVID-Announcement of each company in our dataset.


```python
# This is just to delimitate between the code before and after this point
daily_df2 = daily_df_no_na
```

The cell bellow formats the date-type of our data to enable us to apply them to simple algebra.


```python
date_in_date_format = []
for k in range(len(daily_df2)):
    date_in_date_format.append(daily_df2.index[k].date())
daily_df2.index = date_in_date_format
```

The cell bellow renames the columns of our dataset.


```python
daily_df2_instruments = []
for i in daily_df2.columns:
    daily_df2_instruments.append(str.split(i)[0])
```

Now: we collect daily prices only for dates after the first Post-COVID-Announcement of each instrument of interest


```python
daily_df2_post_COVID_announcement = pandas.DataFrame()
for i,j in zip(daily_df2.columns, daily_df2_instruments):
    daily_df2_post_COVID_announcement = pandas.merge(daily_df2_post_COVID_announcement,
                                                     daily_df2[i][daily_df2.index >= Instruments_Income_Statement_Announcement_COVID_date.loc[j].iloc[0].date()],
                                                     how = "outer", left_index = True, right_index = True) # Note that the following would not work: ' daily_df2_post_COVID_announcement[i] = daily_df2[i][daily_df2.index >= Instruments_Income_Statement_Announcement_COVID_date.loc[j].iloc[0].date()] '
```

Now we can focus on the trend/change in those prices


```python
daily_df2_post_COVID_announcement_trend = pandas.DataFrame()
for i in daily_df2.columns:
    try:
        daily_df2_post_COVID_announcement_trend = pandas.merge(daily_df2_post_COVID_announcement_trend,
                                                               daily_df2_post_COVID_announcement.reset_index()[i].dropna().reset_index()[i] - daily_df2_post_COVID_announcement.reset_index()[i].dropna().iloc[0],
                                                               how = "outer", left_index = True, right_index = True)
    except:
        daily_df2_post_COVID_announcement_trend[i] = numpy.nan
```

And plot them


```python
plot1ax(dataset = daily_df2_post_COVID_announcement_trend,
        ylabel = "Normalised Close Price",
        title = "Index Constituents' Trend In Close Prices From There First Income Statement Announcement Since COVID\n" + 
                "Only companies that announced an Income Statement since the start of COVID (i.e.:" + str(COVID_start_date) + ") will show",
        xlabel = "Days since first Post-COVID-Announcement",
        legend = False, # change to "underneath" to see list of all instruments and their respective colors as per this graph's legend.
        datasubset = datasubset_list)
```


![Figure-1](images/output_64_0.png "Data item browser")


Some companies have lost and gained a great deal following from their first Post-COVID-Announcement, but most seem to have changed by less than 50 United States of america Dollars (USD).

 
### Post COVID Announcement Price Change

The cell bellow simply gathers all stocks that decreased, increased or did not change in price since their first Post-COVID-Announcement in an easy to digest [pandas](https://pandas.pydata.org/) table. Note that is they haven't had a Post-COVID-Announcement yet, they will show as unchanged.


```python
COVID_priced_in = [[],[],[]]
for i in daily_df2_post_COVID_announcement_trend.columns:
    if str(sum(daily_df2_post_COVID_announcement_trend[i].dropna())) != "nan":
        if numpy.mean(daily_df2_post_COVID_announcement_trend[i].dropna()) < 0:
            COVID_priced_in[0].append(str.split(i)[0])
        if numpy.mean(daily_df2_post_COVID_announcement_trend[i].dropna()) == 0:
            COVID_priced_in[1].append(str.split(i)[0])
        if numpy.mean(daily_df2_post_COVID_announcement_trend[i].dropna()) > 0:
            COVID_priced_in[2].append(str.split(i)[0])
COVID_priced_in = pandas.DataFrame(COVID_priced_in, index = ["Did not have the negative impact of COVID priced in enough",
                                                             "Had the effects of COVID priced in (or didn't have time to react to new company announcements)",
                                                             "Had a price that overcompensated the negative impact of COVID"])
COVID_priced_in
```



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>10</th>
      <th>11</th>
      <th>12</th>
      <th>13</th>
      <th>14</th>
      <th>15</th>
      <th>16</th>
      <th>17</th>
      <th>18</th>
      <th>19</th>
      <th>20</th>
      <th>21</th>
      <th>22</th>
      <th>23</th>
      <th>24</th>
      <th>25</th>
      <th>26</th>
      <th>27</th>
      <th>28</th>
      <th>29</th>
      <th>30</th>
      <th>31</th>
      <th>32</th>
      <th>33</th>
      <th>34</th>
      <th>35</th>
      <th>36</th>
      <th>37</th>
      <th>38</th>
      <th>39</th>
      <th>40</th>
      <th>41</th>
      <th>42</th>
      <th>43</th>
      <th>44</th>
      <th>45</th>
      <th>46</th>
      <th>47</th>
      <th>48</th>
      <th>49</th>
      <th>50</th>
      <th>51</th>
      <th>52</th>
      <th>53</th>
      <th>54</th>
      <th>55</th>
      <th>56</th>
      <th>57</th>
      <th>58</th>
      <th>59</th>
      <th>60</th>
      <th>61</th>
      <th>62</th>
      <th>63</th>
      <th>64</th>
      <th>65</th>
      <th>66</th>
      <th>67</th>
      <th>68</th>
      <th>69</th>
      <th>70</th>
      <th>71</th>
      <th>72</th>
      <th>73</th>
      <th>74</th>
      <th>75</th>
      <th>76</th>
      <th>77</th>
      <th>78</th>
      <th>79</th>
      <th>80</th>
      <th>81</th>
      <th>82</th>
      <th>83</th>
      <th>84</th>
      <th>85</th>
      <th>86</th>
      <th>87</th>
      <th>88</th>
      <th>89</th>
      <th>90</th>
      <th>91</th>
      <th>92</th>
      <th>93</th>
      <th>94</th>
      <th>95</th>
      <th>96</th>
      <th>97</th>
      <th>98</th>
      <th>99</th>
      <th>100</th>
      <th>101</th>
      <th>102</th>
      <th>103</th>
      <th>104</th>
      <th>105</th>
      <th>106</th>
      <th>107</th>
      <th>108</th>
      <th>109</th>
      <th>110</th>
      <th>111</th>
      <th>112</th>
      <th>113</th>
      <th>114</th>
      <th>115</th>
      <th>116</th>
      <th>117</th>
      <th>118</th>
      <th>119</th>
      <th>120</th>
      <th>121</th>
      <th>122</th>
      <th>123</th>
      <th>124</th>
      <th>125</th>
      <th>126</th>
      <th>127</th>
      <th>128</th>
      <th>129</th>
      <th>130</th>
      <th>131</th>
      <th>132</th>
      <th>133</th>
      <th>134</th>
      <th>135</th>
      <th>136</th>
      <th>137</th>
      <th>138</th>
      <th>139</th>
      <th>140</th>
      <th>141</th>
      <th>142</th>
      <th>143</th>
      <th>144</th>
      <th>145</th>
      <th>146</th>
      <th>147</th>
      <th>148</th>
      <th>149</th>
      <th>150</th>
      <th>151</th>
      <th>152</th>
      <th>153</th>
      <th>154</th>
      <th>155</th>
      <th>156</th>
      <th>157</th>
      <th>158</th>
      <th>159</th>
      <th>160</th>
      <th>161</th>
      <th>162</th>
      <th>163</th>
      <th>164</th>
      <th>165</th>
      <th>166</th>
      <th>167</th>
      <th>168</th>
      <th>169</th>
      <th>170</th>
      <th>171</th>
      <th>172</th>
      <th>173</th>
      <th>174</th>
      <th>175</th>
      <th>176</th>
      <th>177</th>
      <th>178</th>
      <th>179</th>
      <th>180</th>
      <th>181</th>
      <th>182</th>
      <th>183</th>
      <th>184</th>
      <th>185</th>
      <th>186</th>
      <th>187</th>
      <th>188</th>
      <th>189</th>
      <th>190</th>
      <th>191</th>
      <th>192</th>
      <th>193</th>
      <th>194</th>
      <th>195</th>
      <th>196</th>
      <th>197</th>
      <th>198</th>
      <th>199</th>
      <th>200</th>
      <th>201</th>
      <th>202</th>
      <th>203</th>
      <th>204</th>
      <th>205</th>
      <th>206</th>
      <th>207</th>
      <th>208</th>
      <th>209</th>
      <th>210</th>
      <th>211</th>
      <th>212</th>
      <th>213</th>
      <th>214</th>
      <th>215</th>
      <th>216</th>
      <th>217</th>
      <th>218</th>
      <th>219</th>
      <th>220</th>
      <th>221</th>
      <th>222</th>
      <th>223</th>
      <th>224</th>
      <th>225</th>
      <th>226</th>
      <th>227</th>
      <th>228</th>
      <th>229</th>
      <th>230</th>
      <th>231</th>
      <th>232</th>
      <th>233</th>
      <th>234</th>
      <th>235</th>
      <th>236</th>
      <th>237</th>
      <th>238</th>
      <th>239</th>
      <th>240</th>
      <th>241</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Did not have the negative impact of COVID priced in enough</th>
      <td>CHRW.OQ</td>
      <td>PRGO.N</td>
      <td>BA.N</td>
      <td>MCD.N</td>
      <td>HD.N</td>
      <td>COG.N</td>
      <td>AIZ.N</td>
      <td>COST.OQ</td>
      <td>AMD.OQ</td>
      <td>REG.OQ</td>
      <td>TRV.N</td>
      <td>MOS.N</td>
      <td>WDC.OQ</td>
      <td>VTR.N</td>
      <td>STX.OQ</td>
      <td>VRSN.OQ</td>
      <td>LB.N</td>
      <td>LOW.N</td>
      <td>BSX.N</td>
      <td>MAS.N</td>
      <td>BEN.N</td>
      <td>RJF.N</td>
      <td>CE.N</td>
      <td>LLY.N</td>
      <td>NWL.OQ</td>
      <td>AVB.N</td>
      <td>TPR.N</td>
      <td>CINF.OQ</td>
      <td>SEE.N</td>
      <td>HON.N</td>
      <td>NBL.OQ</td>
      <td>EA.OQ</td>
      <td>CB.N</td>
      <td>MDLZ.OQ</td>
      <td>BLL.N</td>
      <td>JPM.N</td>
      <td>CDW.OQ</td>
      <td>COO.N</td>
      <td>UAL.OQ</td>
      <td>KHC.OQ</td>
      <td>PNR.N</td>
      <td>KSS.N</td>
      <td>DPZ.N</td>
      <td>HPQ.N</td>
      <td>TGT.N</td>
      <td>OXY.N</td>
      <td>EXC.OQ</td>
      <td>AZO.N</td>
      <td>TEL.N</td>
      <td>MXIM.OQ</td>
      <td>AAL.OQ</td>
      <td>VLO.N</td>
      <td>LH.N</td>
      <td>SHW.N</td>
      <td>GD.N</td>
      <td>SBAC.OQ</td>
      <td>COP.N</td>
      <td>GRMN.OQ</td>
      <td>TXT.N</td>
      <td>WELL.N</td>
      <td>PLD.N</td>
      <td>ROST.OQ</td>
      <td>MRK.N</td>
      <td>WEC.N</td>
      <td>TMO.N</td>
      <td>F.N</td>
      <td>LYB.N</td>
      <td>CERN.OQ</td>
      <td>PEP.OQ</td>
      <td>HP.N</td>
      <td>ABMD.OQ</td>
      <td>PH.N</td>
      <td>NSC.N</td>
      <td>BAX.N</td>
      <td>GILD.OQ</td>
      <td>JWN.N</td>
      <td>NOC.N</td>
      <td>PNW.N</td>
      <td>BFb.N</td>
      <td>DE.N</td>
      <td>HSY.N</td>
      <td>FLT.N</td>
      <td>IT.N</td>
      <td>ECL.N</td>
      <td>BXP.N</td>
      <td>GE.N</td>
      <td>ED.N</td>
      <td>WFC.N</td>
      <td>FTV.N</td>
      <td>PRU.N</td>
      <td>DLTR.OQ</td>
      <td>NEE.N</td>
      <td>ILMN.OQ</td>
      <td>XLNX.OQ</td>
      <td>CMS.N</td>
      <td>HPE.N</td>
      <td>ALGN.OQ</td>
      <td>SRE.N</td>
      <td>REGN.OQ</td>
      <td>DHR.N</td>
      <td>CME.OQ</td>
      <td>ADM.N</td>
      <td>FANG.OQ</td>
      <td>WRB.N</td>
      <td>FLS.N</td>
      <td>TROW.OQ</td>
      <td>DRE.N</td>
      <td>MLM.N</td>
      <td>TWTR.N</td>
      <td>L.N</td>
      <td>QCOM.OQ</td>
      <td>ANSS.OQ</td>
      <td>ULTA.OQ</td>
      <td>HRB.N</td>
      <td>FISV.OQ</td>
      <td>XRX.N</td>
      <td>ANET.N</td>
      <td>HLT.N</td>
      <td>NFLX.OQ</td>
      <td>AMGN.OQ</td>
      <td>KIM.N</td>
      <td>XRAY.OQ</td>
      <td>URI.N</td>
      <td>CNC.N</td>
      <td>ANTM.N</td>
      <td>K.N</td>
      <td>LEG.N</td>
      <td>PSA.N</td>
      <td>NLSN.N</td>
      <td>HOG.N</td>
      <td>BK.N</td>
      <td>MO.N</td>
      <td>HST.N</td>
      <td>BBY.N</td>
      <td>LHX.N</td>
      <td>KSU.N</td>
      <td>FE.N</td>
      <td>VZ.N</td>
      <td>NEM.N</td>
      <td>GIS.N</td>
      <td>CMCSA.OQ</td>
      <td>PFE.N</td>
      <td>EIX.N</td>
      <td>UPS.N</td>
      <td>IP.N</td>
      <td>MMM.N</td>
      <td>INTU.OQ</td>
      <td>OKE.N</td>
      <td>ADP.OQ</td>
      <td>EQIX.OQ</td>
      <td>MTD.N</td>
      <td>PEG.N</td>
      <td>CTSH.OQ</td>
      <td>NCLH.N</td>
      <td>HIG.N</td>
      <td>UHS.N</td>
      <td>INCY.OQ</td>
      <td>UNM.N</td>
      <td>LRCX.OQ</td>
      <td>PPG.N</td>
      <td>LKQ.OQ</td>
      <td>TJX.N</td>
      <td>VMC.N</td>
      <td>EW.N</td>
      <td>ALL.N</td>
      <td>MHK.N</td>
      <td>CAT.N</td>
      <td>PG.N</td>
      <td>ZTS.N</td>
      <td>AFL.N</td>
      <td>CPB.N</td>
      <td>MSI.N</td>
      <td>ITW.N</td>
      <td>GPS.N</td>
      <td>ALK.N</td>
      <td>PGR.N</td>
      <td>DISCA.OQ</td>
      <td>KMB.N</td>
      <td>EL.N</td>
      <td>SPGI.N</td>
      <td>ADSK.OQ</td>
      <td>WHR.N</td>
      <td>AKAM.OQ</td>
      <td>LUV.N</td>
      <td>AMZN.OQ</td>
      <td>FTI.N</td>
      <td>BRKb.N</td>
      <td>KR.N</td>
      <td>BKNG.OQ</td>
      <td>MA.N</td>
      <td>SWK.N</td>
      <td>DISCK.OQ</td>
      <td>OMC.N</td>
      <td>GLW.N</td>
      <td>CRM.N</td>
      <td>SBUX.OQ</td>
      <td>TAP.N</td>
      <td>ABT.N</td>
      <td>JNPR.N</td>
      <td>IRM.N</td>
      <td>PEAK.N</td>
      <td>FIS.N</td>
      <td>ROK.N</td>
      <td>HBI.N</td>
      <td>DOW.N</td>
      <td>PPL.N</td>
      <td>ETN.N</td>
      <td>CI.N</td>
      <td>XYL.N</td>
      <td>HAS.OQ</td>
      <td>SO.N</td>
      <td>VNO.N</td>
      <td>CTL.N</td>
      <td>DLR.N</td>
      <td>AVY.N</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>Had the effects of COVID priced in (or didn't have time to react to new company announcements)</th>
      <td>AMCR.N</td>
      <td>SPG.N</td>
      <td>COTY.N</td>
      <td>UAA.N</td>
      <td>MYL.OQ</td>
      <td>CAH.N</td>
      <td>ZBH.N</td>
      <td>AEE.N</td>
      <td>IFF.N</td>
      <td>MAR.OQ</td>
      <td>ETR.N</td>
      <td>UA.N</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>Had a price that overcompensated the negative impact of COVID</th>
      <td>AJG.N</td>
      <td>CNP.N</td>
      <td>WM.N</td>
      <td>FOX.OQ</td>
      <td>WY.N</td>
      <td>HBAN.OQ</td>
      <td>QRVO.OQ</td>
      <td>LVS.N</td>
      <td>AIG.N</td>
      <td>MCO.N</td>
      <td>DIS.N</td>
      <td>PAYX.OQ</td>
      <td>DHI.N</td>
      <td>BWA.N</td>
      <td>IVZ.N</td>
      <td>ZBRA.OQ</td>
      <td>SYK.N</td>
      <td>NVR.N</td>
      <td>SYY.N</td>
      <td>FCX.N</td>
      <td>EVRG.N</td>
      <td>EXPD.OQ</td>
      <td>PAYC.N</td>
      <td>AIV.N</td>
      <td>CL.N</td>
      <td>UNH.N</td>
      <td>ARE.N</td>
      <td>TIF.N</td>
      <td>ISRG.OQ</td>
      <td>PVH.N</td>
      <td>ADS.N</td>
      <td>CBRE.N</td>
      <td>WMB.N</td>
      <td>TMUS.OQ</td>
      <td>TXN.OQ</td>
      <td>PFG.OQ</td>
      <td>KEYS.N</td>
      <td>INFO.N</td>
      <td>RMD.N</td>
      <td>LNT.OQ</td>
      <td>NOV.N</td>
      <td>CMG.N</td>
      <td>PCAR.OQ</td>
      <td>CHTR.OQ</td>
      <td>PWR.N</td>
      <td>CCL.N</td>
      <td>PM.N</td>
      <td>SNA.N</td>
      <td>ESS.N</td>
      <td>JBHT.OQ</td>
      <td>LW.N</td>
      <td>IR.N</td>
      <td>WAB.N</td>
      <td>FBHS.N</td>
      <td>NOW.N</td>
      <td>BR.N</td>
      <td>EMN.N</td>
      <td>FRT.N</td>
      <td>DVN.N</td>
      <td>MKC.N</td>
      <td>IEX.N</td>
      <td>DRI.N</td>
      <td>MAA.N</td>
      <td>JNJ.N</td>
      <td>ABC.N</td>
      <td>ORCL.N</td>
      <td>AMP.N</td>
      <td>ALXN.OQ</td>
      <td>T.N</td>
      <td>NDAQ.OQ</td>
      <td>FFIV.OQ</td>
      <td>SLG.N</td>
      <td>LEN.N</td>
      <td>CDNS.OQ</td>
      <td>MSCI.N</td>
      <td>IBM.N</td>
      <td>VRTX.OQ</td>
      <td>DXCM.OQ</td>
      <td>AOS.N</td>
      <td>BLK.N</td>
      <td>HII.N</td>
      <td>CVS.N</td>
      <td>MSFT.OQ</td>
      <td>HUM.N</td>
      <td>CMA.N</td>
      <td>GL.N</td>
      <td>SLB.N</td>
      <td>AWK.N</td>
      <td>PKG.N</td>
      <td>MMC.N</td>
      <td>BKR.N</td>
      <td>ALLE.N</td>
      <td>CTVA.N</td>
      <td>CF.N</td>
      <td>AEP.N</td>
      <td>FMC.N</td>
      <td>KLAC.OQ</td>
      <td>AME.N</td>
      <td>NUE.N</td>
      <td>WU.N</td>
      <td>D.N</td>
      <td>SJM.N</td>
      <td>EMR.N</td>
      <td>DVA.N</td>
      <td>RHI.N</td>
      <td>BDX.N</td>
      <td>MGM.N</td>
      <td>EFX.N</td>
      <td>USB.N</td>
      <td>HCA.N</td>
      <td>NWSA.OQ</td>
      <td>AAPL.OQ</td>
      <td>MRO.N</td>
      <td>APD.N</td>
      <td>VAR.N</td>
      <td>AXP.N</td>
      <td>CLX.N</td>
      <td>DOV.N</td>
      <td>FDX.N</td>
      <td>JKHY.OQ</td>
      <td>O.N</td>
      <td>PKI.N</td>
      <td>IPG.N</td>
      <td>PHM.N</td>
      <td>PBCT.OQ</td>
      <td>KO.N</td>
      <td>WLTW.OQ</td>
      <td>ES.N</td>
      <td>CTAS.OQ</td>
      <td>GPN.N</td>
      <td>RSG.N</td>
      <td>RF.N</td>
      <td>PXD.N</td>
      <td>DG.N</td>
      <td>EQR.N</td>
      <td>TDG.N</td>
      <td>NKE.N</td>
      <td>AES.N</td>
      <td>GWW.N</td>
      <td>GOOGL.OQ</td>
      <td>GM.N</td>
      <td>CCI.N</td>
      <td>COF.N</td>
      <td>ROP.N</td>
      <td>CXO.N</td>
      <td>C.N</td>
      <td>ODFL.OQ</td>
      <td>GS.N</td>
      <td>MET.N</td>
      <td>WYNN.OQ</td>
      <td>FAST.OQ</td>
      <td>LDOS.N</td>
      <td>ORLY.OQ</td>
      <td>CSX.OQ</td>
      <td>CFG.N</td>
      <td>NI.N</td>
      <td>DD.N</td>
      <td>HOLX.OQ</td>
      <td>STT.N</td>
      <td>JCI.N</td>
      <td>PNC.N</td>
      <td>ROL.N</td>
      <td>KEY.N</td>
      <td>NWS.OQ</td>
      <td>MU.OQ</td>
      <td>UNP.N</td>
      <td>BAC.N</td>
      <td>KMX.N</td>
      <td>VIAC.OQ</td>
      <td>APA.N</td>
      <td>CHD.N</td>
      <td>EBAY.OQ</td>
      <td>BIIB.OQ</td>
      <td>ALB.N</td>
      <td>UDR.N</td>
      <td>DGX.N</td>
      <td>CBOE.Z</td>
      <td>ETFC.OQ</td>
      <td>VRSK.OQ</td>
      <td>AMT.N</td>
      <td>PYPL.OQ</td>
      <td>CAG.N</td>
      <td>TFX.N</td>
      <td>SYF.N</td>
      <td>WAT.N</td>
      <td>IDXX.OQ</td>
      <td>HES.N</td>
      <td>EOG.N</td>
      <td>MNST.OQ</td>
      <td>FRC.N</td>
      <td>BMY.N</td>
      <td>APH.N</td>
      <td>GPC.N</td>
      <td>MCHP.OQ</td>
      <td>FITB.OQ</td>
      <td>XEL.OQ</td>
      <td>HSIC.OQ</td>
      <td>DFS.N</td>
      <td>APTV.N</td>
      <td>MPC.N</td>
      <td>ZION.OQ</td>
      <td>ICE.N</td>
      <td>SWKS.OQ</td>
      <td>EXR.N</td>
      <td>IPGP.OQ</td>
      <td>ADBE.OQ</td>
      <td>WRK.N</td>
      <td>FOXA.OQ</td>
      <td>TSN.N</td>
      <td>TSCO.OQ</td>
      <td>AON.N</td>
      <td>MS.N</td>
      <td>GOOG.OQ</td>
      <td>STZ.N</td>
      <td>ABBV.N</td>
      <td>XOM.N</td>
      <td>HRL.N</td>
      <td>INTC.OQ</td>
      <td>WBA.OQ</td>
      <td>ATO.N</td>
      <td>HAL.N</td>
      <td>TFC.N</td>
      <td>ATVI.OQ</td>
      <td>V.N</td>
      <td>LYV.N</td>
      <td>RE.N</td>
      <td>FTNT.OQ</td>
      <td>DAL.N</td>
      <td>NTRS.OQ</td>
      <td>CVX.N</td>
      <td>LNC.N</td>
      <td>ACN.N</td>
      <td>CTXS.OQ</td>
      <td>DISH.OQ</td>
      <td>NRG.N</td>
      <td>MKTX.OQ</td>
      <td>LMT.N</td>
      <td>PSX.N</td>
      <td>FLIR.OQ</td>
      <td>SCHW.N</td>
      <td>J.N</td>
      <td>SIVB.OQ</td>
    </tr>
  </tbody>
</table>
</div>



 
## Informative Powers of Announcements Per Sector

We will now investigate the insight behind our analysis per industry sector.

The 2 cells bellow allow us to see the movement in daily price of companies with Post-COVID-Announcements per sector


```python
ESector, err = ek.get_data(instruments = [i.split()[0] for i in daily_df2_post_COVID_announcement_trend.dropna(axis = "columns", how = "all").columns],
                           fields = ["TR.TRBCEconSectorCode",
                                     "TR.TRBCBusinessSectorCode",
                                     "TR.TRBCIndustryGroupCode",
                                     "TR.TRBCIndustryCode",
                                     "TR.TRBCActivityCode"])

ESector["TRBC Economic Sector"] = numpy.nan
ESector_list = [[],[],[],[],[],[],[],[],[],[]]
Sectors_list = ["Energy", "Basic Materials", "Industrials", "Consumer Cyclicals",
                "Consumer Non-Cyclicals", "Financials", "Healthcare",
                "Technology", "Telecommunication Services", "Utilities"]

for i in range(len(ESector["TRBC Economic Sector Code"])):
    for j,k in zip(range(0, 10), Sectors_list):
        if ESector.iloc[i,1] == (50 + j):
            ESector.iloc[i,6] = k
            ESector_list[j].append(ESector.iloc[i,0])

ESector_df = numpy.transpose(pandas.DataFrame(data = [ESector_list[i] for i in range(len(ESector_list))],
                                              index = Sectors_list))

ESector_df_by_Sector = []
for k in Sectors_list:
    ESector_df_by_Sector.append(numpy.average([numpy.average(daily_df2_post_COVID_announcement_trend[i + " Close Price"].dropna()) for i in [j for j in ESector_df[k].dropna()]]))

ESector_average = pandas.DataFrame(data = ESector_df_by_Sector,
                                   columns = ["Average of Close Prices Post COVID Announcement"],
                                   index = Sectors_list)


```python
ESector_average
```




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
      <th>Average of Close Prices Post COVID Announcement</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Energy</th>
      <td>0.510261</td>
    </tr>
    <tr>
      <th>Basic Materials</th>
      <td>-0.113836</td>
    </tr>
    <tr>
      <th>Industrials</th>
      <td>0.557313</td>
    </tr>
    <tr>
      <th>Consumer Cyclicals</th>
      <td>1.606089</td>
    </tr>
    <tr>
      <th>Consumer Non-Cyclicals</th>
      <td>0.763417</td>
    </tr>
    <tr>
      <th>Financials</th>
      <td>1.065371</td>
    </tr>
    <tr>
      <th>Healthcare</th>
      <td>0.070465</td>
    </tr>
    <tr>
      <th>Technology</th>
      <td>5.372840</td>
    </tr>
    <tr>
      <th>Telecommunication Services</th>
      <td>1.167946</td>
    </tr>
    <tr>
      <th>Utilities</th>
      <td>-0.540518</td>
    </tr>
  </tbody>
</table>
</div>



The 'ESector_average'  table above shows the Close Prices Post COVID-Announcement for each company averaged per sector
 

 
The cells bellow now allow us to visualise this trend in a graph on an industry sector basis


```python
Sector_Average = []
for k in ESector_average.index:
    Sector_Average1 = []
    for j in range(len(pandas.DataFrame([daily_df2_post_COVID_announcement_trend[i + " Close Price"].dropna() for i in ESector_df[k].dropna()]).columns)):
        Sector_Average1.append(numpy.average(pandas.DataFrame([daily_df2_post_COVID_announcement_trend[i + " Close Price"].dropna() for i in ESector_df[k].dropna()]).iloc[:,j].dropna()))
    Sector_Average.append(Sector_Average1)
```


```python
Sector_Average = numpy.transpose(pandas.DataFrame(Sector_Average, index = ESector_average.index))
```

This cell bellow in particular allows us to collect and save our data before continuing so that we don't have to ask for data from Eikon again were we to manipulate the same content later (just in case)


```python
pickle_out = open("SPX2.pickle","wb")
pickl = (COVID_start_date, days_since_COVID,
         index_Announcement_df, index_Announcement_list,
         Announcement_COVID_date,
         Instruments_Income_Statement_Announcement_COVID_date,
         Instruments_Cash_Flow_Statement_Announcement_COVID_date,
         Instruments_Balance_Sheet_COVID_date,
         daily_df, daily_df_no_na,
         daily_df_trend, datasubset_list)
pickle.dump(pickl, pickle_out)
pickle_out.close()
```


```python
plot1ax(dataset = Sector_Average, ylabel = "Price Movement",
        title = "Index Constituents' Trend In Close Prices From There First Income Statement Announcement Since COVID Sorted By Sector\n" + 
                "Only companies that announced an Income Statement since the start of COVID (i.e.:" + str(COVID_start_date) + ") will show",
        xlabel = "Trading Day", legend = "underneath",
        datasubset = [i for i in range(len(Sector_Average.columns))])
```


![png45](images/output_80_0.png "Data item browser")


 
# Conclusion

Using S&P 500 (i.e.: SPX) data, we can have a wholesome picture of industries in the United States of America (USA). We can see a great negative change in instruments’ daily close prices for stocks in the Consumer Cyclical, Utilities, Healthcare and Industrial markets. This is actually surprising because they are the industries that were suggested to be most hindered by COVID in the media before their financial statement announcements; investors thus ought to have priced the negative effects of the Disease on these market sectors appropriately. \
The graph suggests that it may be profitable to short companies within these sectors just before they are due to release their first post-COVID Financial Statements - but naturally does not account for future changes, trade costs or other such variants external to this investigation. \
Companies in the Financial sector seem to have performed adequately. Reasons for movements in this sector can be complex and numerous due to their exposure to all other sectors. \
Tech companies seem to have had the impact of COVID priced in prior to the release of their financial statements. One may postulate the impact of COVID on their share price was actually positive as people rush to online infrastructures they support during confinement. \
Companies dealing with Basic Material have performed relatively well. This may be an indication that investors are losing confidence in all but sectors that offer physical goods in supply chains (rather than in consumer goods) - a retreat to fundamentals in a time of uncertainty. \

**BUT** one must use both the ESector_average table and the last graph before coming to any conclusion. The ESector_average - though simple - can provide more depth to our analysis. Take the Healthcare sector for example: One may assume – based on the last graph alone – that this sector is performing badly when revealing information via Announcements; but the ESector_average shows a positive ‘Average of Close Prices Post COVID Announcement’. This is because only very few companies within the Healthcare sector published Announcements before May 2020, and the only ones that did performed badly, skewing the data negatively on the graph.


```python

```
