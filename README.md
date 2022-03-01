# ffmpeg-build-script

This repo builds lightweight static binaries of `ffmpeg` and `ffprobe` with core functionality only using GitHub Actions. It builds when a tag is created, by pulling down the official source for that tag and compiling them on a Mac OS build runner.
It is used in LosslessCut for Mac App Store and therefore is built without `--disable-securetransport` to prevent the private API usage that is forbidden by Apple in the App Store. For more info see (this blog post)[https://blog.mifi.no/2020/03/31/automated-electron-build-with-release-to-mac-app-store-microsoft-store-snapcraft/]

## Release new version

- First make any needed changes in the [build-ffmpeg](build-ffmpeg) and commit
- Tag new version
- `git push && git push --tags`

## Credits
This repo is a fork of https://github.com/markus-perl/ffmpeg-build-script
