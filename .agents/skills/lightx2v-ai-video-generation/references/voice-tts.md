# LightX2V TTS And Voice Clone

This reference is for agents using LightX2V built-in TTS voices, cloned voices, or workflow `tts` nodes.

## Authentication

Use the same auth as generation tasks:

```bash
export LIGHTX2V_API_KEY="apikey_..."
export LIGHTX2V_BASE_URL="https://x2v.light-ai.top"
```

For local servers behind a proxy:

```bash
export NO_PROXY=127.0.0.1,localhost
export no_proxy=127.0.0.1,localhost
```

Never print API keys in final answers, logs, or copied commands.

## Built-In Voice Discovery

Always list voices before generating speech. Voice display names are not enough; use the returned `VoiceType` and `ResourceID` together.

```bash
lightx2v voices --limit 10
lightx2v voices --version 2.0 --json
lightx2v voices --fields all --json
```

Important response fields:

| Field | Meaning | Use |
| --- | --- | --- |
| `Name` | Human-readable display name | Show to users |
| `VoiceType` | Built-in voice id | Pass as `--voice-type` or workflow `data.voiceType` |
| `ResourceID` | TTS resource/version | Pass as `--resource-id` or workflow `data.resourceId` |
| `Gender`, `Age`, `Languages`, `Emotions` | Selection metadata | Filter/select voice |

Rule: keep `VoiceType` paired with the `ResourceID` from the same voice row. Many 2.0 voices require `seed-tts-2.0`; relying on defaults can fail.

## Built-In TTS CLI

```bash
lightx2v tts \
  --text "你好，欢迎使用 LightX2V。" \
  --voice-type zh_female_vv_uranus_bigtts \
  --resource-id seed-tts-2.0 \
  -o speech.mp3
```

Optional controls:

```bash
lightx2v tts \
  --text "今天真开心！" \
  --voice-type VOICE_TYPE \
  --resource-id RESOURCE_ID \
  --context-texts "语气活泼、亲切，有轻微兴奋感" \
  --emotion happy \
  --emotion-scale 4 \
  --speech-rate 0 \
  --loudness-rate 0 \
  --pitch 0 \
  -o speech.mp3
```

CLI flags:

| Flag | Required | Notes |
| --- | --- | --- |
| `--text` | yes | Spoken text |
| `--voice-type` | yes | `VoiceType` from `lightx2v voices` |
| `--resource-id` | yes | `ResourceID` from the same voice row |
| `--context-texts` | no | Natural-language tone/emotion instruction; best for supported newer voices |
| `--emotion` | no | Emotion label when supported by voice |
| `--emotion-scale` | no | Integer intensity, default `3` |
| `--speech-rate` | no | Integer adjustment, default `0` |
| `--loudness-rate` | no | Integer adjustment, default `0` |
| `--pitch` | no | Integer adjustment, default `0` |
| `-o`, `--output` | no | MP3 path, default `speech.mp3` |
| `--json` | no | Machine-readable result |

Output is an MP3 file.

## Voice Clone CLI

Use a short, clean reference audio sample. If `--text` is omitted, the server tries ASR.

Create a clone:

```bash
lightx2v voice-clone create ./voice_sample.wav \
  --text "这是一段用于克隆音色的参考语音文本。" \
  --json
```

Create and save in one step:

```bash
lightx2v voice-clone create ./voice_sample.wav \
  --text "这是一段用于克隆音色的参考语音文本。" \
  --save-name "我的音色" \
  --json
```

Manage clones:

```bash
lightx2v voice-clone list
lightx2v voice-clone save SPEAKER_ID "我的音色"
lightx2v voice-clone delete SPEAKER_ID
```

Generate speech with a clone:

```bash
lightx2v voice-clone tts \
  --speaker-id SPEAKER_ID \
  --text "你好，这是克隆音色生成的声音。" \
  --style "正常" \
  --speed 1.0 \
  --volume 0.0 \
  --pitch 0.0 \
  --language ZH_CN \
  -o cloned.wav
```

Clone TTS output is a WAV file.

## Workflow `tts` Node

Workflow `tts` converts `in-text` to `out-audio`.

Ports:

| Port | Direction | Meaning |
| --- | --- | --- |
| `in-text` | input | Text to speak |
| `in-context-tone` | optional input | Tone/emotion instruction text |
| `out-audio` | output | Generated audio |

Preset voice node:

```json
{
  "id": "voice_tts",
  "tool_id": "tts",
  "name": "语音合成",
  "position": {"x": 430, "y": 120},
  "data": {
    "model": "lightx2v",
    "voiceTab": "ai",
    "speakerId": "",
    "voiceType": "ICL_uranus_zh_female_chengshuwenrou_tob",
    "resourceId": "seed-tts-2.0",
    "emotion": "neutral",
    "emotionScale": 3,
    "speechRate": 0,
    "loudnessRate": 0,
    "pitch": 0
  }
}
```

Clone voice node:

```json
{
  "id": "voice_tts",
  "tool_id": "tts",
  "name": "克隆音色合成",
  "position": {"x": 430, "y": 120},
  "data": {
    "model": "lightx2v",
    "voiceTab": "clone",
    "speakerId": "SPEAKER_ID"
  }
}
```

Common connections:

```text
text-input.out-text -> tts.in-text
text-generation.out-text -> tts.in-text
text-input.out-text -> tts.in-context-tone
tts.out-audio -> avatar-gen.in-audio
```

For a digital-human workflow, connect `tts.out-audio` to `avatar-gen.in-audio` and connect a portrait/video input to `avatar-gen.in-image` or `avatar-gen.in-video`.

## Choosing Between Preset And Clone

Use preset voice when:

- The user asks for a common style, language, gender, age, or emotion.
- Fast generation is more important than matching a specific person.
- No reference audio is available.

Use cloned voice when:

- The user provides a reference voice and asks to preserve speaker identity.
- The voice should be reused across multiple tasks.
- The user explicitly asks for voice cloning or custom speaker.

Do not clone without consent to use the reference voice.

## Common Failures

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Built-in TTS fails | Wrong `resource_id` for `voice_type` | Re-run `lightx2v voices --json`; use paired fields |
| `cloned voice required` in workflow | `voiceTab` indicates clone but `speakerId` is empty | Set `speakerId` or switch to preset fields |
| Clone create succeeds but save fails | Route/key permission or backend whitelist issue | Keep returned `speaker_id`; save later after route is enabled |
| Clone quality poor | Noisy/short reference or text/audio mismatch | Use clean 5-30s audio and pass matching `--text` |
| TTS audio generated but avatar fails | `tts.out-audio` not connected to `avatar-gen.in-audio` or avatar visual input missing | Check workflow connections |
| Local API call hits proxy | `127.0.0.1` went through proxy | Set `NO_PROXY` and `no_proxy` |

## Safety

- Do not expose API keys, bearer tokens, or signed URLs.
- Do not clone voices that the user has no right to use.
- Label cloned voices clearly when saving them.
