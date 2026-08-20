Drop the real sound effect files here, named exactly as below (referenced by [src/audio/soundManager.ts](../../src/audio/soundManager.ts)):

- `bugle.mp3` — plays when a modal/popup prompts or informs the player (turn end, reaction window, your-turn notice, order/redeploy/march-facing popovers)
- `musketry.mp3` — plays on Fire and Skirmish Fire resolution
- `march.mp3` — plays on March and Advance/Retire

Until these files exist, `playSound()` fails silently (caught) — no error, just no sound.
