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

## Origin

The timing core started as a Python experiment in phase-locking music to
windshield wipers. 3D printers turn out to suit it far better: infill passes
hold a steady rhythm for minutes, where wipers have an intermittent dwell and
jump speed whenever the driver moves the stalk.

## Licence

MIT.
