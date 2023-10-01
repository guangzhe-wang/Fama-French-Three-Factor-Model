# reproduce_three_factors_model
### Project introduction
In this project, using daily stock data from January 1996 to Junly 2023, I reproduce the classical three factors model following the formula below.
$$r_i - r_f = α + β_1MKTRF + β_2SMB + β_3SML + ε$$
Save the standard deviation of ε as idiosyncratic volatility. And the whole data is inavailable here, readers can obtain the data in WRDS CRSP.
### Contents  
In data_process_part1, all the rows with missing stock return and dislisted code are deleted. Also we only focus on the stock return and delete other columns we will not use to reduce the file size.
In data_process_part2, the firm stocks with monthly data less than 17 are also deleted. And the whole data is divided by 10 parts. Because of the big size of each file, in order to delete all monthly data with rows less than 17, I locate each firm stock by updating the index interval to avoid traversing the whole dataframe. The core code here is attached below.


```ruby
ini_index = last_index = 0 #in the first round, the index should be 0.
for ele in list_new:  
    for in_dex, row in testt.iloc[ini_index:].iterrows():
        if row['COMNAM'] != ele:
            last_index = in_dex
            break
    if ini_index == last_index:
        last_index = testt.index[-1]
    test_stock = testt.iloc[ini_index:last_index+1]
    for j in list(test_stock['year'].unique()):  
            a = test_stock[test_stock['year'] == j]
            for i in list(a['month'].unique()):    
                if len(a[a['month'] == i]) < 17:
                    #print(len(a[a['month'] == i]))
                    testt = testt.drop(a[a['month'] == i].index, axis = 0)
    ini_index = last_index
```
It takes nearly 13 minutes to deal with one part in which there are over 2700 firm stocks with more than 5 millions pieces of data.
In regression.py, I run OLS regression using daily excess return, size, book to market ratio and excess market return in which the excess return works as dependent variable and compute the standerd deviation as the monthly idiosyncratic volatility IVOL. It takes nealy 26 minutes in a round. The core code is attached below.


```ruby
for ele in list_new:  
    for in_dex, row in new_afterlink.iloc[ini_index:].iterrows():
        if row['COMNAM'] != ele:
            last_index = in_dex
            break
    if ini_index == last_index:
        last_index = new_afterlink.index[-1]
    test_stock = new_afterlink.iloc[ini_index:last_index+1]
    for j in list(test_stock['year'].unique()):  
            a = test_stock[test_stock['year'] == j]
            for i in list(a['month'].unique()): 
                b = a[a['month'] == i]
                x = b[['Mkt-RF', 'SMB', 'HML']]
                y = b['excess_return']
                x = sm.add_constant(x)
                model = sm.OLS(y, x).fit()
                residuals = model.resid
                std = np.std(residuals)
                #new_afterlink.loc[(new_afterlink.year == j)&(new_afterlink.month == i), 'IVOL'] = std
                values = test_stock.iloc[0]['COMNAM'] + '-' + str(j) + '-' + str(i) + '-' + str(std)
                ivol_list.append(values)
                
    ini_index = last_index
```
