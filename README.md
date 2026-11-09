# halfsize action

Compress video files to an **exact size** in CI, and get a measured quality report back.

The target size is the thing ffmpeg does not give you with one flag. `-crf` trades quality
and leaves the size wherever it lands; `-b:v` is a guess unless you do the arithmetic and
run two passes. This action does the arithmetic, picks the resolution that budget can carry,
runs both passes, then measures VMAF and SSIM against your source.

```yaml
- uses: YKLovePython/halfsize-action@v1
  with:
    files: artifacts/recording.mp4
    args: --to 20mb
```

Output is written next to the input as `recording.20mb.mp4`, and the quality score is in the
job log:

```
136 MB  →  19.45 MB   7.0x smaller   2.7% under target
VMAF 88.3   SSIM 0.971   slight softening
```

## Why in CI

Recorded demos, Playwright traces turned into video, screenshots stitched into an MP4, test
fixtures checked into git — all of them have a size that some downstream thing refuses. Doing
the compression in the job means the artifact you publish is the artifact that fits, and the
quality number is in the build log instead of in someone's memory.

## Inputs

| Input | Default | Notes |
|---|---|---|
| `files` | — (required) | Paths or globs, e.g. `videos/*.mp4`. Quotes are not preserved, so use a glob for several files. |
| `args` | `--to 20mb` | Anything halfsize accepts: `--to 10mb`, `--for discord`, `--no-audio`, `--codec h265`, `--json` |
| `version` | `0.1.9` | Pin it for reproducible builds, or `latest` |

## A fuller example

```yaml
name: compress

on: workflow_dispatch

jobs:
  compress:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: YKLovePython/halfsize-action@v1
        with:
          files: videos/*.mp4
          args: --to 25mb --json

      - uses: actions/upload-artifact@v4
        with:
          name: compressed
          path: videos/*.25mb.mp4
```

## Notes

- Runs on Linux, macOS and Windows runners (the composite steps use bash; the Windows image
  has it).
- No upload, no account, no API key. FFmpeg runs on the runner.
- If the target is impossible for the clip's duration, the step fails with the smallest size
  that duration can hold instead of writing a file that looks like mud.
- `--json` prints a machine-readable result if you want to read the size and VMAF in a later
  step.

The main project, with the guides and the arithmetic behind every number:
**[YKLovePython/halfsize](https://github.com/YKLovePython/halfsize)**

## License

MIT
