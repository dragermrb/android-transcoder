android-transcoder
=================

Hardware accelerated transcoder for Android, written in pure Java.

## Why?

Android does not offer straight forward way to transcode video.

FFmpeg is the most famous solution for transcoding. But using [FFmpeg binary on Android](https://github.com/WritingMinds/ffmpeg-android) can cause GPL and/or patent issues. Also using native code for Android development can be troublesome because of cross-compiling, architecture compatibility, build time and binary size.

To transcode without any hassle written above, I created this library to provide hardware accelerated transcoding of H.264 (mp4) video without ffmpeg by using [MediaCodec](https://developer.android.com/intl/ja/reference/android/media/MediaCodec.html).

## Requirements

API Level 18 (Android 4.3, JELLY_BEAN_MR2) or later.
If your app targets older Android, you should add below line to AndroidManifest.xml:

```xml
<!-- Only supports API >= 18 -->
<uses-sdk tools:overrideLibrary="net.ypresto.androidtranscoder" />
```

Please ensure checking Build.VERSION by your self.

## Usage

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    ParcelFileDescriptor parcelFileDescriptor = resolver.openFileDescriptor(data.getData(), "r");
    FileDescriptor fileDescriptor = parcelFileDescriptor.getFileDescriptor();
    MediaTranscoder.Listener listener = new MediaTranscoder.Listener() {
        @Override
        public void onTranscodeProgress(double progress) {
            ...
        }

        @Override
        public void onTranscodeCompleted() {
            startActivity(new Intent(Intent.ACTION_VIEW).setDataAndType(Uri.fromFile(file), "video/mp4"));
            ...
        }

        @Override
        public void onTranscodeFailed(Exception exception) {
            ...
        }
    };
    MediaTranscoder.getInstance().transcodeVideo(fileDescriptor, file.getAbsolutePath(),
            MediaFormatStrategyPresets.createAndroid720pStrategy(), listener); // or createAndroid720pStrategy([your bit rate here])
}
```

See `TranscoderActivity.java` in example directory for ready-made transcoder app.

## Quick Setup

### Gradle

Available from [JCenter](https://bintray.com/bintray/jcenter), which is default repo of gradle script generated by recent android studio.

```groovy
repositories {
   jcenter()
}
```

```groovy
compile 'net.ypresto.androidtranscoder:android-transcoder:0.1.10'
```

## Note (PLEASE READ FIRST)

- This library raises `RuntimeException`s (like `IlleagalStateException`) in various situations. Please catch it and provide alternate logics. I know this is bad design according to Effective Java; just is TODO.
- Currently this library does not generate streaming-aware mp4 file.
Use [qtfaststart-java](https://github.com/ypresto/qtfaststart-java) to place moov atom at beginning of file.
- Android does not gurantees that all devices have bug-free codecs/accelerators for your codec parameters (especially, resolution). Refer [supported media formats](http://developer.android.com/guide/appendix/media-formats.html) for parameters guaranteed by [CTS](https://source.android.com/compatibility/cts-intro.html).
- This library does not support video files recorded by other device like digital cameras, iOS (mov files, including non-baseline profile h.264), etc.

## License

```
Copyright (C) 2014-2016 Yuya Tanaka

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## References for Android Low-Level Media APIs

- http://bigflake.com/mediacodec/
- https://github.com/google/grafika
- https://android.googlesource.com/platform/frameworks/av/+/lollipop-release/media/libstagefright
