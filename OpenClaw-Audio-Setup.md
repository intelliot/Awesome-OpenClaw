Requirements:
- Linux
- 8 GB RAM

Features:
- Fully local

Solution: use `whisper.cpp` with OpenClaw CLI audio config.

It’s the best tradeoff of:
- lowest integration friction with OpenClaw (native `whisper-cli` path),
- solid accuracy at `small.en`,
- low memory footprint (Whisper.cpp lists `small` at ~852 MB RAM; `base` at ~388 MB).

**Why not Parakeet first:**
- `nvidia/parakeet-ctc-0.6b` is ~600M params and the main weight file is ~2.44 GB (repo files ~4.87 GB). On 8 GB RAM this is usually too tight once Python/runtime overhead is included (inference).
- `nvidia/parakeet-tdt_ctc-110m` is much lighter (~114M params, ~459 MB weight), but OpenClaw has no built-in Parakeet adapter, so you need a custom wrapper CLI.
- NVIDIA Riva itself is not a fit for this box size (support matrix lists minimum 16 GB system RAM).

## Recommended setup (fully local)

```bash
# 0) Use whichever CLI prefix exists
OC="openclaw"; command -v openclaw >/dev/null || OC="pnpm openclaw"

# 1) Install whisper.cpp
sudo apt-get update
sudo apt-get install -y git build-essential cmake ffmpeg libopenblas-dev
git clone https://github.com/ggml-org/whisper.cpp.git ~/whisper.cpp
cd ~/whisper.cpp
cmake -B build -DGGML_BLAS=1 -DGGML_BLAS_VENDOR=OpenBLAS
cmake --build build -j
./models/download-ggml-model.sh medium.en
```

8 GB RAM should be enough to comfortably run medium, which needs about 3 GB RAM.

With only 1-2 GB RAM, you can run small (which is half the size but 20% worse) then use: `./models/download-ggml-model.sh small.en`

```bash
# 2) Configure OpenClaw audio transcription to be deterministic + local only
$OC config set tools.media.audio.enabled true --json
$OC config set tools.media.audio.maxBytes 20971520 --json
$OC config set tools.media.audio.models '[{"type":"cli","command":"/home/you/whisper.cpp/build/bin/whisper-cli","args":["-m","/home/you/whisper.cpp/models/ggml-small.en.bin","-otxt","-of","{{OutputBase}}","-np","-nt","{{MediaPath}}"],"timeoutSeconds":90}]' --json
```

Replace `/home/you/...` with your real path.

```bash
# 3) Telegram channel setup + group behavior
$OC channels add --channel telegram --token "$TELEGRAM_BOT_TOKEN"
$OC config set channels.telegram.mediaMaxMb 20 --json
$OC config set channels.telegram.groupPolicy open --json
$OC config set 'channels.telegram.groups.*.requireMention' false --json
```

Telegram app side:
1. In `@BotFather`: `/setprivacy` → Disable  
2. Remove bot from group, then re-add it

```bash
# 4) Restart and verify
$OC gateway restart
$OC channels status --probe
$OC logs --follow
```

Send a Telegram voice note and confirm logs show CLI transcription activity.

## If you still want Parakeet
Use `parakeet-tdt_ctc-110m`, not `0.6b`, and wire it as a custom `type:"cli"` model that prints plain text. If you want, I can give you a ready-to-run `parakeet_transcribe.py` + exact OpenClaw config command.

## Sources
- [OpenClaw Audio node docs](https://docs.openclaw.ai/audio)
- [OpenClaw Media Understanding docs](https://docs.openclaw.ai/media-understanding)
- [OpenClaw Telegram docs](https://docs.openclaw.ai/telegram)
- [whisper.cpp README (RAM + build)](https://github.com/ggml-org/whisper.cpp/blob/master/README.md)
- [whisper.cpp model size table](https://raw.githubusercontent.com/ggml-org/whisper.cpp/master/models/README.md)
- [faster-whisper README (CPU RAM benchmarks)](https://raw.githubusercontent.com/SYSTRAN/faster-whisper/master/README.md)
- [NVIDIA Parakeet 110m model card](https://huggingface.co/nvidia/parakeet-tdt_ctc-110m)
- [NVIDIA Parakeet 0.6b model card](https://huggingface.co/nvidia/parakeet-ctc-0.6b)
- [NVIDIA Riva support matrix](https://docs.nvidia.com/deeplearning/riva/user-guide/docs/support-matrix/support-matrix.html)
