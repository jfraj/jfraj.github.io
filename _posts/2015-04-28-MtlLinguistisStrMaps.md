---
layout: post
title: Montreal linguistic street map
---


Last [PyCon](http://us.pycon.org/2015/), I assisted to a [nice talk](https://www.youtube.com/watch?v=MIFOTFdtK2k) by Michelle Fullwood about classifying streetnames in Singapore by linguistic origin.
It made me think that Montreal's multilingual character would be a good candidate for such a study.  I was not the only one to have this reflection and the project was even suggested at last weekends [Montreal Big Data Week Hackathon](http://mtlbdw15.splashthat.com/).  I teamed up with [Philippe](http://www.lesca.ca/2014/03/18/philippe-brouillard-2/), whom I kaggle with.
Here is how we classified and visualized street names by language.

##Data
[Mazen](https://mapzen.com/metro-extracts/) provide city-sized portions of [OpenStreetMap](http://www.openstreetmap.org) of many world's metro area, including Montreal.
We used the `IMPOSM GEOJSON` format which can easily be manipulated with [geopandas](https://github.com/geopandas/geopandas).  Here are the settings and modules that we used:


    import sys  
    reload(sys)  
    sys.setdefaultencoding('utf8')#to take care of french accents
    import geopandas as gpd
    %matplotlib inline
    import matplotlib.pyplot as plt
    from matplotlib.colors import LinearSegmentedColormap

Montreal's streets as a (geo)DataFrame


    mtldata = '../data/montreal_canada/montreal_canada-roads.geojson'
    mtl = gpd.read_file(mtldata)
    mtl.shape

    (112215, 13)



The last line above tells us that the sample contains about 100k roads caracterized by 13 parameters.  Let's look at a few examples.


    mtl.head(n=2)




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>access</th>
      <th>bridge</th>
      <th>class</th>
      <th>geometry</th>
      <th>id</th>
      <th>name</th>
      <th>oneway</th>
      <th>osm_id</th>
      <th>ref</th>
      <th>service</th>
      <th>tunnel</th>
      <th>type</th>
      <th>z_order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> None</td>
      <td> 0</td>
      <td> highway</td>
      <td> LINESTRING (-73.82927614556513 45.439216892268...</td>
      <td> 1</td>
      <td>         Autoroute 20</td>
      <td> 1</td>
      <td> 3453107</td>
      <td> A 20</td>
      <td> None</td>
      <td> 0</td>
      <td>  motorway</td>
      <td> 9</td>
    </tr>
    <tr>
      <th>1</th>
      <td> None</td>
      <td> 0</td>
      <td> highway</td>
      <td> LINESTRING (-73.8529825146727 45.4936175393405...</td>
      <td> 2</td>
      <td> Boulevard Saint-Jean</td>
      <td> 1</td>
      <td> 3453106</td>
      <td> None</td>
      <td> None</td>
      <td> 0</td>
      <td> secondary</td>
      <td> 5</td>
    </tr>
  </tbody>
</table>
</div>



geopandas has turned the geojson into a dataframe from which we are interested in two columns:
* name, for linguistic classification
* geometry, for plotting

plotting with geopandas is simple, here is how we plot the first 5000 streets:


    mtl.head(n=5000).plot()
    plt.xlabel('longitude')
    _ = plt.ylabel('lalitude')


![png]({{jfraj.github.io}}/assets/MtlLinguistisStrMaps_files/MtlLinguistisStrMaps_8_0.png)


##Region selection
The longitude/latitude range of the above figure reveales that the covered area is much more than Montreal.
This is better seen by plotting the limits of the administrative regions available in same downloaded data under the name `montreal_canada-admin.geojson`.
It is again easy to acces them with geopandas.


    admin = gpd.read_file('../data/montreal_canada/montreal_canada-admin.geojson')
    admin.head()




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>admin_leve</th>
      <th>geometry</th>
      <th>id</th>
      <th>name</th>
      <th>osm_id</th>
      <th>type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> 8</td>
      <td> POLYGON ((-73.46162801892102 44.99203598771518...</td>
      <td> 1</td>
      <td>    Champlain</td>
      <td>-176177</td>
      <td> administrative</td>
    </tr>
    <tr>
      <th>1</th>
      <td> 8</td>
      <td> POLYGON ((-73.37881707868057 44.98191098395773...</td>
      <td> 2</td>
      <td> Rouses Point</td>
      <td>-176180</td>
      <td> administrative</td>
    </tr>
    <tr>
      <th>2</th>
      <td> 8</td>
      <td> POLYGON ((-73.68554637124987 45.48971995436481...</td>
      <td> 3</td>
      <td>   Mont-Royal</td>
      <td>-197567</td>
      <td> administrative</td>
    </tr>
    <tr>
      <th>3</th>
      <td> 8</td>
      <td> POLYGON ((-73.65601701018861 45.48295626141797...</td>
      <td> 4</td>
      <td>    Hampstead</td>
      <td>-197569</td>
      <td> administrative</td>
    </tr>
    <tr>
      <th>4</th>
      <td> 8</td>
      <td> POLYGON ((-73.90550402287215 45.44475615679986...</td>
      <td> 5</td>
      <td>     Kirkland</td>
      <td>-197577</td>
      <td> administrative</td>
    </tr>
  </tbody>
</table>
</div>



Plotting the polygone is just as easy


    _ = admin.plot()


![png]({{jfraj.github.io}}/assets/MtlLinguistisStrMaps_files/MtlLinguistisStrMaps_12_0.png)


We only want streets on Montreal's island which is named *Montréal (06)*.
The selection of streets within the shape is straight forward


    mtl_index = admin[admin['name']=='Montréal (06)'].index.tolist()[0]


    fig = plt.figure(figsize=(10,10))
    mtl_shp = admin.geometry[mtl_index]
    in_mtl06 = mtl.geometry.within(mtl_shp)
    admin[admin.index==mtl_index].plot(colormap='Greys')
    _=mtl[in_mtl06].head(n=5000).plot()


![png]({{jfraj.github.io}}/assets/MtlLinguistisStrMaps_files/MtlLinguistisStrMaps_15_0.png)


##Language classification
Now that we have the streets, let's try to see their distributions based on their language.
Classification is performed via a score of [bigram and trigram](http://en.wikipedia.org/wiki/N-gram) linguistic frequencies.
Frequencies were determined by training on 1000 english and french surnames. 
For example, the most frequent french trigrams are *cha*, *eau* and *ier*.
The more french-frequent n-grams contained by a given word, the higher positive score it will get.
Similarly, an english-frequent n-gram with score high negatively.
The code can be find in the function `whichLanguage.py` from [this github](https://github.com/jfraj/street_lang).
In this repository, we have also created a class that contains the (geo) data frame which can calculate the linguistic score for each street.  Here is how to use it


    sys.path.append("/Users/jfr/projects/street_lang")
    import street_languages


    mtl_streets = street_languages.City(mtldata, 'Montreal')
    mtl_streets.df = mtl_streets.df[mtl_streets.df.geometry.within(mtl_shp)]
    mtl_streets.set_language_stats()

The (geo) dataframe in the object mtl_streets now has a new column named `lang` which contains the linguistic score.  The classification can now be done with `mtl_streets.df['lang']>0` for french and `<0` for english.

##Visualization
Everything is now at hand to look at the linguistic distribution of the streets.
First we create simple color maps for clear contrasts between french and english streets.


    cdict1 = {'red':   ((0.0, 0.0, 0.0),(1.0, 0.0, 0.0)),
             'green': ((0.0, 0.0, 0.0),(1.0, 0.0, 0.0)),
             'blue':  ((0.0, 0.0, 1.0),(1.0, 0.0, 0.0))
            }
    cdict2 = {'red':   ((0.0, 0.0, 1.0),(1.0, 0.0, 0.0)),
             'green': ((0.0, 0.0, 0.0),(1.0, 0.0, 0.0)),
             'blue':  ((0.0, 0.0, 0.0),(1.0, 0.0, 0.0))
            }
    
    blues = LinearSegmentedColormap('BlueRed2', cdict1)
    reds = LinearSegmentedColormap('BlueRed2', cdict2)

Now we plot the streets with different color maps based on the linguistic score.


    ax = mtl_streets.df[mtl_streets.df['lang']>0].plot(colormap=blues, alpha=0.5)
    mtl_streets.df[mtl_streets.df['lang']<0].plot(colormap=reds, alpha=0.5)
    fig = ax.get_figure()
    fig.set_size_inches(12,12)
    ax.text(-73.9, 45.7, 'Montreal street names',color='k', fontsize=25)
    ax.text(-73.8, 45.67, 'French',color='b', fontsize=20)
    ax.text(-73.8, 45.65, 'English',color='r', fontsize=20)
    _ = plt.axis('off')


![png]({{jfraj.github.io}}/assets/MtlLinguistisStrMaps_files/MtlLinguistisStrMaps_23_0.png)


Here is the good old west-english & east-french trend.
Anyone who have lived in Montreal for some times knows that this is also a trend for the spoken language (see, for example, this [blog post](http://www.mtlblog.com/2014/12/this-is-where-people-speak-french-or-english-in-montreal/)).
This is not the complete story, the above figure assumes only english and french, but there is obviously more than two languages in Montreal's street names, e.g. names of the Iroquoian family language.
