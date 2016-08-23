Using this book: https://www.amazon.com/Hobbyists-Guide-RTL-SDR-Software-Defined-ebook/dp/B00KCDF1QI/ref=pd_sim_351_1?ie=UTF8&dpID=51du2WUoeDL&dpSrc=sims&preST=_UX300_PJku-sticker-v3%2CTopRight%2C0%2C-44_OU01_AC_UL320_SR200%2C320_&refRID=6R3630ZAGZJR41V8TDRS#nav-subnav

I determined that I have a R820T chip.  I have a MCX connector.

Able to get the hardware to work and receive a FM radio station (NPR KQED 88.5 MGhz here in SF) using ... Gqrx: http://gqrx.dk/


# Installing prerequestives

1. install fftw

brew install fftw


2. gnuradio

brew install gnuradio


3. install rtl-sdr

These instructions pretty much just worked:
https://gist.github.com/jheasly/9477732


4. gr-osmosdr

I initially was able to install this with brew, but I came back and installed it from source ... see Git repo and instructions here: https://github.com/osmocom/gr-osmosdr
I also had to `pip install Cheetah` ...


5. crcmod

pip install crcmod 


# Building gr-tpms

Followed the instructions here: https://github.com/jboone/gr-tpms#building


# Running, fixing errors

When I tried running it:

I got this error:

```
✗ tpms_rx --source rtlsdr --if-rate 400000 --tuned-frequency 315000000
Traceback (most recent call last):
  File "/usr/local/bin/tpms_rx", line 29, in <module>
    from gnuradio import analog
  File "/usr/local/lib/python2.7/site-packages/gnuradio/analog/__init__.py", line 35, in <module>
    from am_demod import *
  File "/usr/local/lib/python2.7/site-packages/gnuradio/analog/am_demod.py", line 22, in <module>
    from gnuradio import gr
  File "/usr/local/lib/python2.7/site-packages/gnuradio/gr/__init__.py", line 44, in <module>
    from top_block import *
  File "/usr/local/lib/python2.7/site-packages/gnuradio/gr/top_block.py", line 30, in <module>
    from hier_block2 import hier_block2
  File "/usr/local/lib/python2.7/site-packages/gnuradio/gr/hier_block2.py", line 25, in <module>
    import pmt
  File "/usr/local/lib/python2.7/site-packages/pmt/__init__.py", line 58, in <module>
    from pmt_to_python import pmt_to_python as to_python
  File "/usr/local/lib/python2.7/site-packages/pmt/pmt_to_python.py", line 22, in <module>
    import numpy
ImportError: No module named numpy
```

To fix it, I did

    pip install numpy

... then I recompiled and reinstalled.

-----

Then, I got another error:

```
✗ tpms_rx --source rtlsdr --if-rate 400000 --tuned-frequency 315000000
Traceback (most recent call last):
  File "/usr/local/bin/tpms_rx", line 37, in <module>
    import tpms
  File "/usr/local/lib/python2.7/site-packages/tpms/__init__.py", line 44, in <module>
    from tpms_swig import *
ImportError: No module named tpms_swig
```

I googled and found this: https://github.com/jboone/gr-tpms/issues/3

So, I fixed it by doing:

    brew install swig
    cmake ..
    make
    sudo make install

When it reinstalled, I started seeing good messages like:

```
-- Checking for module SWIG
-- Found SWIG version 3.0.8.
-- Found SWIG: /usr/local/bin/swig  
```        

-----

Then, I got another error:

```
✗ tpms_rx --source rtlsdr --if-rate 400000 --tuned-frequency 315000000
Fatal Python error: PyThreadState_Get: no current thread
[1]    77993 abort      tpms_rx --source rtlsdr --if-rate 400000 --tuned-frequency 315000000
```

The full stack trace is in the file ./CRASH_gr-tpms.txt

It seems like this is because gr-tpms is pointed at the OSX system Python ... whereas everything else is pointed at the brew installed Python ... this thread was helpful:
https://github.com/Homebrew/homebrew-science/issues/3401#issuecomment-213986011

I came up with this as being the right path:

```
PYTHON_INCLUDE_DIR    /usr/local/Cellar/python/2.7.12/Frameworks/Python.framework/Versions/2.7/include/python2.7
PYTHON_LIBRARY        /usr/local/Cellar/python/2.7.12/Frameworks/Python.framework/Versions/2.7/lib/libpython2.7.dylib
```

To configure these, from the directory /proj/sdr-linari/gr-tpms/build ... run:

ccmake ..
press <t> to 'toggle advanced mode', then hold the <down arrow> to page down through all the options until you get to PYTHON_INCLUDE_DIR & PYTHON_LIBRARY ... you can press <enter>, then copypasta the proper values, press <enter> again to make them take, then when you are done with both, press <c> to save them

then recompile & reinstall with

    cmake ..
    make
    sudo make install

-----

I got another, similar error:

```
✗ tpms_rx --source rtlsdr --if-rate 400000 --tuned-frequency 315000000
Mac OS; Clang version 7.3.0 (clang-703.0.31); Boost_106100; UHD_003.010.000.000-0-unknown
Fatal Python error: PyThreadState_Get: no current thread
[1]    44017 abort      tpms_rx --source rtlsdr --if-rate 400000 --tuned-frequency 315000000
```

The full stack trace is in the file ./CRASH_gr-osmosdr.txt

I think, this is the same error as the one I just fixed, except for the gr-osmosdr package (which I had also installed from source) ... so repeat the above fix, to fix and reinstall this one as well ...


# Success & hopefully profit

So, after all this, it started ... and, once I moved the antenna to be closer to the street (10 meters transmit range, according to Google), it decoded 5-10 transmissions within as many minutes.

See the transcript of the successfull run at tpmx_rx_running_TRANSCRIPT_AUG_22nd.txt
