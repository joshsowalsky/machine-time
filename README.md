# Machine Time

Hold your phone near something that repeats — a 3D printer laying down infill, a
washing machine, a treadmill — and this finds its rhythm, then builds music
locked to it.

**→ https://joshsowalsky.github.io/machine-time/**

One self-contained HTML file. No build step, no dependencies, no server, no
tracking, and nothing leaves the phone.

## How it works

**Finding the rhythm.** The page measures *spectral flux* from the microphone:
energy that newly appears between frames, rather than total loudness. A steady
hum contributes nothing, so only the machine's hits register. That signal is
binned to a fixed 10 ms envelope, then autocorrelated — slid over itself to
find the shift at which it best matches itself. That shift is the repeat
interval.

**Choosing a tempo.** A printer's interval is rarely a musical tempo on its
own, so the page picks a tempo related to it by a simple whole number — two
beats per pass, three, one every other pass. Simple ratios only: at 9/4 the
beat precesses through the bar and there is nothing left to perceive.

**Making the music.** It is synthesized in Web Audio, not streamed. That avoids
the whole problem with the obvious approach: consumer streaming APIs give you
neither sample-accurate scheduling nor a variable playback rate, so a recorded
track can't be held in time with anything. Generating it means the lock is
exact by construction.

## Three decisions that look like details

**Noise suppression is switched off** — along with echo cancellation and auto
gain. Browser audio processing exists to remove steady mechanical sound, which
is exactly the signal here. Left on, it deletes what we came for.

**The microphone stops before playback starts.** Otherwise the phone hears its
own music and locks onto that instead, and iOS routes audio to the earpiece for
as long as any microphone is open. Stopping the track fixes both. The cost is
no continuous re-sync — acceptable, because a motor-driven rhythm drifts far
more slowly than, say, a windshield wiper.

**There is a half/double button.** Autocorrelation genuinely cannot distinguish
a period from twice that period. That is an ambiguity in the method, not a bug
to code around, so it gets a control instead of a workaround.

## Following the machine as it drifts

With **Keep following** on, the microphone stays open and the tempo tracks the
machine instead of being fixed at the moment it locked.

The obvious objection is that the phone will hear its own music. Telling the
two apart sounds like a source separation problem, and it isn't: we wrote the
music, so we know both its frequency content and the exact moment of every hit.
Two exclusions replace any amount of separation. Detection listens only between
2.5 and 5.5 kHz, where stepper whine and gantry noise live — hats move up to
8.5 kHz and every tonal voice is capped at 2.2 kHz while tracking, so the band
is almost ours alone before anything is subtracted. Snares straddle the lower
edge, so each is registered as it's scheduled and the detector ignores 90 ms
after it.

The real hazard isn't feedback. A *wrong* lock would hear its own wrong tempo,
confirm it, and stay wrong forever. So the loop never re-seeds, rejects
anything more than a quarter period out, adapts its period slowly, and is
fenced to ±5% of where it started. Simulating it caught this before it ever
ran: at the original settings, three stray observations were enough to walk the
estimate 6.4% off, out of the correction window, killing tracking silently and
permanently.

Default is on for Android and off for iPhone, because the platforms genuinely
differ — iOS routes output to the earpiece for as long as any microphone is
open. One tap either way, and the music never depends on tracking working.

## Where a model would fit, and where it wouldn't

Worth writing down, because "use AI for it" is the obvious suggestion and it is
mostly wrong here.

**Not for finding the rhythm.** Neural beat trackers are trained on music,
where the hard part is expressive timing and syncopation. A printer has none of
that — it's a near-perfectly periodic mechanical signal, which is the exact
case autocorrelation is optimal for. A model would be slower, larger, and
trained on the wrong problem.

**Not for generating the music.** MusicGen and similar would sound far richer
than these oscillators. But they need a GPU server, take seconds to minutes per
clip, and — the part that actually rules them out — cannot be asked for
sample-accurate tempo lock. You'd be back to generating audio and stretching it
onto the grid, which is the problem synthesis was chosen to avoid. Facebook's
MusicGen is also CC-BY-NC-4.0, so non-commercial only.

**Plausibly, for knowing what it is listening to.** Audio Spectrogram
Transformer fine-tuned on AudioSet runs in the browser through
transformers.js — [Xenova/ast-finetuned-audioset-10-10-0.4593](https://hf.co/Xenova/ast-finetuned-audioset-10-10-0.4593).
AudioSet's classes cover machinery, so it could identify roughly what kind of
device it's hearing and pick the musical character from that, rather than
leaving it to three buttons. It stays on-device, so the page remains free and
private. The cost is real though: the smallest quantised weights are about
51 MB and the full model 347 MB, against a page that is currently 37 KB and
loads instantly. That is the trade, and it hasn't been made.

## Origin

The timing core started as a Python experiment in phase-locking music to
windshield wipers. 3D printers turn out to suit it far better: infill passes
hold a steady rhythm for minutes, where wipers have an intermittent dwell and
jump speed whenever the driver moves the stalk.

## Licence

MIT.
