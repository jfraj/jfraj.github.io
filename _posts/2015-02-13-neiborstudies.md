---
layout: post
title: Banks and Pawn shops in Montreal's arrondissements
---

During a meeup of MtlData, a question was asked if there was any way to know the
value property of a neighborhood.  Of course, the simplest answer would be to
look at the housing tax property but we were told that, although this is public
data, it is not easy to access systematically by a script.  So the question was:
is it possible to infer the property values from some data available on the web?

Then I remembered the very nice tv documentary "Les naufragés des villes" airing
on Radio-Canada.  The documentary is about poverty and the obstacles to get out
of it.  One thing that striked me was the mentionning that banks tend to be
around wealthier neiberhoods conviniently close to their prefered clients. On
the other hand the pawn shops -owing their profit to people's financial
difficulties- usually fill that void.  One simple example is the delay to get
your money from a check in a bank can be several days while the pawn shops will
do it for a fee.  If you need money right away and no savings, the pawn shops
might be your only alternative.  The assumption is that pawn shops tend to be
close to those in need without savings and banks close to those who have
savings.  Hence making those two types of business economical markers of a
neighborhood.

I will try below to check if this assumption make sense (although I will not try
to prove it).

##Getting data
To get the list of pawn shop and bank address, I decided to copy-paste a search
at yellowpage.ca for the keyword "pawn" and "bank"for the region of Montreal.
After some wrangling, I produced two csv files which contained the postal code
of the businesses.  In order to turn an address into a neighborhood
(arrondissement) name, I used this [wiki page](http://en.wikipedia.org/w/index.p
hp?title=List_of_H_postal_codes_of_Canada&action=edit) and wrangled its source
code into a csv file.  Finally, I will be normalizing my results with population
in order to have a better feel between neighborhood since their population can
vary a lot.  For this I have used the nice and clean [2011
census](http://www12.statcan.gc.ca/census-recensement/2011/dp-pd/hlt-fst/pd-pl
/Table-Tableau.cfm?LANG=Eng&T=1201&SR=1&RPP=25&S=22&O=D) even available in csv
format.

In order to visualize the content, I will use pandas.  In order to know the
neighborhoods name from the postal code, I worte a little module to create a
dictionnary to map a postal code to an arrondissement name.  Here are the
modules that I will need"


    import pandas as pd
    %matplotlib inline
    import matplotlib.pyplot as plt
    import sectorstudies

###Pawn shops
Let's first turn the csv file into a data frame


    dfpawn = pd.read_csv('pawnmtl.csv')
    dfpawn.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>streetnum</th>
      <th>streetname</th>
      <th>city</th>
      <th>province</th>
      <th>postalcode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>  1954</td>
      <td>  boul de Maisonneuve E</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H2L1Z1</td>
    </tr>
    <tr>
      <th>1</th>
      <td> 31175</td>
      <td>          rue Ontario E</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H2L1R3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>  4554</td>
      <td>       rue Jean-Talon O</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H3N1R5</td>
    </tr>
    <tr>
      <th>3</th>
      <td> 56001</td>
      <td>            av Victoria</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H3W2R9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>  6504</td>
      <td>         rue Beaubien E</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H2S1S5</td>
    </tr>
  </tbody>
</table>
</div>




    dfpawn.shape
    (59, 7)



