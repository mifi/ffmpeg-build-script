# ffmpeg-build-script

This repo builds lightweight static binaries of `ffmpeg` and `ffprobe` with core functionality only using GitHub Actions. It builds when a tag is created, by pulling down the official source for that tag and compiling them on a Mac OS build runner.
It is used in LosslessCut for Mac App Store and therefore is built with `--disable-securetransport` to prevent the private API usage that is forbidden by Apple in the App Store. For more info see [this blog post](https://blog.mifi.no/2020/03/31/automated-electron-build-with-release-to-mac-app-store-microsoft-store-snapcraft/). For HTTPS (TLS) we use OpenSSL instead.

## Make changes

First make any needed changes in the [build-ffmpeg](build-ffmpeg).

Now build locally:

```bash
# rm -rf packages workspace
./build-ffmpeg --build
```

If all is well, test it and commit.

```bash
git push
```

## Release

Wait for [Github Actions](https://github.com/mifi/ffmpeg-build-script/actions). The build fails if `ffmpeg` or `ffprobe` link anything other than system libraries (`/System/Library/`, `/usr/lib/`), or if their minimum macOS version differs from `MACOSX_DEPLOYMENT_TARGET`.

Now download and test the artifacts.

To make a new release, tag a new version, and then push the tags, for example:
```bash
git tag -m '' -a 6.0-1

git push --follow-tags
```

Wait for GitHub actions to finish (again). The release is published automatically once both architectures are uploaded.

Then update the FFmpeg version in LosslessCut.

CI runs on pushes to `master` and on tags. To build another branch, run the workflow manually (Actions → Build → Run workflow).

## Upgrading

Bump `FFMPEG_VERSION` (tags at https://github.com/FFmpeg/FFmpeg/tags) and the `build "<package>" "<version>"` lines in [build-ffmpeg](build-ffmpeg). Prefer the newest patch release of FFmpeg's release branch that BtbN builds for Windows/Linux (see https://github.com/mifi/ffmpeg-builds), so all platforms are on the same branch. Release tags are named `<ffmpeg version>-<n>`, e.g. `8.1.3-1`. A weekly workflow fails when there's a new patch release on our FFmpeg branch (GitHub disables scheduled workflows after 60 days without repo activity, so re-enable it if needed).

## Credits

This repo is a fork of https://github.com/markus-perl/ffmpeg-build-script. Upstream was restructured in 2026 (the script is split into `src/` and builds many more libraries), so don't merge it wholesale. Port the relevant fixes and version bumps by hand.
