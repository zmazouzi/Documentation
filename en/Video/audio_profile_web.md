
---
title: Set the Stereo/High-quality Audio Profile
description: How to set high-quality audio on Web
platform: Web
updatedAt: Tue Nov 20 2018 07:05:02 GMT+0000 (UTC)
---
# Set the Stereo/High-quality Audio Profile
## Feature Description 

In some professional scenarios, the audio quality is essential for the user experience. For example, podcast applications require stereo and high-quality audio. The so-called high-quality audio refers to the audio profile of 48 KHz sampling rate and 192 Kbps bitrate, which suits the high-fidelity musical scenarios, such as online radio and singing competitions.

## Implementation

Agora Web SDK provides the [setAudioProfile](https://docs.agora.io/en/Video/API%20Reference/web/interfaces/agorartc.stream.html#setaudioprofile) method for developers to set appropriate audio profiles according to the scenarios. The `profile` parameter sets the sampling rate, bitrate, and encode mode.

```javascript
  // Set the audio profile of 48 KHz sampling rate, stereo, and 192 Kbps bitrate.
  localStream.setAudioProfile("high_quality_stereo");
  localStream.init(function(){
   // init successful
  });
```

> For more options, see  [setAudioProfile](https://docs.agora.io/en/Video/API%20Reference/web/interfaces/agorartc.stream.html#setaudioprofile).

## Considerations

- Call this method before `Stream.init`.