#!/bin/bash

# HOMEPAGE: https://github.com/markus-perl/ffmpeg-build-script
# LICENSE: https://github.com/markus-perl/ffmpeg-build-script/blob/master/LICENSE

# Min electron supported version
MACOS_MIN=10.10

PROGNAME=$(basename "$0")
FFMPEG_VERSION=4.4.1
CWD=$(pwd)
PACKAGES="$CWD/packages"
WORKSPACE="$CWD/workspace"
CFLAGS="-I$WORKSPACE/include -mmacosx-version-min=${MACOS_MIN}"
LDFLAGS="-L$WORKSPACE/lib -mmacosx-version-min=${MACOS_MIN}"
LDEXEFLAGS=""
# EXTRALIBS="-ldl -lpthread -lm -lz"
EXTRALIBS="-lpthread -lm"
MACOS_M1=false
CONFIGURE_OPTIONS=()
LATEST=false


# Check for Apple Silicon
if [[ ("$(uname -m)" == "arm64") && ("$OSTYPE" == "darwin"*) ]]; then
  # If arm64 AND darwin (macOS)
  export ARCH=arm64
  export MACOSX_DEPLOYMENT_TARGET=11.0
  MACOS_M1=true
fi

# Speed up the process
# Env Var NUMJOBS overrides automatic detection
if [[ -n "$NUMJOBS" ]]; then
  MJOBS="$NUMJOBS"
elif [[ -f /proc/cpuinfo ]]; then
  MJOBS=$(grep -c processor /proc/cpuinfo)
elif [[ "$OSTYPE" == "darwin"* ]]; then
  MJOBS=$(sysctl -n machdep.cpu.thread_count)
  CONFIGURE_OPTIONS=("--enable-videotoolbox")
  MACOS_LIBTOOL="$(which libtool)" # gnu libtool is installed in this script and need to avoid name conflict
else
  MJOBS=4
fi

make_dir() {
  remove_dir "$1"
  if ! mkdir "$1"; then
    printf "\n Failed to create dir %s" "$1"
    exit 1
  fi
}

remove_dir() {
  if [ -d "$1" ]; then
    rm -r "$1"
  fi
}

download() {
  # download url [filename[dirname]]

  DOWNLOAD_PATH="$PACKAGES"
  DOWNLOAD_FILE="${2:-"${1##*/}"}"

  if [[ "$DOWNLOAD_FILE" =~ tar. ]]; then
    TARGETDIR="${DOWNLOAD_FILE%.*}"
    TARGETDIR="${3:-"${TARGETDIR%.*}"}"
  else
    TARGETDIR="${3:-"${DOWNLOAD_FILE%.*}"}"
  fi

  if [ ! -f "$DOWNLOAD_PATH/$DOWNLOAD_FILE" ]; then
    echo "Downloading $1 as $DOWNLOAD_FILE"
    curl -L --silent -o "$DOWNLOAD_PATH/$DOWNLOAD_FILE" "$1"

    EXITCODE=$?
    if [ $EXITCODE -ne 0 ]; then
      echo ""
      echo "Failed to download $1. Exitcode $EXITCODE. Retrying in 10 seconds"
      sleep 10
      curl -L --silent -o "$DOWNLOAD_PATH/$DOWNLOAD_FILE" "$1"
    fi

    EXITCODE=$?
    if [ $EXITCODE -ne 0 ]; then
      echo ""
      echo "Failed to download $1. Exitcode $EXITCODE"
      exit 1
    fi

    echo "... Done"
  else
    echo "$DOWNLOAD_FILE has already downloaded."
  fi

  make_dir "$DOWNLOAD_PATH/$TARGETDIR"

  if [[ "$DOWNLOAD_FILE" == *"patch"* ]]; then
    return
  fi

  if [ -n "$3" ]; then
    if ! tar -xvf "$DOWNLOAD_PATH/$DOWNLOAD_FILE" -C "$DOWNLOAD_PATH/$TARGETDIR" 2>/dev/null >/dev/null; then
      echo "Failed to extract $DOWNLOAD_FILE"
      exit 1
    fi
  else
    if ! tar -xvf "$DOWNLOAD_PATH/$DOWNLOAD_FILE" -C "$DOWNLOAD_PATH/$TARGETDIR" --strip-components 1 2>/dev/null >/dev/null; then
      echo "Failed to extract $DOWNLOAD_FILE"
      exit 1
    fi
  fi

  echo "Extracted $DOWNLOAD_FILE"

  cd "$DOWNLOAD_PATH/$TARGETDIR" || (
    echo "Error has occurred."
    exit 1
  )
}

execute() {
  echo "$ $*"

  OUTPUT=$("$@" 2>&1)

  # shellcheck disable=SC2181
  if [ $? -ne 0 ]; then
    echo "$OUTPUT"
    echo ""
    echo "Failed to Execute $*" >&2
    exit 1
  fi
}

build() {
  echo ""
  echo "building $1 - version $2"
  echo "======================="

  if [ -f "$PACKAGES/$1.done" ]; then
    if grep -Fx "$2" "$PACKAGES/$1.done" >/dev/null; then
      echo "$1 version $2 already built. Remove $PACKAGES/$1.done lockfile to rebuild it."
      return 1
    elif $LATEST; then
      echo "$1 is outdated and will be rebuilt with latest version $2"
      return 0
    else
      echo "$1 is outdated, but will not be rebuilt. Pass in --latest to rebuild it or remove $PACKAGES/$1.done lockfile."
      return 1
    fi
  fi

  return 0
}

command_exists() {
  if ! [[ -x $(command -v "$1") ]]; then
    return 1
  fi

  return 0
}

library_exists() {
  if ! [[ -x $(pkg-config --exists --print-errors "$1" 2>&1 >/dev/null) ]]; then
    return 1
  fi

  return 0
}

build_done() {
  echo "$2" > "$PACKAGES/$1.done"
}

verify_binary_type() {
  if ! command_exists "file"; then
    return
  fi

  BINARY_TYPE=$(file "$WORKSPACE/bin/ffmpeg" | sed -n 's/^.*\:\ \(.*$\)/\1/p')
  echo ""
  case $BINARY_TYPE in
  "Mach-O 64-bit executable arm64")
    echo "Successfully built Apple Silicon (M1) for ${OSTYPE}: ${BINARY_TYPE}"
    ;;
  *)
    echo "Successfully built binary for ${OSTYPE}: ${BINARY_TYPE}"
    ;;
  esac
}

