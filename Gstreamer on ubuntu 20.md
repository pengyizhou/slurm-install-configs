# Kaldi (Newest) + Gstreamer (ver. 1.0) Configuration (Ubuntu 20.04 LTS)
## Kaldi installation
### Basic install
*Check INSTALL file in `$kaldi_root`*
### Extra tools install
Install portaudio
```
cd $kaldi_root/tools
./extra/install_portaudio
```

## Gstreamer Installation
### Install gstreamer-1.0 Dependencies
```
sudo apt-get update
sudo apt-get install -y bison flex libffi-dev libmount-dev libpcre3 libpcre3-dev zlib1g-dev libssl-dev gtk-doc-tools
```

### Install ORC-0.4.29
```
wget https://gstreamer.freedesktop.org/src/orc/orc-0.4.29.tar.xz
tar xvJf orc-0.4.29.tar.xz
cd orc-0.4.29/      
./autogen.sh --prefix=/usr/lib
make -j 20     
sudo make install
```
### Install GLIB-2.58.3
```
wget http://ftp.acc.umu.se/pub/GNOME/sources/glib/2.58/glib-2.58.3.tar.xz                           
tar xvJf glib-2.58.3.tar.xz   
cd glib-2.58.3/         
./autogen.sh                           
make -j 20                                                                                          
sudo make install
```
### Download GStreamer
```
wget https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.14.5.tar.xz
wget https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.14.5.tar.xz
wget https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.14.5.tar.xz
wget https://gstreamer.freedesktop.org/src/gst-plugins-bad/gst-plugins-bad-1.14.5.tar.xz
wget https://gstreamer.freedesktop.org/src/gst-plugins-ugly/gst-plugins-ugly-1.14.5.tar.xz
wget https://gstreamer.freedesktop.org/src/gst-plugins-ugly/gst-plugins-ugly-1.14.5.tar.xz
wget https://gstreamer.freedesktop.org/src/gst-rtsp-server/gst-rtsp-server-1.14.5.tar.xz
```

### Install GStreamer
```
tar xvJf  gstreamer-1.14.5.tar.xz gstreamer-1.14.5/
cd gstreamer-1.14.5/
./configure 
make -j 20                                                                                          
sudo make install
```
### Install GStreamer-plugins-base
```
tar xvJf gst-plugins-base-1.14.5.tar.xz 
cd gst-plugins-base-1.14.5/
./configure                     
make -j 20
sudo make install
```
### Install GStreamer-plugins-good
```
sudo apt-get install libxext-dev
tar xvJf gst-plugins-good-1.14.5.tar.xz 
cd gst-plugins-good-1.14.5/
./configure 
vim sys/v4l2/ext/types-compat.h (reference to https://gitlab.freedesktop.org/gstreamer/gst-plugins-good/-/merge_requests/292/diffs)

#include <glib.h>

#ifndef __TYPES_COMPAT_H__
#define __TYPES_COMPAT_H__
#ifdef   __linux__
#include <linux/types.h>
#include <asm/ioctl.h>

#else /* One of the BSDs */
/* From linux/types.h */
#ifndef __bitwise__
#  ifdef __CHECKER__
#    define __bitwise__ __attribute__((bitwise))
#  else
#    define __bitwise__
#  endif
#endif

#ifndef __bitwise
#  ifdef __CHECK_ENDIAN__
#    define __bitwise __bitwise__
#  else
#    define __bitwise
#  endif
#endif

typedef guint8  __u8;
typedef guint16 __u16;
typedef gint32  __s32;
typedef guint32 __u32;
typedef gint64  __s64;
typedef guint64 __u64;
typedef guint32 __bitwise __le32;

#endif
#define __user

#endif /* __TYPES_COMPAT_H__ */

make -j 20
sudo make install

```
### Install GStreamer-plugins-bad
```
tar xvJf gst-plugins-bad-1.14.5.tar.xz 
cd gst-plugins-bad-1.14.5/
./configure CXXFLAGS=-std=c++11 
make -j 20
sudo make install
```
### Install GStreamer-plugins-ugly
```
sudo apt-get install libx264-dev
tar xvJf gst-plugins-ugly-1.14.5.tar.xz 
cd gst-plugins-ugly-1.14.5/
./configure 
make -j 20
sudo make install
```
### Update user env
```
vim ~/.bashrc
export LD_LIBRARY_PATH=/usr/local/lib
export GST_PLUGIN_PATH=/usr/local/lib:/usr/local/lib/gstreamer-1.0
```
### Install GST-RTSP-SERVER
```
tar xvJf gst-rtsp-server-1.14.5.tar.xz 
cd gst-rtsp-server-1.14.5/
./configure 
make -j 20
sudo make install
```