The above shows that there is only 59 pawn shops, but the presence of only few
will hopefully be enough as an indicator.  Let's add a column with the
arrondissement name using the postal code dictionary.  Lets also add a column
with the first three characters of the postal code (I will call it geoname) as
it will be use to merge with the census data later.


    cpdic = sectorstudies.getPostalCodeDic()
    dfpawn['arrondissement'] = dfpawn['postalcode'].apply(lambda p: cpdic[p[:3]])
    dfpawn['geoname'] = dfpawn['postalcode'].apply(lambda p: p[:3])
    dfpawn.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>streetnum</th>
      <th>streetname</th>
      <th>city</th>
      <th>province</th>
      <th>postalcode</th>
      <th>arrondissement</th>
      <th>geoname</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>  1954</td>
      <td>  boul de Maisonneuve E</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H2L1Z1</td>
      <td>      Centre-Sud</td>
      <td> H2L</td>
    </tr>
    <tr>
      <th>1</th>
      <td> 31175</td>
      <td>          rue Ontario E</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H2L1R3</td>
      <td>      Centre-Sud</td>
      <td> H2L</td>
    </tr>
    <tr>
      <th>2</th>
      <td>  4554</td>
      <td>       rue Jean-Talon O</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H3N1R5</td>
      <td>  Parc-Extension</td>
      <td> H3N</td>
    </tr>
    <tr>
      <th>3</th>
      <td> 56001</td>
      <td>            av Victoria</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H3W2R9</td>
      <td> Côte-des-Neiges</td>
      <td> H3W</td>
    </tr>
    <tr>
      <th>4</th>
      <td>  6504</td>
      <td>         rue Beaubien E</td>
      <td>  Montréal</td>
      <td> QC</td>
      <td> H2S1S5</td>
      <td>   Petite-Patrie</td>
      <td> H2S</td>
    </tr>
  </tbody>
</table>
</div>



I make two grouping: by geoname (finer ensembles) and by arrondissement (bigger
ensemble but more intuitive).


    dfpawnbyarr = dfpawn.groupby('arrondissement')
    dfpawnbygeo = dfpawn.groupby('geoname')
    pawnbyarr_size = dfpawnbyarr.size()
    pawnbyarr_size.sort(ascending=False)
    spawngeo = dfpawnbygeo.size()##Creates a series
    spawngeo.sort()
    dfpawngeo = pd.DataFrame(spawngeo)
    dfpawngeo.columns = ['pawnshops']

Let's look at the pawn shop distribution by arrondissement


    figpawn_byarr = plt.figure(figsize=(16, 6))
    _ = pawnbyarr_size.plot(kind='bar')


![png]({{jfraj.github.io}}/assets/neiborstudies_files/neiborstudies_12_0.png)


###Banks
Same thing for the banks below as for the pawn shops above.


    dfbank = pd.read_csv('banksmtl.csv')
    #dfbank['arrondissement'] = dfbank['postalcode'].apply(lambda p: cpdic[p[:3]])
    dfbank['arrondissement'] = dfbank['postalcode'].apply(lambda p: cpdic[p[:3]] if p[:3] in cpdic.keys()  else None)
    dfbank['geoname'] = dfbank['postalcode'].apply(lambda p: p[:3])
    dfbank.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>streetnum</th>
      <th>streetname</th>
      <th>city</th>
      <th>province</th>
      <th>postalcode</th>
      <th>arrondissement</th>
      <th>geoname</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> 12208</td>
      <td>     boul René-Lévesque O</td>
      <td>        Montréal</td>
      <td> QC</td>
      <td> H3H1R6</td>
      <td> Downtown Montreal</td>
      <td> H3H</td>
    </tr>
    <tr>
      <th>1</th>
      <td> 27-50</td>
      <td>          boul Crémazie O</td>
      <td>        Montréal</td>
      <td> QC</td>
      <td> H2P1A2</td>
      <td>          Villeray</td>
      <td> H2P</td>
    </tr>
    <tr>
      <th>2</th>
      <td>  7031</td>
      <td>  ch de la cote-Saint-Luc</td>
      <td>  Cote Saint-Luc</td>
      <td> QC</td>
      <td> H4V1J2</td>
      <td>    Côte Saint-Luc</td>
      <td> H4V</td>
    </tr>
    <tr>
      <th>3</th>
      <td>  8130</td>
      <td>           boul Champlain</td>
      <td>         Lasalle</td>
      <td> QC</td>
      <td> H8P1B4</td>
      <td>           LaSalle</td>
      <td> H8P</td>
    </tr>
    <tr>
      <th>4</th>
      <td>    26</td>
      <td>         av Westminster N</td>
      <td>  Montréal-Ouest</td>
      <td> QC</td>
      <td> H4X1Z2</td>
      <td>     Montreal West</td>
      <td> H4X</td>
    </tr>
  </tbody>
