---
layout: post
title: In-script audio recording with pyaudio
---

I am regularly recording audio for some analysis and it was time to do better than opening quicktime each time.
What I was looking for was to do it directly in the analyzing script.  I found an easy option with [pyaudio](https://people.csail.mit.edu/hubert/pyaudio/docs/), a "Python bindings for PortAudio, the cross-platform audio I/O library".  It allows easy access to a computer's microphone.  I found in [this blog](http://sharewebegin.blogspot.ca/2013/07/record-from-mic-python.html) a simple example of use for audio recording.  Here are the steps with some explanations.

## Installing pyaudio
Following the suggestion of `pip install` error messages, installation finally worked with

`pip install --allow-external pyaudio --allow-unverified pyaudio  pyaudio`


    import pyaudio

## Interface with PortAudio
PortAudio initialization/termination and stream management is done through the api `pyaudio` class.


    p = pyaudio.PyAudio()

`p` can now be used to get audio input.  The `get_default_input_device_info()` method tells information about the audio source,


    p.get_default_input_device_info()

    {'defaultHighInputLatency': 0.01292517006802721,
     'defaultHighOutputLatency': 0.1,
     'defaultLowInputLatency': 0.002766439909297052,
     'defaultLowOutputLatency': 0.01,
     'defaultSampleRate': 44100.0,
     'hostApi': 0L,
     'index': 0L,
     'maxInputChannels': 2L,
     'maxOutputChannels': 0L,
     'name': u'Built-in Microph',
     'structVersion': 2L}



These come from `PortAudio` parameters, and some are already helpful without having to go through the documentation.  The `name: Built-in Microph` lets me know that my laptop microphone will be used and `defaultSampleRate': 44100.0` is the sample rate that I am used to.
So far so good!

##Recording
To record audio, I need to *open a stream* with the following parameters


    FRAMES_PERBUFF = 2048 # number of frames per buffer
    FORMAT = pyaudio.paInt16 # 16 bit int
    CHANNELS = 1 # I guess this is for mono sounds
    FRAME_RATE = 44100 # sample rate

In the above parameters, I had to increase the number of frame per buffer from 1024 to 2048 in order to avoid `IOError: [Errno Input overflowed]`.
The default rate is already 44100 but this variable will be needed later and I do not want to depend on the default.
I now open a stream on the microphone (default device).


    stream = p.open(format=FORMAT,
                    channels=CHANNELS,
                    rate=FRAME_RATE,
                    input=True,
                    frames_per_buffer=FRAMES_PERBUFF) #buffer

Using this interface with the microphone, I can record audio frames in a list


    frames = []

The class `pyaudio.Stream` has a `read` method which takes argument the number of frames to read.
In order not to overflow the memory, reading is done by chunks (`FRAMES_PERBUFF`).
Let say I want to record 5 seconds, then I have to read `5*FRAME_RATE` frames at a rate of 44100Hz.
These frames are buffered in chunks of `FRAMES_PERBUFF`.


    RECORD_SECONDS = 5
    nchunks = int(RECORD_SECONDS * FRAME_RATE / FRAMES_PERBUFF)
    for i in range(0, nchunks):
        data = stream.read(FRAMES_PERBUFF)
        frames.append(data) # 2 bytes(16 bits) per channel
    print("* done recording")
    stream.stop_stream()
    stream.close()
    p.terminate()

    * done recording


## Output
`read()` outputs a string, here is a section of the 2nd chunk read,


    frames[1][:20]

    '\xfd\xff\x02\x00\x0f\x00\x0e\x00\t\x00\t\x00\xfd\xff\xff\xff\x00\x00\t\x00'



This looks like stings of binary values which can be converted to printable ASCII characters with `ord`:


    print(map(ord, frames[1][:20]))

    [253, 255, 2, 0, 15, 0, 14, 0, 9, 0, 9, 0, 253, 255, 255, 255, 0, 0, 9, 0]


Let's look at the length of a buffered chunk of audio signal,


    len(frames[1])

    4096



The parameter `FORMAT = pyaudio.paInt16` requests 16 bits (2 bytes) per channel (1 channel here) so each buffer provides 4096/2 = 2048 frames, as requested  by the `FRAMES_PERBUFF` parameter.

##Saving
To save the file in a common audio format is quite easy with the `wave` module:


    import wave
    wf = wave.open('recorded_audio.wav', 'wb')
    wf.setnchannels(CHANNELS)
    wf.setsampwidth(p.get_sample_size(FORMAT))
    wf.setframerate(FRAME_RATE)
    wf.writeframes(b''.join(frames))
    wf.close()