### Install gst-kaldi-nnet2-online
```
git clone https://github.com/alumae/gst-kaldi-nnet2-online.git
cd src
KALDI_ROOT=$kaldiroot make depend
KALDI_ROOT=$kaldiroot make
```
### Test gst-kaldi-nnet2-online
Run the following command at $gst_root

```
GST_PLUGIN_PATH=. gst-inspect-1.0 kaldinnet2onlinedecoder
```

And you would get the information like below if it is successfully installed.
```
Factory Details:
  Rank                     none (0)
  Long-name                KaldiNNet2OnlineDecoder
  Klass                    Speech/Audio
  Description              Convert speech to text
  Author                   Tanel Alumae <tanel.alumae@phon.ioc.ee>

Plugin Details:
  Name                     kaldinnet2onlinedecoder
  Description              kaldinnet2onlinedecoder
  Filename                 ./src/libgstkaldinnet2onlinedecoder.so
  Version                  1.0
  License                  unknown
  Source module            Kaldi
  Binary package           GStreamer
  Origin URL               http://gstreamer.net/
  ...
  ...
```

## Configure ASR model (e.g. /home/demos/Uyghur_model_Oct_6 @SERVER3)
### The GST project folder should be organized as follows
```
.
├── asr-engine.sh
├── client
├── conf
│   ├── ivector_extractor.conf
│   ├── mfcc_hires.conf
│   ├── online_cmvn.conf
│   └── splice.conf
├── decode
├── ivector-extractor
│   ├── 10.ie
│   ├── final.dubm
│   ├── final.mat
│   ├── final.ie.id
│   ├── global_cmvn.stats
│   ├── online_cmvn.conf
│   ├── splice_opts
├── kaldigstserver
├── log
│   ├── server.log
│   └── worker.log
├── path.sh
├── tdnnf
│   ├── final.mdl
│   └── graph-base
│       ├── disambig_tid.int
│       ├── HCLG.fst
│       ├── num_pdfs
│       ├── phones
│           ├── align_lexicon.int
│           ├── align_lexicon.txt
│           ├── disambig.int
│           ├── disambig.txt
│           ├── optional_silence.csl
│           ├── optional_silence.int
│           ├── optional_silence.txt
│           ├── silence.csl
│           ├── word_boundary.int
│           └── word_boundary.txt
│       ├── phones.txt
│       └── words.txt
├── $language.yaml
```
### **$language.yaml** file
```
# You have to download Librispeech "online nnet2" models in order to use this sample
# Run download-librispeech-nnet2.sh in 'test/models' to download them.
use-nnet2: true
decoder:
    # All the properties nested here correspond to the kaldinnet2onlinedecoder GStreamer plugin properties.
    # Use gst-inspect-1.0 ./libgstkaldionline2.so kaldinnet2onlinedecoder to discover the available properties
    nnet-mode: 3 
    use-threaded-decoder: true

    model : tdnnf/final.mdl
    word-syms : tdnnf/graph-base/words.txt
    fst : tdnnf/graph-base/HCLG.fst
    mfcc-config : conf/mfcc_hires.conf
    ivector-extraction-config : conf/ivector_extractor.conf

    max-active: 5000
    beam: 10.0
    lattice-beam: 4.0
    acoustic-scale: 1.0
    do-endpointing : true
    endpoint-silence-phones : "1:2:3:4:5" 
    # You should check the silence phones table
    traceback-period-in-secs: 0.25
    chunk-length-in-secs: 0.2
    num-nbest: 10
    feature-type: mfcc
    frame-subsampling-factor: 3
    #Additional functionality that you can play with:
    #lm-fst:  man_model/G.fst
    #big-lm-const-arpa: man_model/G.carpa
    #phone-syms: man_model/phones.txt
    #word-boundary-file: man_model/word_boundary.int
    #do-phone-alignment: true
out-dir: tmp

use-vad: False
silence-timeout: 10

# Just a sample post-processor that appends "." to the hypothesis
post-processor: perl -npe 'BEGIN {use IO::Handle; STDOUT->autoflush(1);} s/(.*)/\1./;'

# A sample full post processor that add a confidence score to 1-best hyp and deletes other n-best hyps
#full-post-processor: ./sample_full_post_processor.py

logging:
    version : 1
    disable_existing_loggers: False
    formatters:
        simpleFormater:
            format: '%(asctime)s - %(levelname)7s: %(name)10s: %(message)s'
            datefmt: '%Y-%m-%d %H:%M:%S'
    handlers:
        console:
            class: logging.StreamHandler
            formatter: simpleFormater
            level: DEBUG
    root:
        level: DEBUG
        handlers: [console]
```

