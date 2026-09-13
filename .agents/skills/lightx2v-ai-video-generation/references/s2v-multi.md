# Multi-Person And Multi-Role S2V

Use this reference when several people appear in one image and one or more selected
people should lip-sync to assigned audio. This includes a group image where only one
person should speak. The request uses one group image and a directory-style
`input_audio`; ordinary `--audio` is the single-speaker path and cannot select a person
inside a group image.

Discover live models first with `lightx2v models --json`. Prefer
`SekoTalk-V3-Multi` when it is available. `SekoTalk-V3` is the single-speaker member
of that family. Older `SekoTalk` / `SekoTalk-V2.7` deployments may expose multi-role
support, but do not assume model support without live metadata.

FFmpeg can draw a mask after a person rectangle is known and can silence audio after
speaker time ranges are known. It does not detect people, identify speakers, or prove
which track belongs to which person. When the user provides only mixed audio, first
derive anonymous speaker ranges with an available diarization tool, transcription plus
listening, or user-provided timings. Use visual inspection and user confirmation to
bind those anonymous tracks to people in the image.

## Required Package

A two-object package looks like this:

```text
multi-role-audio/
├── config.json
├── original_audio.mp3
├── p1.wav
├── p1_mask.png
├── p2.wav
└── p2_mask.png
```

```json
{
  "talk_objects": [
    {"audio": "p1.wav", "mask": "p1_mask.png"},
    {"audio": "p2.wav", "mask": "p2_mask.png"}
  ]
}
```

`talk_objects` may contain one object. Use that form when several people are visible
but only one selected person should speak:

```json
{
  "talk_objects": [
    {"audio": "p1.wav", "mask": "p1_mask.png"}
  ]
}
```

Each entry is one binding: its audio drives only the person selected by its mask.
Keep every referenced file at the package root and use plain filenames in
`config.json`; do not use absolute paths, `..`, or nested paths. Include exactly one
`original_audio.*` containing the final mixed soundtrack. Prefer to trim it to the
same target duration as the role tracks.

## 1. Identify And Review Person Regions

Read the source dimensions:

```bash
ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height \
  -of default=noprint_wrappers=1 ./group.png
```

Inspect the source image, name each relevant person, and record an absolute-pixel
rectangle `[x1, y1, x2, y2]`. The rectangle should cover the complete person region
that should be driven, not only the mouth. Clamp it to the image bounds and avoid
overlap with other people where practical. Compute `BOX_W=x2-x1` and `BOX_H=y2-y1`.

Create a mask with the exact source width and height. The selected rectangle is pure
white and every other pixel is pure black:

```bash
ffmpeg -y -f lavfi -i color=c=black:s=WIDTHxHEIGHT \
  -vf "drawbox=x=X1:y=Y1:w=BOX_W:h=BOX_H:color=white:t=fill,format=rgb24" \
  -frames:v 1 ./multi-role-audio/p1_mask.png
```

Create a review image showing the same rectangle on the source:

```bash
ffmpeg -y -i ./group.png \
  -vf "drawbox=x=X1:y=Y1:w=BOX_W:h=BOX_H:color=red@0.9:t=5" \
  -frames:v 1 ./p1_bbox_review.png
```

For a one-object package, only that person's region is white. Other people in the
image remain black and should not be driven. If a person is obscured, overlaps another
person, or cannot be bounded confidently, stop and ask the user for guidance or use
Free Mode. Do not infer a rectangle merely from left-to-right order.

Verify every mask before continuing:

```bash
ffprobe -v error -select_streams v:0 \
  -show_entries stream=width,height,pix_fmt \
  -of default=noprint_wrappers=1 ./multi-role-audio/p1_mask.png
```

## 2. Build Role Tracks Without Moving Speech

First make a speaker timeline. If `pyannote-audio` and its pretrained pipeline are
already available, it can diarize a mixed recording without changing the source:

```bash
pyannote-audio apply pyannote/speaker-diarization-community-1 \
  ./multi-role-audio/original_audio.wav \
  --into ./multi-role-audio/diarization.rttm \
  --device auto
```

The pipeline may require model access or a prior download. Do not assume it is
installed, install dependencies, upload private audio, or obtain external credentials
without the user's permission. If it is unavailable, use a role-labeled script,
transcription plus careful listening, user-provided timings, or Free Mode.

Each RTTM `SPEAKER` row contains the start time in column 4, duration in column 5,
and an anonymous label such as `SPEAKER_00` in column 8. Compute
`end = start + duration`, group rows by label, and review short fragments and overlaps
before treating them as speech ranges. Diarization labels are arbitrary clusters:
`SPEAKER_00` does not mean the first, leftmost, male, or primary person in the image.

Summarize the resulting timeline before making role tracks:

| Anonymous speaker | Speaking ranges |
| --- | --- |
| `SPEAKER_00` | `0.5-3.2s`, `8.0-10.1s` |
| `SPEAKER_01` | `3.3-7.9s` |

Create a full-timeline track for each role. Preserve that role's original speech
positions and silence everything else:

```bash
ffmpeg -y -i ./multi-role-audio/original_audio.mp3 \
  -af "volume=volume=0:enable='not(between(t,0.5,3.2)+between(t,8.0,10.1))'" \
  -ar 44100 -ac 1 -c:a pcm_s16le ./multi-role-audio/p1.wav
```

