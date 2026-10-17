---
name: minimax-h3-colab
description: Create short MiniMax H3 first-frame or reference-to-video clips from local images through Google Colab CLI. Use when a user asks for image-guided H3 video generation or a sequential batch of clips.
---

# MiniMax H3 on Colab

Use the bundled notebook and `scripts/runner.py` to generate MiniMax H3 video from local images. Jobs default to Ref2VA; choose `first_frame` for FL2VA first-frame conditioning. The Colab CLI must be installed and authenticated with an available compute-unit balance and GPU allocation.

## Workflow

1. Inspect the requested image(s) and any supplied prompt file. `reference` accepts 1–9 ordered images mapped to `<Picture 1>` through `<Picture 9>`. `first_frame` requires exactly one image and uses FL2VA first-frame conditioning.
2. If the user supplied a prompt file, pass it through unchanged. Otherwise, for `reference`, write a complete Ref2VA prompt with `subject_definitions`, `summary`, `retention_analysis`, `detailed_description` shot blocks, `overall_soundscape`, and `non_diegetic_music`; include dialogue in its shot with `<d>[Chinese] ...</d>` when requested. Use the bundled runner's `compose_ref2va_prompt` helper for structured shot input. For `first_frame`, describe motion and shots from the supplied opening image using FL2VA prompt conventions.
3. Check Colab access and balance with `python3 scripts/runner.py usage --json`. Sufficient compute units do not guarantee that Colab will allocate the requested GPU or high-memory runtime. Report allocation failures plainly.
4. Put the jobs in a UTF-8 JSON manifest and run one batch. The runner uploads each job's images and prompt, executes the bundled notebook sequentially on the same live session, downloads each MP4, and stops the session when it created it. If reusing a session, pass `--stop-on-complete` when the user wants it stopped after the batch.
5. Confirm each output exists and report its local path. Preserve completed clips if a later job fails.

Example manifest:

```json
{
  "jobs": [
    {
      "title": "intro",
      "inference_mode": "reference",
      "reference_images": ["/absolute/path/girl.jpg"],
      "prompt_file": "/absolute/path/prompt.txt",
      "width": 1344,
      "height": 768,
      "duration_seconds": 12,
      "output_name": "intro"
    }
  ]
}
```

Omit `inference_mode` to keep the compatible `reference` default. Width and height default to 1344×768; each must be a positive multiple of 32 and their area must not exceed 1344×768 pixels. A first-frame job must have exactly one reference image.

Run from this skill directory:

```bash
python3 scripts/runner.py batch --manifest /absolute/path/jobs.json --gpu A100 --timeout 10800 --output-dir /absolute/path/outputs
```

For a single FL2VA clip, pass `--mode first_frame` and one `--image` to `scripts/runner.py single`; optionally set `--width` and `--height`. The shell launcher accepts the same options.

Use `--no-high-mem` when high-memory allocation is unavailable or not desired. For a named existing session, add `--session SESSION --stop-on-complete` if the requested workflow should end that session.

On Windows hosts, run the Colab CLI and this runner inside WSL, where the CLI is installed and authenticated.

## Operational limits

- Each job requires 1–9 non-empty images, a non-empty UTF-8 prompt, and a duration from 4–15 seconds. `first_frame` requires exactly one image.
- Keep all jobs in one batch to reuse the same Colab session and loaded ComfyUI/model state.
- Do not blindly retry a timed-out `colab exec`: the remote kernel may still be working. The runner stops the session during cleanup and records completed jobs before reporting the failure.
- Do not expose OAuth credentials or runtime tokens in prompts, manifests, browser state, or logs.
- The bundled `assets/MiniMax_H3_Turbo_Colab.ipynb` is the inference notebook used by the runner.
