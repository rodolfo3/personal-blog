---
layout: post
title:  "Why Intent BROWSABLE?"
date:   2015-01-24 00:00:00 -2
categories: android
---

If "Activities that can be safely invoked from a browser must support this category" [1], when it should be used other then URLs?

I have my own answer, testing this HTML:

``` html
<html>
  <body>
    <a href="geo:+0,+0">check this!</a>
  </body>
</html>
```

If you open this in your phone, the link will open the Google Maps activity.

The "$ adb shell dumpsys package" command shows google maps into "https:" and "http:" intents too. This is to allow open links pointing to [any Google Maps location][google-maps-link] in the Google Maps app.

There also an [RFC][rfc5870] to describe the geo URI, with an [website][geouri] to support it.
More information about call an Android Intent from the web web browser on [Chrome Developer Documentation][chrome-android-intents]. The geo URI launch the intent on Firefox, but I didn't testes start an specific activitity on it (like the chrome's sample).

[google-maps-link]: https://www.google.com.br/maps/place/New+York,+NY,+USA
[rfc5870]: http://www.ietf.org/rfc/rfc5870.txt
[geouri]: http://geouri.org/
[chrome-android-intents]: https://developer.chrome.com/multidevice/android/intents