</table>
</div>




    dfbankbyarr = dfbank.groupby('arrondissement')
    dfbankbygeo = dfbank.groupby('geoname')
    sbankgeo = dfbankbygeo.size()##Creates a series
    sbankgeo.sort()
    bankbyarr_size = dfbankbyarr.size()
    bankbyarr_size.sort(ascending=False)
    dfbankgeo = pd.DataFrame(sbankgeo)
    dfbankgeo.columns = ['banks']


    figbank_byarr = plt.figure(figsize=(16, 6))
    _ = bankbyarr_size.plot(kind='bar')


![png]({{jfraj.github.io}}/assets/neiborstudies_files/neiborstudies_16_0.png)


###Census
Since the census data already come as a csv, there is not much to do to have it
in a nice format.  Let's change the column name "Geographic name" (first three
characters of the postal code) into "geoname" to be consistent with the above
data frame.  And change "Population, 2011" to population2011 and set the index
to geoname for convenience.


    dfpop = pd.read_csv('census.csv')
    dfpop.rename(columns={'Geographic name':'geoname'}, inplace=True)
    dfpop.rename(columns={'Population, 2011':'population2011'}, inplace=True)
    dfpop = dfpop.set_index(dfpop['geoname'])
    dfpop.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geoname</th>
      <th>Incompletely enumerated Indian reserves and Indian settlements, 2011</th>
      <th>population2011</th>
      <th>Total private dwellings, 2011</th>
      <th>Private dwellings occupied by usual residents, 2011</th>
    </tr>
    <tr>
      <th>geoname</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Canada</th>
      <td> Canada</td>
      <td>   T</td>
      <td> 33476688</td>
      <td> 14569633</td>
      <td> 13320614</td>
    </tr>
    <tr>
      <th>A0A</th>
      <td>    A0A</td>
      <td> NaN</td>
      <td>    46297</td>
      <td>    23950</td>
      <td>    18701</td>
    </tr>
    <tr>
      <th>A0B</th>
      <td>    A0B</td>
      <td> NaN</td>
      <td>    20985</td>
      <td>    12585</td>
      <td>     8854</td>
    </tr>
    <tr>
      <th>A0C</th>
      <td>    A0C</td>
      <td> NaN</td>
      <td>    12834</td>
      <td>     8272</td>
      <td>     5482</td>
    </tr>
    <tr>
      <th>A0E</th>
      <td>    A0E</td>
      <td> NaN</td>
      <td>    23384</td>
      <td>    12733</td>
      <td>     9659</td>
    </tr>
  </tbody>
</table>
</div>



##Merging the data sets
I put everything in the same data frame to make the comparison of the pawn shop
and bank distributions.  I create two new columns, "bankdensity" and
"pawnshopdensity", so the comparison between arondissement is not biased by the
population.


    df = pd.concat([dfpop, dfpawngeo, dfbankgeo], axis=1,join='outer')
    df['arrondissement'] = df['geoname'].apply(lambda p: cpdic[p] if p in cpdic.keys()  else None)
    dfbyarr = df.groupby('arrondissement')
    dfarrsum = dfbyarr.sum()
    dfarrsum = dfarrsum[dfarrsum.banks>0]
    dfarrsum['bankdensity'] = dfarrsum['banks']/dfarrsum['population2011']
    dfarrsum['pawnshopdensity'] = dfarrsum['pawnshops']/dfarrsum['population2011']
    dfarrsum = dfarrsum.sort('bankdensity')

##Comparing pawn shop distributions and banks
Let's now compare the distribution of the bank and the pawnshop density.


    _ = dfarrsum[['bankdensity', 'pawnshopdensity']].plot(kind='bar', figsize=(16,6), ylim=[0.000001, 0.0005], 
                                                          color=['b', 'r'], label = ['Bank density', 'Pawn shop density'])
    _ = plt.legend(loc='best')


![png]({{jfraj.github.io}}/assets/neiborstudies_files/neiborstudies_22_1.png)


Clearly, there are more pawn shops when there are fewer banks.  The nature of
the relationship is not clear and can be investigated later.  But we can say
that the above plot shows that pawn shops tend to be away from the banks.


    
