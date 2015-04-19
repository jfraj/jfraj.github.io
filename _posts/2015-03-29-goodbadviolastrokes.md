---
layout: post
title: Classifying pro vs mediocre viola players
---


Musicians and experienced music lovers can detect good and bad players very quickly. It is as if they have acces to hidden channels while the rest of us are deaf to finess. I sometimes feel like there is this rich and colorful world behind the music that is out of my reach like the pot-of-gold at the end of a rainbow, a frustration only akin to the [eye-killing autostereograms](https://www.youtube.com/watch?v=ArWY-Ck-CPc).  A reasonable way to fix this handicap would be to train myself by listening to more music, who knows, this could be an edifying experience.  But maybe a computer can do it for me.
Here I try to use unsupervised clustering to distinguish a professional viola player ([Marina](http://www.marinathibeaultviola.com)) from a mediocre one (me). For simplicity, the comparison is performed with only one type of stroke on the viola's highest pitch string (A).

A lot of the code below was inspired (sometimes copy-pasted) from Steve Tjoa's [Music Information Retrieval workshop tutorial](https://github.com/stevetjoa/stanford-mir).

##Tools
To manipulate the audio files, I will use the Essentia module with the usual matplotlib and numpy.


    ## Numeric manipulation and plotting
    %matplotlib inline
    import matplotlib.pyplot as plt
    import numpy as np
    
    ## Audio information retrieval
    import essentia.standard as ess
    from essentia.standard import MonoLoader
    from essentia.standard import Centroid, Spectrum, Windowing
    
    ## Classifier and feature processing
    from sklearn import preprocessing
    from sklearn.cluster import KMeans

##Data
I recorded five strokes of a professional and mediocer player with my laptop's microphone and quicktime player which produce MP4 files by default.


    marina = MonoLoader(filename='/Users/jean-francoisrajotte/myaudio/marina.m4a')()
    marina = marina[50000:-10000]##Cleaning useless head and tail of the file
    jfraj = MonoLoader(filename='/Users/jean-francoisrajotte/myaudio/jfraj.m4a')()
    jfraj = jfraj[70000:-400000]

by default the sampling frequency is 44100 Hz so to show the sampling by seconds, I divide by 44100


    fig_sig = plt.figure(figsize=[8,5])
    
    N_marina = len(marina)
    t_marina = np.arange(0, N_marina)/44100.0
    ax1 = plt.subplot(2,1,1)
    plt.plot(t_marina, marina)
    ax1.text(7.7, 0.5, 'Marina', fontsize=15)
    
    
    N_jf = len(jfraj)
    t_jf = np.arange(0, N_jf)/44100.0
    ax2 = plt.subplot(2,1,2)
    plt.plot(t_jf, jfraj)
    ax2.text(7, 0.5, 'jfraj', fontsize=15)
    _ = plt.xlabel('Time (seconds)')


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_8_0.png)


The above figure shows the audio signal (air pressure) as function of time.
The mediocrity of jfraj's signal is already obvious: less smooth and less consistent.
Let's see if there is a way to categorize good and bad.

##Stroke sampling
To caracterize each stroke, the signal must be isolated.
Let's first combine both audio signals so we can extract the strokes from a single object.


    fig_all = plt.figure(figsize=[8,3])
    all_sig = np.append(jfraj, marina)
    N_all = len(all_sig)
    t_all = np.arange(0, N_all)/44100.0
    plt.plot(t_all, all_sig)
    _ = plt.xlabel('Time (seconds)')


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_11_0.png)


The first five strokes are jfraj's and the other five are Marina's.
To isolate the strokes, I use essentia's function ess.OnsetRate(), one of its output is an array of onsets.  The onsets are defined as transient region in the signal (cf. [this tutorial](https://files.nyu.edu/jb2843/public/Publications_files/2005_BelloEtAl_IEEE_TSALP.pdf)).


    get_onsets = ess.OnsetRate()
    onset_times, onset_rate = get_onsets(all_sig)

I now have the onset times (in seconds), but I want the onset samples (indexes in the array), 
to do this, I convert the time with the sampling rates 44100 Hz.


    onset_samples = [int(44100*i) for i in onset_times]

Let's see if it works by plotting the onsets on top of the signal.


    fig_onset = plt.figure(figsize=[8,3])
    plt.plot(all_sig)
    for isamp in onset_samples:
        plt.axvline(isamp, color='r')


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_17_0.png)


Almost.  Some false onsets were detected by a variation within the stroke.  Let's remove them by hand.


    toremove = [1, 3, 10, 13]##ith onset to remove starting from 0
    onset_samples = np.array([element for i, element in enumerate(onset_samples) if i not in toremove])
    onset_times = np.array([element for i, element in enumerate(onset_times) if i not in toremove])
    ## plot the resulting onsets
    fig_onset_fixed = plt.figure(figsize=[8,3])
    plt.plot(all_sig)
    for isamp in onset_samples:
        plt.axvline(isamp, color='r')


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_19_0.png)


Now that I know when the strokes start, the stroke signals can be extracted by selecting, say, 0.5 seconds starting from the onset.


    frame_sz = int(0.500*44100)
    segments = np.array([all_sig[i:i+frame_sz] for i in onset_samples])

