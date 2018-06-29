# WebRTC interoperability tests
## Why do we need interoperability tests?
The current state of interoperability testing for WebRTC is still mostly as described by two
blog posts written by Google’s test engineer, Patrik Höglund in mid­2014
[here](http://googletesting.blogspot.se/2014/08/chrome-firefox-webrtc-interop-test-pt-1.html) and 
[here](http://googletesting.blogspot.se/2014/09/chrome-firefox-webrtc-interop-test-pt-2.html).

Basically, the testing process is using the [apprtc sample application](https://apprtc.appspot.com)
in a hermetic environment on Linux to test interoperability with Firefox as part of the Chrome release process.

Other notable examples include the work done by NTTs Yoshimasa Iwase who is running full
factorial tests including different NAT configurations ([described here](http://en.slideshare.net/iwashi86/extreme-testing-of-webrtc-applications)).


Yet, some breakages happen which are not detected by the testing process mentioned
above. For example, an upgrade to Chrome's DTLS library broke the interoperability with the
Jitsi Videobridge in January 2015. This was 
[noticed only very shortly](https://blog.andyet.com/2015/01/30/chrome-update-killed-the-webrtc-star/)
before rolling out to all Chrome users.

Similar issues happened when Mozilla Firefox started to require Perfect Forward Secrecy for
DTLS without announcing this change widely enough. This 
[broke interoperability](https://hacks.mozilla.org/2015/02/webrtc-requires-perfect-forward-secrecy-pfs-starting-in-firefox-38/)
for several mobile applications based on older versions of the webrtc.org library, including
Facebook Messenger which forced Mozilla to postpone the upgrade for several weeks.

More recently, [SRTP between Firefox 49 and Edge was broken](https://bugzilla.mozilla.org/show_bug.cgi?id=1310061) (in one direction) which went unnoticed
for two weeks after Firefox 49 rolled out. And Video interop between Chrome Canary (M56) and Edge was broken in one direction as well due to 
[stricter validation of H264 profile level ids](https://bugs.chromium.org/p/webrtc/issues/detail?id=6552).

Most of these issues have been noticed by people testing manually.

## Acknowledgements
Support for this work was provided in part by the International Multimedia Telecommunications Consortium (IMTC) - http://www.imtc.org 
Many thanks to Bernard Aboba for making me start this!

## Testing process
The testing process is based on the process used in [adapter.js](https://github.com/webrtc/adapter)
and the [samples](https://github.com/webrtc/samples). It uses selenium and
[webdriverjs](https://github.com/SeleniumHQ/selenium/wiki/WebDriverJs) and tests are written using
[tape](https://github.com/substack/tape).

### Chrome
H264 tests currently require Chrome 50 which adds H264 behind a flag.

### Microsoft Edge
The tests for Microsoft Edge only run on Windows currently. The video tests require either the
insider version of Edge (which has H264 enabled by default or the TH2 release of Windows where
H264 is available behind a flag).

### Firefox 
Firefox uses a binary module from the OpenH264 project to provide H264 support. Typically, this
module is downloaded by Firefox shortly after the creation of a new profile. Since Selenium
creates a new profile for each test, the binary needs to be provided in a template profile.
See [this README](h264profile/gmp-gmpopenh264/1.6) for details.

VP9 can be enabled with a flag in Firefox. However, this is compatbile with Chrome only in Firefox 45+.

## AppRTC tests
apprtc.js shows how to test the [AppRTC](https://appr.tc) example provided by Google.
It uses a number of URL parameters to tweak application behaviour, e.g. to force the VP9 or H264
video codec.
Both ICE connectivity as well as video interoperability is tested. For the latter, the frame checker
from [testRTC](https://github.com/webrtc/testrtc) is used.

## Raw interop tests
There is a second set of tests which use plain HTML pages and adapter.js. 
Those tests emulate the PeerConnection API to some degree which makes them look very similar
to some of the tests in adapter.js or the JSFiddles written by Mozilla's [Jan-Ivar](https://github.com/jan-ivar).

Tests currently include
* audio interoperability tests working in Chrome, Microsoft Edge and Firefox
* video interoperability tests in Chrome and Firefox, using VP8, VP9 and H264.
* upgrade tests which upgrade an audio-only call to an audio-video call.