cleanup() {
  remove_dir "$PACKAGES"
  remove_dir "$WORKSPACE"
  echo "Cleanup done."
  echo ""
}

usage() {
  echo "Usage: $PROGNAME [OPTIONS]"
  echo "Options:"
  echo "  -h, --help                     Display usage information"
  echo "      --version                  Display version information"
  echo "  -b, --build                    Starts the build process"
  echo "  -c, --cleanup                  Remove all working dirs"
  echo "      --latest                   Build latest version of dependencies if newer available"
  echo ""
}

echo "ffmpeg-build-script"
echo "========================="
echo ""

while (($# > 0)); do
  case $1 in
  -h | --help)
    usage
    exit 0
    ;;
  -*)
    if [[ "$1" == "--build" || "$1" =~ '-b' ]]; then
      bflag='-b'
    fi
    if [[ "$1" == "--cleanup" || "$1" =~ '-c' && ! "$1" =~ '--' ]]; then
      cflag='-c'
      cleanup
    fi
    if [[ "$1" == "--latest" ]]; then
      LATEST=true
    fi
    shift
    ;;
  *)
    usage
    exit 1
    ;;
  esac
done

if [ -z "$bflag" ]; then
  if [ -z "$cflag" ]; then
    usage
    exit 1
  fi
  exit 0
fi

echo "Using $MJOBS make jobs simultaneously."

if [ -n "$LDEXEFLAGS" ]; then
  echo "Start the build in full static mode."
fi

mkdir -p "$PACKAGES"
mkdir -p "$WORKSPACE"

export PATH="${WORKSPACE}/bin:$PATH"
PKG_CONFIG_PATH="/usr/local/lib/x86_64-linux-gnu/pkgconfig:/usr/local/lib/pkgconfig"
PKG_CONFIG_PATH+=":/usr/local/share/pkgconfig:/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib64/pkgconfig"
export PKG_CONFIG_PATH

if ! command_exists "make"; then
  echo "make not installed."
  exit 1
fi

if ! command_exists "g++"; then
  echo "g++ not installed."
  exit 1
fi

if ! command_exists "curl"; then
  echo "curl not installed."
  exit 1
fi

if ! command_exists "cargo"; then
  echo "cargo not installed. rav1e encoder will not be available."
fi

if ! command_exists "python3"; then
  echo "python3 command not found. Lv2 filter and dav1d decoder will not be available."
fi

##
## build tools
##

if build "giflib" "5.2.1"; then
  download "https://netcologne.dl.sourceforge.net/project/giflib/giflib-5.2.1.tar.gz"
    if [[ "$OSTYPE" == "darwin"* ]]; then
      download "https://sourceforge.net/p/giflib/bugs/_discuss/thread/4e811ad29b/c323/attachment/Makefile.patch"
      execute patch -p0 --forward "${PACKAGES}/giflib-5.2.1/Makefile" "${PACKAGES}/Makefile.patch" || true
    fi
  cd "${PACKAGES}"/giflib-5.2.1 || exit
  #multicore build disabled for this library
  execute make
  execute make PREFIX="${WORKSPACE}" install
  build_done "giflib" "5.2.1"
fi

if build "pkg-config" "0.29.2"; then
  download "https://pkgconfig.freedesktop.org/releases/pkg-config-0.29.2.tar.gz"
  execute ./configure --silent --prefix="${WORKSPACE}" --with-pc-path="${WORKSPACE}"/lib/pkgconfig --with-internal-glib
  execute make -j $MJOBS
  execute make install
  build_done "pkg-config" "0.29.2"
fi

if build "yasm" "1.3.0"; then
  download "https://github.com/yasm/yasm/releases/download/v1.3.0/yasm-1.3.0.tar.gz"
  execute ./configure --prefix="${WORKSPACE}"
  execute make -j $MJOBS
  execute make install
  build_done "yasm" "1.3.0"