Here are the individual strokes the five above are jfraj's and bottom are Marina's.


    fig = plt.figure(figsize=[15,8])
    for iseg in range(segments.shape[0]):
        plt.subplot(2,5,iseg+1)
        plt.plot(segments[iseg])


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_23_0.png)


##Extracting features
I define below a function that extracts two features that I (more or less) randomly selected.
The first one is the [zero-crossing rate](http://en.wikipedia.org/wiki/Zero-crossing_rate) and the second one is the [spectral centroid](http://en.wikipedia.org/wiki/Spectral_centroid).
The function defined below returns those values for a given frame.


    centroid = Centroid(range=22050)
    hamming_window = Windowing(type='hamming')
    zcr = ess.ZeroCrossingRate()
    spectrum = ess.Spectrum()
    def extract_features_from_audio(frame):
        spectral_magnitude = spectrum(hamming_window(frame))
        feature_vector = [zcr(frame), centroid(spectral_magnitude)]
        return feature_vector

The features are then determined for each strokes (segments)


    feature_table = np.array([extract_features_from_audio(segment) for segment in segments])

It is always safer to scale the features if they are intended to be used with a classifier.
Scikit-learn's MinMaxScaler from "preprocessing" will be used to scale in the [-1,1] interval.


    min_max_scaler = preprocessing.MinMaxScaler(feature_range=(-1, 1))
    features_scaled = min_max_scaler.fit_transform(feature_table)

I can now look at the strokes for the chosen features


    plt.scatter(features_scaled[:, 0], features_scaled[:, 1], c='b',s=70)
    plt.xlabel('zero-crossing (scaled)')
    _ = plt.ylabel('spectral centroid (scaled)')


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_31_0.png)


##Clustering
The above plot already suggest that the sample contains two clusters distinguisable by eye.   Let's see how the [k-Means](http://en.wikipedia.org/wiki/K-means_clustering) classifier does with it.


    est = KMeans(2)  # 2 clusters for two players
    est.fit(features_scaled)
    y_kmeans = est.predict(features_scaled)
    plt.scatter(features_scaled[:, 0][y_kmeans==0], features_scaled[:, 1][y_kmeans==0], c='r', s=70)
    plt.scatter(features_scaled[:, 0][y_kmeans==1], features_scaled[:, 1][y_kmeans==1], c='g', s=70)
    plt.xlabel('zero-crossing rate (scaled)')
    _ = plt.ylabel('spectral centroid (scaled)')


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_33_0.png)


Good!  just like the eye would expect.  But which strokes are these clusters associated to?  The way the features were defined was with jfraj's stroke for the first five and the last ones from Marina's.


    print y_kmeans

    [1 1 1 1 1 0 0 0 0 0]


Indeed, the clustering fits the player!

##A closer look at the zero-crossing rate
As a final check, let's see how the first feature, zero-crossing, helped us to classify the players. 
As the name suggests, zero-crossing happens when the signal changes sign so it should be possible to see by zooming on the signal.  Let's plot a signal from jfraj and Marina and insert a zoom on a 20-millisecond window to have a clear view of the zero-crossing.


    bad_seg = segments[1]
    good_seg = segments[7]
    t_xaxis = np.arange(0, len(bad_seg))/44100.0
    sub_range = [7000, 7882]
    zoom_interval = (sub_range[1]-sub_range[0])*1000/44100.0##in milliseconds
    fig = plt.figure(figsize=[15,5])
    
    ax_bad = plt.subplot(1,2,1)
    plt.plot(t_xaxis, bad_seg)
    plt.text(0.75, 0.06, 'jfraj', fontsize=16, transform=ax_bad.transAxes)
    plt.axis([0, len(bad_seg)/44100.0, 1.1*np.amin(bad_seg), 3*np.amax(bad_seg) ])
    plt.xlabel('Time (seconds)')
    zoom_bad = plt.axes([.2, .57, .25, .27], axisbg='y')
    plt.plot(bad_seg[sub_range[0]:sub_range[1]])
    plt.setp(zoom_bad, xticks=[], yticks=[])
    plt.text(0.45, 0.01, '{}ms'.format(int(zoom_interval)), fontsize=13, transform=zoom_bad.transAxes)
    
    
    ax_good = plt.subplot(1,2,2)
    plt.plot(t_xaxis, good_seg)
    plt.axis([0, len(good_seg)/44100.0, 1.1*np.amin(good_seg), 3*np.amax(good_seg) ])
    plt.xlabel('Time (seconds)')
    zoom_good = plt.axes([.62, .57, .25, .27], axisbg='y')
    plt.plot(good_seg[sub_range[0]:sub_range[1]])
    plt.setp(zoom_good, xticks=[], yticks=[])
    plt.text(0.45, 0.01, '{}ms'.format(int(zoom_interval)), fontsize=13, transform=zoom_good.transAxes)
    _ = plt.text(0.75, 0.06, 'Marina', fontsize=16, transform=ax_good.transAxes)


![png]({{jfraj.github.io}}/assets/goodbadviolastrokes_files/goodbadviolastrokes_38_0.png)


It's now obvious that the professional signal from Marina changes sign more frequently.
This is a result of many years of practice to generate a stroke creating [overtones](http://en.wikipedia.org/wiki/Overtone) with neighbor strings, the instrument's body and bow.
