# mkvtoolnix

Container images with the command-line tools of [MKVToolNix](https://mkvtoolnix.download/). `mkvmerge` creates Matroska and WebM files from other media files, `mkvinfo` shows their structure, `mkvpropedit` changes titles, languages, flags, chapters, tags and attachments in place, and `mkvextract` writes tracks, chapters, tags and attachments back out. They are compiled from the signed source release on Ubuntu and Alpine, without the GUI. The default image also contains FFmpeg and MediaInfo. The images are rebuilt when MKVToolNix publishes a release and when the base image changes, for `linux/amd64` and `linux/arm64`.

This is an unofficial build, not affiliated with or endorsed by the MKVToolNix project. Report problems with the image in this repository and problems with MKVToolNix itself [upstream](https://codeberg.org/mbunkus/mkvtoolnix/issues).

## Quick start

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/mkvtoolnix -o output.mkv input.mp4
```

The same images can also be pulled as `randomcontainers.com/mkvtoolnix`.

mkvmerge copies the tracks without re-encoding them. File options such as `--language 0:de` apply to the file name that follows them, and their track IDs count from 0 in each file. Add German subtitles to a video:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/mkvtoolnix \
  -o output.mkv input.mkv --language 0:de --track-name 0:Deutsch subtitles.de.srt
```

`-J` prints the tracks, chapters, tags and attachments of a file as JSON, with the track IDs the other options use. `-i` prints a short text summary instead:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/mkvtoolnix -J input.mkv
```

mkvmerge exits with 0 when it wrote the file, 1 when it wrote the file but printed warnings, and 2 on errors.

The entrypoint runs `mkvmerge` under `tini`. For the other tools, override it:

```sh
# Change the title and the language of the first audio track without rewriting the file
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint mkvpropedit \
  ghcr.io/randomcontainers/mkvtoolnix input.mkv --edit info --set "title=New title" --edit track:a1 --set language=en

# Write track 2 to an SRT file and the chapters in the simple OGM format
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint mkvextract \
  ghcr.io/randomcontainers/mkvtoolnix input.mkv tracks 2:subtitles.srt chapters --simple chapters.txt

# Show the elements of a Matroska file
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint mkvinfo \
  ghcr.io/randomcontainers/mkvtoolnix input.mkv
```

The [MKVToolNix documentation](https://mkvtoolnix.download/docs.html) covers every option.

## What is in the image

| | slim | default |
|---|---|---|
| `mkvmerge`, `mkvinfo`, `mkvextract` and `mkvpropedit` | yes | yes |
| FFmpeg: `ffmpeg` and `ffprobe` | no | yes |
| MediaInfo: `mediainfo` | no | yes |

FFmpeg and MediaInfo are the [randomcontainers/ffmpeg](https://github.com/randomcontainers/ffmpeg) and [randomcontainers/mediainfo](https://github.com/randomcontainers/mediainfo) builds.

The MKVToolNix programs use Qt Core, Boost, GMP, libogg, libvorbis, FLAC, libdvdread and zlib from the distro. mkvmerge guesses the type of an attachment with Qt's MIME database, which comes from shared-mime-info on Ubuntu and is built into Qt Core on Alpine. libebml, libmatroska, fmt, pugixml, nlohmann-json and utf8-cpp are compiled in from the copies in the source tarball on both distros, so both images run the same code. With libdvdread, `--chapters` also reads the chapters of a DVD, for example `--chapters /work/VIDEO_TS:2` for title 2.

Not included: the MKVToolNix GUI, the `mkvtoolnix` wrapper program, translations, the man pages, which are [online](https://mkvtoolnix.download/docs.html), and developer tools such as `ebml_validator`. The configure options and the libraries configure chose are in `/usr/local/share/randomcontainers/mkvtoolnix/buildinfo`.

## Default or slim

Use the default image (`latest`) when a job needs more than muxing: FFmpeg converts audio or video that you want in another codec, `ffprobe` and `mediainfo` check the result, and mkvmerge puts the tracks together. For example, replace the audio of a file with an Opus version of its first audio track:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint ffmpeg \
  ghcr.io/randomcontainers/mkvtoolnix -i input.mkv -map 0:a:0 -c:a libopus -b:a 128k audio.opus
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/mkvtoolnix \
  -o output.mkv --no-audio input.mkv --language 0:en audio.opus
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint mediainfo \
  ghcr.io/randomcontainers/mkvtoolnix output.mkv
```

`slim` has the four MKVToolNix programs and the libraries they need, without FFmpeg and MediaInfo. Use it to build your own image, or when you only need MKVToolNix.

The default image is also published as `ghcr.io/randomcontainers/mkvtoolnix-ffmpeg-mediainfo`, built in the [mkvtoolnix-ffmpeg-mediainfo](https://github.com/randomcontainers/mkvtoolnix-ffmpeg-mediainfo) repository with the same contents and a different digest.

## Tags

`<version>` is an MKVToolNix release such as `102.0`. `<major>` is its first part, `102`, and follows the newest release with that number. A point release such as `102.0.1` also gets `<major>.<minor>` tags (`102.0`, `102.0-slim-alpine` and so on), which follow the newest point release.

| Default (with FFmpeg and MediaInfo) | Slim | Base |
|---|---|---|
| `latest`, `<version>`, `<major>` | `slim`, `<version>-slim`, `<major>-slim` | Ubuntu |
| `ubuntu`, `<version>-ubuntu`, `<major>-ubuntu` | `slim-ubuntu`, `<version>-slim-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04` | `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `<version>-alpine`, `<major>-alpine` | `slim-alpine`, `<version>-slim-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24` | `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current MKVToolNix version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Both are compiled natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

mkvpropedit changes the file in place, so the mounted directory must be writable. mkvmerge refuses to write to a file that is also one of its inputs.

## Untrusted files

mkvmerge reads dozens of container and stream formats, and all four programs parse Matroska files, with parsers written in C++. libebml, libmatroska, avilib and librmff are compiled into the programs from the MKVToolNix tarball, so their fixes reach the image only with MKVToolNix releases. The programs do not use the network. For files from unknown sources, mount them read-only and take away what the container does not need:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work:ro" \
  --network none --read-only --cap-drop ALL --security-opt no-new-privileges \
  --memory 1g --pids-limit 64 \
  ghcr.io/randomcontainers/mkvtoolnix -J untrusted.mkv
```

To write output, mount a separate writable directory and give mkvmerge a path in it with `-o`. The images pick up a new MKVToolNix release about a day after it is tagged.

## Extending the slim image

Use a `slim` tag as the base for your own image. It has no FFmpeg or MediaInfo, so their updates do not rebuild it. The packages MKVToolNix needs are listed in `/usr/local/share/randomcontainers/mkvtoolnix/runtime-deps`. Switch to root to install more, then back:

```dockerfile
FROM ghcr.io/randomcontainers/mkvtoolnix:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends jq \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

On Alpine, use `apk add --no-cache jq`. jq reads the JSON that `mkvmerge -J` prints. The entrypoint is `["tini", "--", "mkvmerge"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new MKVToolNix releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/mkvtoolnix:latest \
  --repo randomcontainers/mkvtoolnix --signer-repo randomcontainers/ci
```

Images from `ghcr.io/randomcontainers/mkvtoolnix-ffmpeg-mediainfo` are built in that repository, so verify them with `--repo randomcontainers/mkvtoolnix-ffmpeg-mediainfo` and the same `--signer-repo`.

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/mkvtoolnix:latest --format '{{ json .SBOM }}'
```

Before compiling, the build checks the tarball against the SHA-256 recorded in `package.yml` and its signature against Moritz Bunkus's key in `keys/mkvtoolnix-release.gpg` (fingerprint `D919 9745 B054 5F2E 8197 062B 0F92 290A 445B 9007`, signing subkey `3301 A29D 88D0 1A0C F999 954F 74AF 00AD F2E3 2C85`), which the [MKVToolNix authenticity page](https://mkvtoolnix.download/authenticity.html) names.

## Updates

The project checks the `release-<version>` tags of [mbunkus/mkvtoolnix](https://codeberg.org/mbunkus/mkvtoolnix/tags) on Codeberg every 15 minutes. A release is picked up once its tag is 24 hours old and its tarball and signature are on mkvtoolnix.download. The tarball is checked against the `mkvtoolnix-<version>.tar.xz.sha256` file published next to it, the new version and the tarball's SHA-256 are committed to `package.yml`, and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes, the default ones when a new FFmpeg or MediaInfo image is published, and all of them at least every 7 days, so distro security fixes reach the current tags.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_SHA256=<sha256 from package.yml> \
  -t mkvtoolnix:local .
```

Use `Dockerfile.alpine` for the Alpine image. `--build-arg JOBS=<n>` limits the number of parallel compile jobs. The default image is generated from the `combos` entry in `package.yml` by [randomcontainers/ci](https://github.com/randomcontainers/ci).

## Licenses

MKVToolNix is licensed under the GNU General Public License, version 2 (GPL-2.0-only). The programs also contain code from the source tarball: avilib (GPL-2.0-or-later), libebml, libmatroska and librmff (LGPL-2.1-or-later), fmt, pugixml, nlohmann-json and a Base64 encoder (MIT), utf8-cpp (BSL-1.0) and a public domain MD5 implementation. Their license files and notices are in `/usr/local/share/randomcontainers/mkvtoolnix/licenses/`. Qt Core, Boost, GMP, libogg, libvorbis, FLAC, libdvdread, zlib and the other libraries from Ubuntu or Alpine keep their own licenses; the SBOM lists them.

The corresponding source for each image:

- MKVToolNix: every version has a GitHub release in this repository, named `v<version>`, with the exact `mkvtoolnix-<version>.tar.xz` that was compiled and its signature. `/usr/local/share/randomcontainers/mkvtoolnix/source` lists that release and the mkvtoolnix.download URLs.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label. The Dockerfiles hold every configure option.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

The default image adds FFmpeg and MediaInfo. Their license files and corresponding source are described in the [ffmpeg](https://github.com/randomcontainers/ffmpeg#licenses) and [mediainfo](https://github.com/randomcontainers/mediainfo#licenses) repositories.

The files in this repository are available under the MIT license, see [LICENSE](LICENSE).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
