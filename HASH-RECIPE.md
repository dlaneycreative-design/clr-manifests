# CLR Music hash recipe

How the two checksums in `clear-music-batches-1-6.json` are computed, so any
future batch can be deduped against the library exactly.

## sha256 (file hash)

SHA-256 of the packaged audio file's bytes, exactly as distributed. No
decoding, no transformation.

## pcm_sha256 (content hash, the manifest field `pcm_sha256`)

SHA-256 of the audio decoded to canonical PCM:

- Decode the packaged file as distributed (any supported input format).
- Output format: raw headerless PCM, signed 32-bit little-endian (`s32le`).
- Sample rate: 48,000 Hz.
- Channels: 2 (stereo).
- No normalization. No volume adjustment, no loudnorm, no trimming, no
  fades. A plain decode with resampling and channel conversion to the
  canonical format, nothing else.
- Hash the raw PCM byte stream with SHA-256. No WAV header, no metadata.

Reference command:

```
ffmpeg -v error -i <input-file> -f s32le -acodec pcm_s32le -ac 2 -ar 48000 - | sha256sum
```

This definition matches the batch packages' own documentation, which
describes the inspection basis as "48 kHz stereo signed 32-bit
little-endian PCM; comparison only" and notes the distributed audio is
never substituted or altered.

Verification: on 2026-10-18 this exact command was run against a packaged
batch 6 file (cm6-03ff4da71e550bff97e3, Coins Single 26) and reproduced
the manifest's pcm_sha256 byte for byte (b037b16153b3d20f...), and the
file's plain sha256 likewise matched the manifest's sha256.

Notes:

- Two files with the same pcm_sha256 are the same recording content, even
  if the wrappers (mp3, ogg, wav) or file bytes differ.
- Two files with different pcm_sha256 can still be perceptually similar;
  this is exact-audio deduplication, not similarity screening.
- Lossy decoders can differ in principle at the margins between versions;
  the values in the manifest come from the original packaging pipeline,
  and the recipe above reproduced them exactly with a stock ffmpeg build.
- Batch 1's 500 starter records have blank creator fields. They are CC0
  and the credit is genuinely unknown on our side; the blank is preserved
  deliberately. Do not backfill a guessed creator.