### **conf/ivector_extractor.conf**
```
--cmvn-config=conf/online_cmvn.conf
--ivector-period=10
--splice-config=conf/splice.conf
--lda-matrix=ivector-extractor/final.mat
--global-cmvn-stats=ivector-extractor/global_cmvn.stats
--diag-ubm=ivector-extractor/final.dubm
--ivector-extractor=ivector-extractor/final.ie
--num-gselect=5
--min-post=0.025
--posterior-scale=0.1
--max-remembered-frames=1000
--max-count=0
```

### **conf/online_cmvn.conf** is Empty
### **conf/splice.conf** is the same as **splice_opts** in ivector-extractor folder, just modify the format from one line to two lines
```
--left-context=3
--right-context=3
```
### **asr-engine.sh** 
An example of running scripts based on Uyghur
```
#!/bin/bash
. path.sh
UY_YAML=Uyghur.yaml

UY_PORT=10011 # We have 10011-10018 ports for eight languages

UY_NUM=2 # Number of decoders, one decoder can only work for one request at one moment

if [ $# = 0 ];then
    echo "USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND"
    ps -aux | grep kaldigstserver | grep -v grep
        exit 1
fi
#--------------------------------------------------------------------------------------------------
if [ $1 = start ];then

    if [ ! $UY_NUM = 0 ];then(./kaldigstserver/master_server.py --port=$UY_PORT > ./log/server.log 2>&1)& fi
    for x in $(seq 1 $UY_NUM); do
        echo "start worker port:$UY_PORT No.$x configure:$UY_YAML"
        (./kaldigstserver/worker.py -u ws://localhost:$UY_PORT/worker/ws/speech -c ./$UY_YAML  > ./log/worker.log 2>&1)&
    done
        echo ""
        echo "USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND"
    ps -aux | grep kaldigstserver | grep -v grep
fi
#--------------------------------------------------------------------------------------------------
if [ $1 = killall ];then
        echo ""
        echo "Kill all server and worker"
        echo ""
        process=`ps -ef | grep kaldigstserver/worker.py | grep -v grep | grep -v PPID | awk '{print $2}'`
    for x in $process;do kill $x; echo "Kill the worker process [ $x ]"; done
        process=`ps -ef | grep kaldigstserver/master_server.py | grep -v grep | grep -v PPID | awk '{print $2}'`
    for x in $process;do kill $x; echo "Kill the server process [ $x ]"; done
```

### Start GST service
```
### START SERVICE
./asr-engine.sh start
### KILL ALL SERVICES
./asr-engine.sh killall
### SHOW RUNNING SERVICES
./asr-engine.sh
```
