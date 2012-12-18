# How to record audio in Chrome with native HTML5 APIs

Two weeks ago a [new version](http://googlechromereleases.blogspot.com/2012/11/stable-channel-release-and-beta-channel.html) of Chrome was released. Google switched from the default Adobe's Flash Player to an in-house developed version called "[Pepper Flash](http://blog.chromium.org/2012/08/the-road-to-safer-more-stable-and.html)". Unfortunately Pepper Flash has a [problem with audio recording](http://code.google.com/p/chromium/issues/detail?id=157613), resulting in [distorted audio](https://www.youtube.com/watch?v=3ugdk89ojxU) on almost all Macs.

<a href="http://dubjoy.com"><img src="http://media.tumblr.com/tumblr_me7lakk15j1rwn17m.png" style="padding-left: 10px; padding-bottom: 10px; float: right;"></a>

This happened right in the middle of our efforts to build the [Dubjoy](http://dubjoy.com) Editor, a browser-based, easy to use tool for translating (dubbing) online videos. Relying on Flash for audio recording was our first choice, but when confronted with this devastating issue, we started looking into other options. Using native HTML5 APIs seemed like a viable solution.

We started researching the space and checked a lot of sample code out there, but had limited success.

From what you can find on [html5rocks](http://html5rocks.com), capturing audio seems to be well supported. We started with the [sample code for capturing video](http://html5rocks.com/en/tutorials/getusermedia/intro) and modified it for our audio recording test:

    <html>
      <body>
        <audio controls autoplay></audio>

        <input onclick="startRecording()" type="button" value="start recording" />
        <input onclick="stopRecording()" type="button" value="stop recording and play" />

        <script>
          var onFail = function(e) {
            console.log('Rejected!', e);
          };

          var onSuccess = function(s) {
            stream = s;
          }

          window.URL = window.URL || window.webkitURL;
          navigator.getUserMedia  = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;

          var stream;
          var audio = document.querySelector('audio');

          function startRecording() {
            if (navigator.getUserMedia) {
              navigator.getUserMedia({audio: true}, onSuccess, onFail);
            } else {
              console.log('navigator.getUserMedia not present');
            }
          }

          function stopRecording() {
            audio.src = window.URL.createObjectURL(stream);
          }
        </script>
      </body>
    </html>

Everything seems easy and pretty straightforward, right? *Wrong!*

When clicking the "start recording" button, a permission request to use the microphone appears. After allowing access the recording should start, but clicking on the "stop recording and play" button does absolutely nothing.

Looks like the problem lies in assigning the recorded stream to the native audio source as it's done in the [video sample on html5rocks](http://html5rocks.com/en/tutorials/getusermedia/intro):

    audio.src = window.URL.createObjectURL(stream);

After playing around and searching the web for hours, we found countless posts of people asking on forums why it isn't working. The answer is that the current implementation of Chrome returns raw audio samples. These are not playable by the native `<audio>` control.

What you need to do is to create an *audio context* and a *media stream source*:

    var context = new webkitAudioContext();
    var mediaStreamSource = context.createMediaStreamSource(s);

These can be used for creating an audio loop that enables you to hear your own voice:

    mediaStreamSource.connect(context.destination);

To implement the recording functionality you need to buffer the returned raw audio samples until the recording is done. If you want to play it back, you need to convert the buffered samples to a format that can be played natively by the `<audio>` control. This can be quite cumbersome and not something you want to spend your time with.

Luckily there are libraries available that handle this for you. [Recorderjs](https://github.com/mattdiamond/Recorderjs) is the one we used and did the trick.

The *media stream source* created above, can be passed as a parameter to the recorder object for buffering raw audio samples:

    recorder = new Recorder(mediaStreamSource);
    recorder.record();

When done, the recorder object can promptly convert the buffered audio to a natively playable WAV file:

    recorder.stop();
    recorder.exportWAV(function(s) {
      audio.src = window.URL.createObjectURL(s);
    });

Which is exactly what we were looking for.

Here's the full version of the HTML5 native audio recorder complete with playback functionality:

    <html>
      <body>
        <audio controls autoplay></audio>
        <script type="text/javascript" src="recorder.js"> </script>

        <input onclick="startRecording()" type="button" value="start recording" />
        <input onclick="stopRecording()" type="button" value="stop recording and play" />

        <script>
          var onFail = function(e) {
            console.log('Rejected!', e);
          };

          var onSuccess = function(s) {
            var context = new webkitAudioContext();
            var mediaStreamSource = context.createMediaStreamSource(s);
            recorder = new Recorder(mediaStreamSource);
            recorder.record();

            // audio loopback
            // mediaStreamSource.connect(context.destination);
          }

          window.URL = window.URL || window.webkitURL;
          navigator.getUserMedia  = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msGetUserMedia;

          var recorder;
          var audio = document.querySelector('audio');

          function startRecording() {
            if (navigator.getUserMedia) {
              navigator.getUserMedia({audio: true}, onSuccess, onFail);
            } else {
              console.log('navigator.getUserMedia not present');
            }
          }

          function stopRecording() {
            recorder.stop();
            recorder.exportWAV(function(s) {
              audio.src = window.URL.createObjectURL(s);
            });
          }
        </script>
      </body>
    </html>

Unfortunately there’s a caveat. In the current stable version of Chrome, the support for native HTML5 audio playback is **not enabled by default**. For the code above to work you need to enable "Web Audio Input" in *chrome://flags*, which is a huge deal breaker for us.

Before starting the development of the [Dubjoy](http://dubjoy.com) Editor, we decided to support Chrome as our only browser, because of its wide adoption on both Macs and Windows, auto-updates and the built-in Flash Player.

After trying hard to find a workaround, we’re still waiting for the Chrome team to fix the "[Pepper Flash Bug](http://code.google.com/p/chromium/issues/detail?id=157613)" (that in the meantime has spread to millions of users around the world) or to enable "Web Audio Input" by default in their stable version of Chrome.

HTML5 is very promising and when browsers will support it widely, a lot of the problems we face today will disappear. But we’re not quite there yet.

You can [download sample code](https://github.com/rokgregoric/html5record/archive/master.zip) from Github.
