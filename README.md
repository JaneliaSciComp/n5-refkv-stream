# n5-refkv-stream

An [N5](https://github.com/saalfeldlab/n5) block codec for the `refkv.stream` convention of
[refkv](https://github.com/d-v-b/refkv) (conventions §10): video in Zarr arrays, where a chunk is the coded frames
of a stream (H.264, HEVC, VP9 or AV1) that refkv's `zarrify` points at inside the original file (an MP4, say).
The codec decodes a chunk's frames with FFmpeg ([JavaCPP presets](https://github.com/bytedeco/javacpp-presets)) and
returns one plane (`y`, `u` or `v`) of each picture, so N5 and the tools on it (BigDataViewer, ImgLib2) read a video as an
ordinary `uint8` array of shape `[frame, y, x]`.

Put the jar on the classpath: N5 finds the codec, `refkv.stream`, through its annotation index.

```xml
<dependency>
  <groupId>org.refkv</groupId>
  <artifactId>n5-refkv-stream</artifactId>
  <version>0.1.0-SNAPSHOT</version>
</dependency>
<!-- the FFmpeg natives of the platform: org.bytedeco:ffmpeg:8.1.2-1.5.14 with classifier macosx-arm64, linux-x86_64,
     windows-x86_64, ..., or org.bytedeco:ffmpeg-platform -->
```

## What it reads

- `format` of `h264`, `hevc`, `vp9` or `av1`; `setup`, the decoder's configuration record, base64 (for H.264 the `avcC`
  box); `plane` of `y`, `u` or `v`.
- 8-bit samples (`uint8`) of a three-axis array. Chunks are decoded whole, in order, and each picture is placed by the
  position the chunk's header gives, so frames that the stream stores out of order (B-frames) come out in presentation order.
- Reading only: `encode` throws.

Not yet: 10-bit samples (`uint16`), audio formats, sharding.

## Long videos need the rectilinear chunk grid

A long video's chunks differ in length, so its arrays use the
[rectilinear chunk grid](https://github.com/zarr-developers/zarr-extensions/tree/main/chunk-grids/rectilinear).
`n5-zarr` 2.0.2 reads regular grids only; the rectilinear grid is proposed in
[saalfeldlab/n5-zarr#104](https://github.com/saalfeldlab/n5-zarr/pull/104). Until it is in a release, a video with
uniform chunks works as is, and a long one needs a build of that branch. The codec itself does not depend on the grid.

## Build and test

```
mvn test
```

The tests zarrify two small MP4/MOV files with refkv, read their planes through `n5-zarr`, and compare each plane's SHA-256
with what the FFmpeg command-line tool decodes the file to. They need `org.refkv:refkv-javacpp` (the zarrify
bindings) installed: `pixi run demo-install` in [refkv-javacpp](https://github.com/JaneliaSciComp/refkv-javacpp). The
fixtures in `src/test/resources/fixtures` are from refkv (MIT OR Apache-2.0).

## Licence

BSD-3-Clause OR Apache-2.0 (see `LICENSE-BSD-3-Clause`, `LICENSE-APACHE`), copyright Howard Hughes Medical Institute.

The JavaCPP FFmpeg build is LGPL v3 or later (built with `--enable-version3` and without `--enable-gpl`) and is linked
dynamically: an application that ships it follows the LGPL for FFmpeg.