fi

if build "nasm" "2.15.05"; then
  download "https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/nasm-2.15.05.tar.xz"
  execute ./configure --prefix="${WORKSPACE}" --disable-shared --enable-static
  execute make -j $MJOBS
  execute make install
  build_done "nasm" "2.15.05"
fi

if build "zlib" "1.2.11"; then
  download "https://www.zlib.net/zlib-1.2.11.tar.gz"
  execute ./configure --static --prefix="${WORKSPACE}"
  execute make -j $MJOBS
  execute make install
  build_done "zlib" "1.2.11"
fi

if build "m4" "1.4.19"; then
  download "https://ftp.gnu.org/gnu/m4/m4-1.4.19.tar.gz"
  execute ./configure --prefix="${WORKSPACE}"
  execute make -j $MJOBS
  execute make install
  build_done "m4" "1.4.19"
fi

if build "autoconf" "2.71"; then
  download "https://ftp.gnu.org/gnu/autoconf/autoconf-2.71.tar.gz"
  execute ./configure --prefix="${WORKSPACE}"
  execute make -j $MJOBS
  execute make install
  build_done "autoconf" "2.71"
fi

if build "automake" "1.16.5"; then
  download "https://ftp.gnu.org/gnu/automake/automake-1.16.5.tar.gz"
  execute ./configure --prefix="${WORKSPACE}"
  execute make -j $MJOBS
  execute make install
  build_done "automake" "1.16.5"
fi

if build "libtool" "2.4.6"; then
  download "https://ftpmirror.gnu.org/libtool/libtool-2.4.6.tar.gz"
  execute ./configure --prefix="${WORKSPACE}" --enable-static --disable-shared
  execute make -j $MJOBS
  execute make install
  build_done "libtool" "2.4.6"
fi

if build "cmake" "3.22.1"; then
  download "https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1.tar.gz"
  execute ./configure --prefix="${WORKSPACE}" --parallel="${MJOBS}" -- -DCMAKE_USE_OPENSSL=OFF
  execute make -j $MJOBS
  execute make install
  build_done "cmake" "3.22.1"
fi

##
## Codecs
##

if build "x264" "5db6aa6"; then
	download "https://code.videolan.org/videolan/x264/-/archive/5db6aa6cab1b146e07b60cc1736a01f21da01154/x264-5db6aa6cab1b146e07b60cc1736a01f21da01154.tar.gz" "x264-5db6aa6.tar.gz"
	cd "${PACKAGES}"/x264-5db6aa6 || exit

	if [[ "$OSTYPE" == "linux-gnu" ]]; then
		execute ./configure --prefix="${WORKSPACE}" --enable-static --enable-pic CXXFLAGS="-fPIC"
	else
		execute ./configure --prefix="${WORKSPACE}" --enable-static --enable-pic
	fi

	execute make -j $MJOBS
	execute make install
	execute make install-lib-static

	build_done "x264" "5db6aa6"
fi
CONFIGURE_OPTIONS+=("--enable-libx264")

##
## FFmpeg
##

EXTRA_VERSION=""
if [[ "$OSTYPE" == "darwin"* ]]; then
  EXTRA_VERSION="${FFMPEG_VERSION}"
fi

FFMPEG_URL="https://git.ffmpeg.org/gitweb/ffmpeg.git/snapshot/n$FFMPEG_VERSION.tar.gz"
#FFMPEG_URL="https://github.com/FFmpeg/FFmpeg/archive/refs/heads/release/$FFMPEG_VERSION.tar.gz"

build "ffmpeg" "$FFMPEG_VERSION"
download "${FFMPEG_URL}" "FFmpeg-release-$FFMPEG_VERSION.tar.gz"
# shellcheck disable=SC2086
./configure "${CONFIGURE_OPTIONS[@]}" \
  --disable-debug \
  --disable-doc \
  --disable-shared \
  --enable-pthreads \
  --enable-static \
  --enable-version3 \
  --extra-cflags="${CFLAGS}" \
  --extra-ldexeflags="${LDEXEFLAGS}" \
  --extra-ldflags="${LDFLAGS}" \
  --extra-libs="${EXTRALIBS}" \
  --pkgconfigdir="$WORKSPACE/lib/pkgconfig" \
  --pkg-config-flags="--static" \
  --prefix="${WORKSPACE}" \
  --extra-version="${EXTRA_VERSION}" \
	\
	--disable-securetransport \
	--disable-ffplay \
	--disable-lzma \
	--enable-runtime-cpudetect \
	--enable-avfilter \
	--enable-filters \
	\
	--enable-libx264 \


execute make -j $MJOBS
execute make install

verify_binary_type

otool -L "$WORKSPACE/bin/ffmpeg"
otool -L "$WORKSPACE/bin/ffprobe"

echo ""
echo "Building done. The following binaries can be found here:"
echo "- ffmpeg: $WORKSPACE/bin/ffmpeg"
echo "- ffprobe: $WORKSPACE/bin/ffprobe"
echo "- ffplay: $WORKSPACE/bin/ffplay"
echo ""

exit 0
