---
layout: post
title: "First look at hackingCBCMTL data"
---

This weekend I will be attending
[#HackingCBCMTL](http://www.cbc.ca/news/canada/montreal/how-can-cbc-montreal-
use-tech-to-help-cover-the-news-1.2987416) and they have just released the [data
](https://docs.google.com/spreadsheets/d/1FEY0kokq_awO0H3mzEt5v5JLlmZzaMyPA7BiNw
murqM/edit#gid=0).  Here is a first look a the CBC DATA set, "machine extracted
summaries of stories" published at [CBC](http://cbc.ca/news/canada/montreal) in
2014.

I start with the usual tools import and settings


    %matplotlib inline
    import pandas as pd
    import matplotlib.pyplot as plt
    pd.set_option('display.mpl_style', 'default')
    plt.rcParams['figure.figsize'] = (15, 5)
    plt.rc('ytick', labelsize=20) 
    import sys
    import json

The data is in JSON format, easy to play with python and pandas.


    json_fname = '../data/stories-montreal-2014-01-01-to-2015-03-09.json'
    db = json.load(open(json_fname))
    len(db)




    3034



Each entry is a news, 3034 published last year about 8 stories per day.

##News global feature
Let's see what information we have about each news:


    db[0].keys()




    [u'storyId', u'storyUrl', u'title', u'concepts']



Nice, not only do we have the some basic infomation like title and
concepts(explored below), but also the URL for further scraping if needed.  Here
is what the first news looks like:


    db[0]['storyId'], db[0]['storyUrl'], db[0]['title']




    (u'1.2530186',
     u'http://www.cbc.ca/news/canada/montreal/mega-moving-probed-by-police-after-couple-s-belongings-disappear-1.2530186',
     u"Mega Moving probed by police after couple's belongings disappear")



Ok, so the *storyId* format with a point (maybe because of some CBC convention)
will be interpreted as a float. It's probably not a problem unless there are
many more digits after the point.
Moving on, the [URL is valid](http://www.cbc.ca/news/canada/montreal/mega-
moving-probed-by-police-after-couple-s-belongings-disappear-1.2530186) and the
title is right.  So far so good.  I excluded the last element, *concept*,
because it contains multiple items, here are the first 2 in JSON format:


    db[0]['concepts'][:2]




    [{u'groups': [{u'count': 10,
        u'group': u'concept',
        u'weight': 0.141631171057803}],
      u'item': u'couple'},
     {u'groups': [{u'count': 1,
        u'group': u'category',
        u'weight': 0.0635425122097419}],
      u'item': u'montreal'}]



##Creating the data frame
In order to deal with this, I decided to make a long list separated for each
items (as data frame) and concatenate into a bigger data frame.  This is not the
most memory efficient (lots of repeated information), but it is a luxury we can
afford with such a small amount of data.  The information "by news" can be
recovered with grouping by *storyId*.


    news_items = []
    total_news = len(db)
    for counter, inews in enumerate(db):
        for iconcepts in inews['concepts']:
            inews_item = pd.DataFrame(iconcepts['groups'])
            inews_item['storyId'] = inews['storyId']
            inews_item['item'] = iconcepts['item']
            news_items.append(inews_item)
        if counter%100==0:
            print 'row {} ({}%)'.format(counter, 100*counter/total_news)
        #if counter >100:
        #    break
    news_items = pd.concat(news_items, ignore_index=True)



How much bigger is the sample now?


    news_items.shape




    (118595, 5)



ok, that's acceptable, here are few items


    news_items.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>count</th>
      <th>group</th>
      <th>weight</th>
      <th>storyId</th>
      <th>item</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> 10</td>
      <td>  concept</td>
      <td> 0.141631</td>
      <td> 1.2530186</td>
      <td>               couple</td>
    </tr>
    <tr>
      <th>1</th>
      <td>  1</td>
      <td> category</td>
      <td> 0.063543</td>
      <td> 1.2530186</td>
      <td>             montreal</td>
    </tr>
    <tr>
      <th>2</th>
      <td>  1</td>
      <td> taxonomy</td>
      <td> 0.065894</td>
      <td> 1.2530186</td>
      <td> news/canada/montreal</td>
    </tr>
    <tr>
      <th>3</th>
      <td>  1</td>
      <td>  concept</td>
      <td> 0.085446</td>
      <td> 1.2530186</td>
      <td>       summer clothes</td>
    </tr>
    <tr>
      <th>4</th>
      <td> 12</td>
      <td>  concept</td>
      <td> 0.580728</td>
      <td> 1.2530186</td>
      <td>           belongings</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 5 columns</p>
</div>



Count is not explained on the page, but I guess (and checked for *couple and
belongings*) it is the number of time the word is found on the page (including
the URL which is a bit of a cheat since it it made from the title... on the
other hand, if it's the title it's ok if it count double).  Weight is explained
as "reflecting the prominence in the page".  The following figure shows that the
weight is between 0 and 1:


    news_items['weight'].hist(bins=30)
    plt.title('weight')
    _ = plt.ylabel('number of items')


![png]{{jfraj.github.io}}/assets/hackingcbcmtl_files/data_first_look_18_0.png)


##Groups
The group defines the different classifications of the items:


    print news_items['group'].unique()

    [u'concept' u'category' u'taxonomy' u'entity' u'author' u'location'
     u'person' u'language' u'site' u'pageclass' u'cbc-categories' u'company'
     u'acronym']