Do not concatenate a role's speech clips end-to-end: doing so changes the dialogue
timeline and makes lip motion occur at the wrong times. Role tracks should share one
timeline and target duration. Prefer PCM signed 16-bit WAV at 44.1 kHz, mono. If the
source must be trimmed to an account or model duration limit, apply the same start,
end, and padding policy to the mixed soundtrack and every role track.

Probe and listen to every result:

```bash
ffprobe -v error -show_entries stream=codec_name,sample_rate,channels \
  -show_entries format=duration -of default=noprint_wrappers=1 \
  ./multi-role-audio/p1.wav
```

The role track must contain only one anonymous speaker at the original times. Play it
back before assigning it to an image person. If there is crosstalk, overlapping speech,
an unknown speaker, inconsistent clustering, or uncertain segment boundaries, stop and
ask the user or use Free Mode. Diarization identifies time ranges, not clean source
separation; FFmpeg cannot remove another voice from an overlapping segment.

## 3. Mandatory User-Confirmation Gate

After masks and tracks exist, stop before producing `request.json`, quoting, or
submitting. Give the user all of the following:

1. The numbered source or each `pN_bbox_review.png`.
2. Every `pN_mask.png`.
3. Every playable `pN.wav` plus a short summary of its speech and time ranges.
4. One complete mapping table:

| Object | Person in image | Mask rectangle | Track | Speech summary |
| --- | --- | --- | --- | --- |
| `p1` | Left-side woman | `[x1,y1,x2,y2]` | `p1.wav` | ... |
| `p2` | Right-side man | `[x1,y1,x2,y2]` | `p2.wav` | ... |

For a one-object package, show one row and explicitly state that the other visible
people will not be driven.

Ask: **"Please confirm that every mask is paired with the correct role track."**

Proceed only after the user explicitly approves the complete displayed mapping. A
timeout, silence, file numbering, left-to-right ordering, or an ambiguous instruction
such as "use your judgment" is not confirmation. If the user corrects anything,
regenerate the affected artifact, show the complete table again, and ask again.

## 4. Encode The Directory Request

Only after confirmation, Base64-encode `config.json`, the mixed soundtrack, every
role track, and every mask. Directory values are raw Base64 strings without a
`data:*;base64,` prefix. `config.json` itself must also be encoded.

For a one-object package, Bash process substitution avoids placing large media in
shell arguments:

```bash
WIDTH=1920   # replace with ffprobe output
HEIGHT=1080  # replace with ffprobe output

jq -n \
  --argjson width "$WIDTH" \
  --argjson height "$HEIGHT" \
  --rawfile config <(base64 -w 0 ./multi-role-audio/config.json) \
  --rawfile original <(base64 -w 0 ./multi-role-audio/original_audio.mp3) \
  --rawfile p1_audio <(base64 -w 0 ./multi-role-audio/p1.wav) \
  --rawfile p1_mask <(base64 -w 0 ./multi-role-audio/p1_mask.png) \
  '{
    s2v_multi_role_mode: true,
    input_meta: {
      image: {width: $width, height: $height}
    },
    input_audio: {
      type: "directory",
      data: {
        "config.json": $config,
        "original_audio.mp3": $original,
        "p1.wav": $p1_audio,
        "p1_mask.png": $p1_mask
      }
    }
  }' > ./request.json
```

For more objects, add each `pN.wav` and `pN_mask.png` to both the `jq` inputs and the
`input_audio.data` map. The map filenames must exactly match `config.json`.

Submit with the existing CLI:

```bash
lightx2v run s2v/SekoTalk-V3-Multi \
  --image ./group.png \
  --input @request.json \
  --duration AUDIO_SECONDS \
  --prompt "Only the selected person speaks naturally" \
  -o ./result.mp4
```

`--image` supplies the original group image. Single-object and multi-object packages
use the same directory protocol. Set `input_meta.image.width` and `.height` from
`ffprobe`, not from the mask rectangle. `--duration` supplies
`input_meta.audio_seconds`; use the exact final mixed-track duration. These metadata
fields are required by S2V billing/quote validation even when `--image` and directory
audio contain the real media. Current CLI versions do not necessarily derive image
dimensions from `--image` automatically.

`lightx2v run --quote` quotes and then continues to submission, so do not invoke it
until after the user-confirmation gate. Quote and submit must use the same image
dimensions, audio duration, model, and directory payload.

## Common Failures

| Symptom | Cause | Action |
| --- | --- | --- |
| Wrong person speaks | Mask/track binding is wrong | Stop, correct artifacts, repeat user confirmation |
| Several people move for one track | Mask is too broad or overlaps roles | Tighten the white rectangle and review again |
| Speech timing shifts | Clips were concatenated | Rebuild a full-timeline track with silence |
| Track contains another speaker | Speaker ranges are wrong or overlap | Ask the user or correct in Free Mode |
| Diarization returns anonymous labels | Speaker clustering cannot identify image people | Play each track and confirm every person/track binding with the user |
| Diarization misses or merges speakers | Crosstalk, short turns, noise, or model error | Correct the timeline from listening/transcription or ask the user; do not guess |
| Quote says `input_meta.image width/height ... required` | CLI did not infer source dimensions | Add `input_meta.image.{width,height}` from `ffprobe` and pass the exact `--duration` |
| Request rejects directory input | Model lacks multi-role support or a file is absent | Refresh live models and compare all config filenames |
| Output has wrong/missing sound | `original_audio.*` is absent or mismatched | Include one final mixed soundtrack with the intended duration |
