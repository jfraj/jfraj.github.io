---
layout: post
title: Slip out of your anaconda's grip with virtualenv
---

Trying to install a python library, I found out that my python distribution was not supported.  I did not want to change all my global environment (and everything that might depend on it) just for this library.  This is a classic case for [virtualenv](https://pypi.python.org/pypi/virtualenv).  There is a easy-to-read [blog entry](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/) about virtualenv, here I show how I used it to reconcile the [anaconda python distribution](https://store.continuum.io/cshop/anaconda/) and the nice music information retrieval library [essentia](http://essentia.upf.edu).

##Taming the anaconda
note: this is not necessary but it helps to have environment separated.

The first thing I did was to have my anaconda distribution on-demand (as suggested [here](http://stackoverflow.com/questions/20364700/using-two-different-python-distributions)), 
by removing the anaconda path to my 'PATH' in my bash_profile and add the following line

`alias anacondainit='export PATH="[my-path-to]/anaconda/bin:$PATH"'`

The distribution is now only available after typing `anacondainit`.

##Making a hospitable environment for essentia
Assuming virtualenv is installed (`pip install virtualenv`), the environment can be created in the directory of the project

{% highlight bash %}
virtualenv env
source env/bin/activate
{% endhighlight %}

The last line is to spare you writing the path to your environment, e.g. `python` instead of `[path to your env]/python'`.

##Install
Everything is now ready to follow the [installation proceedure](http://essentia.upf.edu/documentation/installing.html).  Since the python is now isolated, many libraries might need to be install (after `source env/bin/activate`)


{% highlight bash %}
pip install ipython numpy
pip install matplotlib
{% endhighlight %}

###Configure warning: prefix may be needed
In order to override the default path `/usr/bin/local/`, the configure might need an explicite path to the environment's python
```
./waf configure --mode=release --with-python --prefix=[path to your]/env
```

then the usual

{% highlight bash %}
./waf
./waf install
{% endhighlight %}

##Quick test
Let's check that everything works fine with some code stolen from this [tutorial](http://nbviewer.ipython.org/github/stevetjoa/stanford-mir/blob/master/notebooks/kmeans.ipynb)


    #Some basic header
    %matplotlib inline
    import matplotlib.pyplot as plt
    import essentia.standard as ess
    import numpy as np


    ###Get some basic audio file


    from urllib import urlretrieve
    urlretrieve('https://ccrma.stanford.edu/workshops/mir2014/audio/simpleLoop.wav', filename='simpleLoop.wav')
    simple_loop = ess.MonoLoader(filename='simpleLoop.wav')()


    _ = plt.plot(simple_loop)


![png]({{jfraj.github.io}}/assets/essentia_with_virtualenv_files/essentia_with_virtualenv_9_0.png)


Good enough, things seem to work, we are ready to play with audio files.
