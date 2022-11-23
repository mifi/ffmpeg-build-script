# ffmpeg-build-script

This repo builds lightweight static binaries of `ffmpeg` and `ffprobe` with core functionality only using GitHub Actions. It builds when a tag is created, by pulling down the official source for that tag and compiling them on a Mac OS build runner.
It is used in LosslessCut for Mac App Store and therefore is built without `--disable-securetransport` to prevent the private API usage that is forbidden by Apple in the App Store. For more info see (this blog post)[https://blog.mifi.no/2020/03/31/automated-electron-build-with-release-to-mac-app-store-microsoft-store-snapcraft/]

## Release new version

- First make any needed changes in the [build-ffmpeg](build-ffmpeg) and commit
- Tag new version
- `git push && git push --tags`

## After build complete

Check the Github Actions build that `otool -L` looks something like this:

```
/System/Library/Frameworks/Foundation.framework/Versions/C/Foundation (compatibility version 300.0.0, current version 1677.104.0)
/System/Library/Frameworks/AudioToolbox.framework/Versions/A/AudioToolbox (compatibility version 1.0.0, current version 1000.0.0)
/System/Library/Frameworks/CoreAudio.framework/Versions/A/CoreAudio (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1281.100.1)
/System/Library/Frameworks/AVFoundation.framework/Versions/A/AVFoundation (compatibility version 1.0.0, current version 2.0.0)
/System/Library/Frameworks/CoreVideo.framework/Versions/A/CoreVideo (compatibility version 1.2.0, current version 1.5.0)
/System/Library/Frameworks/CoreMedia.framework/Versions/A/CoreMedia (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/CoreGraphics.framework/Versions/A/CoreGraphics (compatibility version 64.0.0, current version 1355.22.0)
/System/Library/Frameworks/OpenGL.framework/Versions/A/OpenGL (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/QuartzCore.framework/Versions/A/QuartzCore (compatibility version 1.0.1, current version 5.0.0)
/System/Library/Frameworks/AppKit.framework/Versions/C/AppKit (compatibility version 45.0.0, current version 1894.60.100)
/usr/lib/libbz2.1.0.dylib (compatibility version 1.0.0, current version 1.0.5)
/usr/lib/libz.1.dylib (compatibility version 1.0.0, current version 1.2.11)
/usr/lib/libiconv.2.dylib (compatibility version 7.0.0, current version 7.0.0)
/System/Library/Frameworks/VideoToolbox.framework/Versions/A/VideoToolbox (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation (compatibility version 150.0.0, current version 1677.104.0)
/System/Library/Frameworks/CoreServices.framework/Versions/A/CoreServices (compatibility version 1.0.0, current version 1069.24.0)
/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
```

MacOS arm64 must be build manually for now. Build using `./build-ffmpeg --build` then upload the artifacts to the GitHub release at https://github.com/mifi/ffmpeg-builds

## Credits
This repo is a fork of https://github.com/markus-perl/ffmpeg-build-script
