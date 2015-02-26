---
layout: post
title: On canadian solitudes
---

I spent last weekend at the canandian hackathon
[CODE2015](https://www.canadianopendataexperience.ca).  While the "app" I
developed needs more work before it can be shown, I present here a Canadian
issue that revealed itself to me in a very simple plot I made over the weekend.

For the hackathon, [many open data sets](http://open.canada.ca/en/content
/featured-datasets) were available but I concentrated on the [Individual Tax
Statistics by Area](http://open.canada.ca/data/en/dataset/5692418d-5b96-4357
-bd3a-2a05676551f2) (for the year 2011).  It contains the income in amount
brackets, but I have only used the total income and the number of declaration to
get the average.  You can find my code to create a clean pandas dataframe with
all the available canadian cities and there average income on my [github
page](https://github.com/jfraj/CODE2015).  The only thing I will use here is the
function get_clean_tax_df from the cleanTax.py module.

Let's import what we need and improve the presentation settings.


    %matplotlib inline
    import pandas as pd
    import matplotlib.pyplot as plt
    pd.set_option('display.mpl_style', 'default')
    plt.rcParams['figure.figsize'] = (15, 5)
    import sys
    sys.path.append('../')
    from cleanTax import get_clean_tax_df

##The individual income data set
I first get the data frame of average income for each cities i.e. each row is a
city's average income calculated from all individual tax declarations.  Note
that it is ordered by income in increasing order.


    df = get_clean_tax_df('../data/cndtbl1.csv')

Here is the structure of the data frame.


    df.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>income</th>
      <th>ndeclaration</th>
      <th>province</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2743</th>
      <td> Shoal Lake 28A</td>
      <td> 1525.925926</td>
      <td> 270</td>
      <td> 47</td>
    </tr>
    <tr>
      <th>2290</th>
      <td>     Wasagamack</td>
      <td> 2192.207792</td>
      <td> 770</td>
      <td> 46</td>
    </tr>
    <tr>
      <th>2744</th>
      <td>   Red Earth 29</td>
      <td> 2201.923077</td>
      <td> 520</td>
      <td> 47</td>
    </tr>
    <tr>
      <th>2171</th>
      <td>  Dakota Tipi 1</td>
      <td> 2345.454545</td>
      <td> 110</td>
      <td> 46</td>
    </tr>
    <tr>
      <th>2165</th>
      <td>    Sandy Bay 5</td>
      <td> 2369.791667</td>
      <td> 960</td>
      <td> 46</td>
    </tr>
  </tbody>
</table>
</div>



Let's look at all the cities' average income to have an idea of the data sets


    figful, axfull = plt.subplots(1, 2)
    
    plt.subplot(1,2,1)
    df['income'].hist(bins=50)
    plt.title('Average income distribution of canadian cities')
    plt.ylabel('# of cities')
    plt.xlabel('CAD')
    
    logsubplt = plt.subplot(1,2,2)
    df['income'].hist(bins=50)
    plt.title('Average income distribution of canadian cities')
    plt.xlabel('CAD')
    plt.ylabel('# of cities')
    plt.yscale('log')
    x1,x2,y1,y2 = plt.axis()
    plt.axis((x1,x2,0.1,y2))
    _ = logsubplt.text(100000, 100, 'Vertical\nlog scale', fontsize=25, color='Red')


![png]({{jfraj.github.io}}/assets/incomeblogentry_files/incomeblogentry_8_0.png)


Both plot above contain the same data, but the one on the right has a
logarithmic scale to show the less populated income bins.  At first glance we
can see that the average income is around 30000$ per year and the highest
average income is somewhere between 140k and 150k.

##The high income tail
Since I ordered the data, it's easy to find the city with highest income, it's
the last (tail) entries.


    df.tail(n = 4)




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>income</th>
      <th>ndeclaration</th>
      <th>province</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2905</th>
      <td>       Wood Buffalo</td>
      <td>  92183.015726</td>
      <td> 54050</td>
      <td> 48</td>
    </tr>
    <tr>
      <th>2973</th>
      <td>   Foothills No. 31</td>
      <td>  93985.143288</td>
      <td> 13260</td>
      <td> 48</td>
    </tr>
    <tr>
      <th>2472</th>
      <td> Montmartre No. 126</td>
      <td> 121925.000000</td>
      <td>    40</td>
      <td> 47</td>
    </tr>
    <tr>
      <th>1422</th>
      <td> Lac-Tremblant-Nord</td>
      <td> 147500.000000</td>
      <td>    50</td>
      <td> 24</td>
    </tr>
  </tbody>
</table>
</div>



The two "richest" city have 50 or less declaration, so they might be just some
pretigious area, Lac-Tremblant-Nord for example is close the the posh ski resort
of Mont-Tremblant.  The 3rd and 4th richest are much more populated and have
income closer to the bulk distributions.  Their wealth seems to be believable
too (e.g. Wood Buffalo seems to be a [tar sands
bonanza](http://en.wikipedia.org/wiki/Regional_Municipality_of_Wood_Buffalo)).

##The low income bump
Now let's turn to the other end of the spectrum. There is an obvious isolated
cluster of cities with average income centered around 5000-10000\$. Such low
income is of the order of welfar: e.g in Quebec today it is about 7200$. But
there are cities with average income smaller that this. Maybe they have, like
the cities with highest income, small populations with few special cases
potentially affecting the average. To avoid these, let's make selection of
cities with more than 500 tax declarations and also a selection of Quebec only
to make sure they have the same welfare levels.


    allincome = df['income'].get_values()
    
    minimalpop = df['ndeclaration'] > 500
    quebec = df['province'] == 24
    
    minpopincome = df[minimalpop]['income'].get_values()
    minpopQCincome = df[minimalpop&quebec]['income'].get_values()

Now let's see how the subset of cities compares with the full sample.


    figsel, axsel = plt.subplots(1, 2)
    
    plt.subplot(1,2,1)
    plt.hist([minpopQCincome, minpopincome, allincome],
                   range=(0,160000),bins=50,stacked=False,histtype='stepfilled',
                    label=['Pop>500 and QC', 'Pop>500','All'],alpha=0.5)
    plt.title('Average income distribution of canadian cities')
    plt.xlabel('CAD')
    plt.ylabel('# of cities')
    plt.legend(loc='best')
    
    logsubselplt = plt.subplot(1,2,2)
    plt.hist([minpopQCincome, minpopincome, allincome],
                   range=(0,160000),bins=50,stacked=False,histtype='stepfilled',
                    label=['Pop>500 and QC', 'Pop>500','All'],alpha=0.5)
    plt.title('Average income distribution of canadian cities')
    plt.xlabel('CAD')
    plt.ylabel('# of cities')
    plt.yscale('log')
    x1,x2,y1,y2 = plt.axis()
    plt.axis((x1,x2,0.1,y2))
    _ = logsubselplt.text(100000, 100, 'Vertical\nlog scale', fontsize=25, color='Red')


![png]({{jfraj.github.io}}/assets/incomeblogentry_files/incomeblogentry_16_0.png)


So the cluster of low income still remains for reasonably populated cities in
Quebec.  Let see which cities they are.


    df[minimalpop&quebec].head(n=10)




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>income</th>
      <th>ndeclaration</th>
      <th>province</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1366</th>
      <td>   Akwesasne</td>
      <td>  2821.323529</td>
      <td> 1360</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1557</th>
      <td>   Obedjiwan</td>
      <td>  3044.117647</td>
      <td> 1020</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1330</th>
      <td>     Manawan</td>
      <td>  3201.886792</td>
      <td> 1060</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1556</th>
      <td>    Wemotaci</td>
      <td>  4672.222222</td>
      <td>  540</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1613</th>
      <td>    Pessamit</td>
      <td>  4828.985507</td>
      <td> 1380</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1656</th>
      <td>   Waswanipi</td>
      <td>  6660.714286</td>
      <td>  840</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1658</th>
      <td> Waskaganish</td>
      <td>  6772.727273</td>
      <td> 1100</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1661</th>
      <td>    Wemindji</td>
      <td>  7991.358025</td>
      <td>  810</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1657</th>
      <td>  Mistissini</td>
      <td>  9782.119205</td>
      <td> 1510</td>
      <td> 24</td>
    </tr>
    <tr>
      <th>1662</th>
      <td>   Chisasibi</td>
      <td> 10566.666667</td>
      <td> 1980</td>
      <td> 24</td>
    </tr>
  </tbody>
</table>
</div>



Well these names are already giving a clue of what is going on, they all belong
to first nations communities.  A bit of research (although I should know as a
canadian) shows that natives living on a reserve puts you on a different
[financial help program](http://www.mess.gouv.qc.ca/regles-normatives/a
-identification-clientele/02-statuts-particuliers/02.10.html).  Actually, a
canadian native living on a reserve is a whole [different type of citizen](http
://lois-laws.justice.gc.ca/eng/acts/I-5/index.html), with different rights.
Here is a dated but still relevant and disturbing
[paper](http://www.lactualite.com/actualites/politique/abolir-la-loi-sur-les-
indiens/)(in french) about the indian act.  Talk about solitudes!
