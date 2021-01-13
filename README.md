# Are We DAW Yet?
A checklist of features needed to create a fully-featured Digital Audio Workstation (DAW) in the Rust programming language.

Join our community at the [`Rust Audio Discord`]!

# Important first steps:
We must get these right to avoid massive headaches later.
- [x] Single project vs modular. - Aim to create a modular system of repositories rather than one monolithic project.
- [x] Licensing - Use GPLv3 for all RustyDAW projects (except for any custom plugin formats).
- [ ] Decide how audio should be routed. Reaper is a good inspiration here.
- [ ] Decide how to handle time. Look at Ardour for inspiration.
- [ ] Decide how to handle tempo and time signatures.

# Modular DAW System Checklist

## Algorithms - [`rusty-daw-algorithms`]
- [x] gain & smooth gain
- [ ] summing
- [ ] scene graph
- [ ] panning
- [ ] dithering
- [ ] resampling
- [ ] time stretch
- [ ] sample-accurate timer
- [ ] multithreading
- [ ] swing-tempo

## IO - [`rusty-daw-io`]
- [ ] JACK audio
- [ ] JACK MIDI
- [ ] ALSA audio
- [ ] ALSA MIDI
- [ ] ASIO
- [ ] Generic Windows IO
- [ ] CoreAudio

## UI - [`rusty-daw-widgets`]
- [x] UI library - [`tuix`]
- [ ] peak/rms meters
- [ ] oscilloscope
- [ ] spectrometer
- [ ] waveform rendering
- [ ] envelope rendering

## Plugin hosting - [`rusty-daw-plugin-host`]
- [ ] lv2 plugin hosting
- [ ] vst2 plugin hosting
- [ ] vst3 plugin hosting
- [ ] auv2 plugin hosting
- [ ] auv3 plugin hosting?
- [ ] our own custom plugin format?
- [ ] plugin sandboxing?

## Sound file formats - [`rusty-daw-encode`]
- [ ] wav
- [ ] ogg
- [ ] flac
- [ ] mp3
- [ ] aiff

## MIDI - [`rusty-daw-midi`]
- [ ] midi reader/writer
- [ ] midi2 reader/writer
- [ ] MPE support

## Housekeeping
- [ ] website?

# Nice-to-have Features

## Generators
- [ ] one-shot sampler
- [ ] test tone synth
### Lower-priority
- [ ] sf2/sfz multisampler
- [ ] multi-drum sampler
- [ ] basic subtractive synthesizer
- [ ] basic additive synthesizer
- [ ] basic drum synthesizer
- [ ] wavetable synthesizer
- [ ] granular synthesizer
- [ ] drum slicer

## FX
- [ ] gain & pan
- [ ] LP/BP/HP filter
- [ ] parametric EQ
- [ ] basic compressor
- [ ] basic limiter
- [ ] basic waveshaper/clipper
- [ ] basic delay unit
- [ ] stereo width adjustment tool
- [ ] multi-band splitter
- [ ] mid-side splitter
- [ ] patching plugin
- [ ] convolution plugin
- [ ] basic reverb
- [ ] delay sound by samples/time
- [ ] phase offset
- [ ] chorus
- [ ] phaser/flanger
- [ ] comb filter
### Lower-priority
- [ ] analouge-modeled EQ
- [ ] analouge-modeled compressor
- [ ] multiband compressor
- [ ] multiband distortion
- [ ] bus compressor
- [ ] look-ahead mastering limiter
- [ ] dynamic EQ
- [ ] linear-phase EQ
- [ ] tape distortion
- [ ] advanced delay unit
- [ ] pedal/guitar effects
- [ ] advanced reverb
- [ ] shimmering reverb
- [ ] vocoder
- [ ] pitch correction tool
- [ ] desser

## MIDI FX
- [ ] LFO
- [ ] transposer
- [ ] multi-note/chords
- [ ] basic arpeggiator

# Inspiration Resources
- https://www.cockos.com/wdl/

<img src="/images/dank_meme.jpg">

[`tuix`]: https://github.com/geom3trik/tuix
[`iced`]: https://github.com/hecrj/iced
[`egui`]: https://github.com/emilk/egui
[`Rust Audio Discord`]: https://discord.com/channels/590254806208217089/590283781806620672
[`rusty-daw-algorithms`]: https://github.com/RustyDAW/rusty-daw-algorithms
[`rusty-daw-io`]: https://github.com/RustyDAW/rusty-daw-io
[`rusty-daw-plugin-host`]: https://github.com/RustyDAW/rusty-daw-plugin-host
[`rusty-daw-encode`]: https://github.com/RustyDAW/rusty-daw-encode
[`rusty-daw-midi`]: https://github.com/RustyDAW/rusty-daw-midi
[`rusty-daw-widgets`]: https://github.com/RustyDAW/rusty-daw-widgets