Some groups are easy to guess, like *person* and *location* but others need to
be checked.  Looking at few examples should clear this up:

###Group: concept


    print news_items[news_items['group']=='concept']['item'].unique()[:10]

    [u'couple' u'summer clothes' u'belongings' u'cbc news' u'e-mail exchange'
     u'birth certificates' u'family photos' u'moving' u'lift chair'
     u'couple time']


Ok, that pretty wide, but that could be interesting to investigate.

###Group: category


    print news_items[news_items['group']=='category']['item'].unique()

    [u'montreal' u'canada' u'news' u'quebec-votes-2014']


Only four, and two of them are location...

###Group: taxonomy


    print news_items[news_items['group']=='taxonomy']['item'].unique()

    [u'news/canada/montreal' u'news' u'news/canada'
     u'news/canada/montreal/quebec-votes-2014']


Again very few and similar to category.

###Group: entity


    print news_items[news_items['group']=='entity']['item'].unique()[:10]

    [u'montreal investigates' u'canadian association' u'lagues'
     u'mega moving and storage' u'fehmi yasar'
     u'canadian association of movers' u'better business bureau' u'news'
     u'liesse road' u'c\xf4te de liesse road']


This can be useful.  We see that we'll have to deal with the accent eventually.

###Group: author


    print news_items[news_items['group']=='author']['item'].unique()[:10]

    [u'cbc news']


Too bad, the real author names would have been fun to play with.

###Group: location


    print news_items[news_items['group']=='location']['item'].unique()[:10]

    [u'ontario' u'longueuil' u'nova scotia' u'chatham' u'welland' u'montreal'
     u'quebec' u'liberal' u'canada' u'saudi arabia']


###Group: person


    print news_items[news_items['group']=='person']['item'].unique()[:10]

    [u'kim lague' u'jim carney' u'david lague' u'al lafrance' u'britt dash'
     u'matt goldberg' u'pauline marois' u'philippe couillard' u'simon delorme'
     u'john baird']


###Group: language


    print news_items[news_items['group']=='language']['item'].unique()

    [u'en']


###Group: site


    print news_items[news_items['group']=='site']['item'].unique()

    [u'cbc.ca']


###Group: pageclass


    print news_items[news_items['group']=='pageclass']['item'].unique()

    [u'article' u'frontpage']


###Group: cbc-categories


    print news_items[news_items['group']=='cbc-categories']['item'].unique()[:10]

    [u'business' u'community' u'community/politics' u'news/national' u'news'
     u'canpoli' u'lifestyle' u'news/local' u'canpoli/riding' u'community/law']


This is potentially interesting as it might be hand-made categories less
polluted by machine search.

###Group: company


    print news_items[news_items['group']=='company']['item'].unique()[:10]

    [u'amnesty international' u'soconex' u'google'
     u'empire life insurance company' u'magnotta' u'facebook' u'common inc'
     u'cambridge inc' u'champlain bridge corp' u'qu\xe9bec inc']


###Group: acronym


    print news_items[news_items['group']=='acronym']['item'].unique()[:10]

    [u'automobile protection association' u'montreal police'
     u'canadian food inspection agency' u'transportation safety board'
     u'agence m\xe9tropolitaine de transport'
     u'mcgill university health centre' u'there is significantly less'
     u'soci\xe9t\xe9 des alcools du qu\xe9bec'
     u'centre hospitalier r\xe9gional de trois-rivi\xe8res'
     u'transportation safety board of canada']


##Basic exploration
As a first test, it would be nice to have an idea of the important of some news
subject.

###Adding another metric: wcount
There are two parameters to see if something is important, the number of time an
item is mentionned (*count*) and the prominence in the page (*weight*).  In
order to have a single *importance* metric, the simplest way to do it is to
multiply them.


    news_items['wcount'] = news_items['weight']*news_items['count']

###Group by items
Now let's group all the news items and sum the counts


    news_byitems = news_items.groupby(['group', 'item'])

Let's first look at raw counts of the most impotant


    count_sum = news_byitems['count'].sum()
    count_sum.sort()
    count_sum['person'].tail(10).plot(kind='barh', )
    plt.title('Number of word counts')
    _ = plt.ylabel('')


![png]({{jfraj.github.io}}/assets/hackingcbcmtl_files/data_first_look_files/data_first_look_59_0.png)



    news_byitems['wcount']
    wcount_sum = news_byitems['wcount'].sum()
    wcount_sum = wcount_sum.order()
    wcount_sum['person'].tail(10).plot(kind='barh')
    plt.ylabel('')
    _ = plt.title('Importance in 2014 news')


![png]({{jfraj.github.io}}/assets/hackingcbcmtl_files/data_first_look_files/data_first_look_60_0.png)



    wcount_sum['company'].tail(10).plot(kind='barh')
    plt.ylabel('')
    _ = plt.title('Company importance in 2014 news')


![png]({{jfraj.github.io}}/assets/hackingcbcmtl_files/data_first_look_files/data_first_look_61_0.png)


Obviously, the data needs to be cleaned: *magnotta* is not a company and
*facebook* and *facebook inc* should be merged.

##Final note
This is fun, but must be taken with a grain of salt; word count -even weighted
by the prominence of the page- is can be misleading.  For example, maybe a word
must be written many time just because it needs to be explained more thoroughly
or there are many citations that refer to it (e.g. exerpt from court order).  On
the other hand, a word can be counted less because there are many synonyms or
the journalist alternate between, say, the person's title and his name.
