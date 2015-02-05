---
layout: post
title: List reduction with Pandas data frame
---


I recently had to deal with a data frame containing strings as space-separated-
value lists of variable length.  Here is an example:


    from pandas import DataFrame
    datadic = {'c1': ['1 2 3 4', '5 6 7', '8 9 10 11 12 13 14 15', '16 17 18 19 20']}
    df = DataFrame(datadic)
    print df

                          c1
    0                1 2 3 4
    1                  5 6 7
    2  8 9 10 11 12 13 14 15
    3         16 17 18 19 20


My first take was to turn them into a list with a split:


    df['c1AsList'] = df['c1'].apply(lambda s: s.split())
    print df

                          c1                        c1AsList
    0                1 2 3 4                    [1, 2, 3, 4]
    1                  5 6 7                       [5, 6, 7]
    2  8 9 10 11 12 13 14 15  [8, 9, 10, 11, 12, 13, 14, 15]
    3         16 17 18 19 20            [16, 17, 18, 19, 20]


Or I might as well make it a list of float


    df['c1AsListOfFloats'] = df['c1'].apply(lambda s: map(float, s.split()))
    print df

                          c1                        c1AsList  \
    0                1 2 3 4                    [1, 2, 3, 4]   
    1                  5 6 7                       [5, 6, 7]   
    2  8 9 10 11 12 13 14 15  [8, 9, 10, 11, 12, 13, 14, 15]   
    3         16 17 18 19 20            [16, 17, 18, 19, 20]   
    
                                     c1AsListOfFloats  
    0                            [1.0, 2.0, 3.0, 4.0]  
    1                                 [5.0, 6.0, 7.0]  
    2  [8.0, 9.0, 10.0, 11.0, 12.0, 13.0, 14.0, 15.0]  
    3                  [16.0, 17.0, 18.0, 19.0, 20.0]  


##Data reduction
Now let's say that for some (machine learning) purposes, you like to have one
value per column.  If I agree to lose some information, I could transform the
list with three values:
* Mean
* Variation
* Number of values

I kept Variation vague but two examples would be the standard deviation and the
range.  Small number of values as in the example above, I think the range (max -
min) is more appropriate.  Let's do it using a numpy array instead of a list of
float:


    import numpy as N
    df = DataFrame(datadic)
    df['mean'] = df['c1'].apply(lambda s : N.array(map(float, s.split())).mean())
    df['range'] = df['c1'].apply(lambda s : N.array(map(float, s.split())).ptp(axis=0))
    df['nval'] = df['c1'].apply(lambda s : len(s.split()))
    print df

                          c1  mean  range  nval
    0                1 2 3 4   2.5      3     4
    1                  5 6 7   6.0      2     3
    2  8 9 10 11 12 13 14 15  11.5      7     8
    3         16 17 18 19 20  18.0      4     5


Or this could be done by one function


    def reduceList(x):
        xarray = N.array(map(float, x.split()))
        return xarray.mean(), xarray.ptp(axis=0),len(xarray)


    df = DataFrame(datadic)
    df['mean'], df['range'], df['nval'] = zip(*df['c1'].apply(reduceList))
    print df

                          c1  mean  range  nval
    0                1 2 3 4   2.5      3     4
    1                  5 6 7   6.0      2     3
    2  8 9 10 11 12 13 14 15  11.5      7     8
    3         16 17 18 19 20  18.0      4     5



    
