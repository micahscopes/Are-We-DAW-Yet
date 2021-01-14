# Design Document for the RustyDAW Project

# Overview
The RustyDAW project is a community-driven effort to create a set of modular open-source software libraries that make it easier for end-users of the libraries
to create a Digital Audio Workstation (DAW). We believe Rust to be the perfect language for this because if its safety and modularity features.

# Context
We believe the current state of audio software development is slow, bogged down by old technology, and standards development is too closed. A DAW is a unique project in that it can drive change for many different technologies and if it gains enough traction, define new standards. It's important for that work to be done in the open, to avoid the mistakes of other technologies and to accelerate the pace of innovation. 

We believe the current state of open-source DAWs to be lacking. This is due to a number of reasons.
- DAWs are inherently complex. Many topics and factors must be taken into account and work together. With Rust, we can easily create separate modules that focus
on indiviual topics, and have those modules fit nicely with eachother.
- Existing DAWs rely on old technologies and standards such as C++. Rust can create a more maintainable code base while creating safe abstractions over these
existing standards such as VST and LV2.
- People have different tastes on workflow. Having a non-modular environment locks projects into a specific use-case. With modularity, a whole ecoystem of DAWs
with different workflows can be acheived.

# Goals and Non-Goals
## Goals
- Identify the core algorithms that can be shared across separate DAW projects.
- Identify key issues and learn from the mistakes of past open-source projects.
- Create the individual modules needed to create the minimum-viable product.
- Work together to create an example minimum-viable DAW.
- Implement more features from there as we like.

## Non-Goals
- This project will not deal with specific UI design details, only broad design concepts and reusable widgets.
- Every file and plugin standard does not need to be implemented for a minimum-viable product. Identify which standards are absolutely necessary first.
- Specific plugins like synthesizers, filters, delay, reverb, etc, should not be included in the minimum-viable product. Only focus on the core backend-algorithms
for now.

# Proposed Solution
## First steps
We must get these issues sorted out to avoid massive headaches later on.
- [x] 1. Strive to use GPLv3 for all RustyDAW projects unless it is agreed to use MIT (i.e. custom plugin formats).
- [ ] 2. Decide how to handle time. [`Ardour`] is a good inspiration here.
  - Identify the core algorithms needed to create this time-keeping algorithm.
- [ ] 3. Decide how to handle tempo and time signatures.
  - Identify the core algorithms needed to create this algorithm.
- [ ] 4. Decide what needs to be done to get a timeline to communicate audio and midi information to the audio thread.
  - This should strive to be as generic as possible. Possible factors to include:
    - Telling the audio thread which portion of audio clips should be filled in the buffer at a specific place. This will include information from
    the previously-described time-keeping algorithm.
    - Telling the audio thread which midi notes to store in its buffer. This will probably include information from the tempo and time-keeping
    algorithms.
    - An API that lets the end-user create custom messages to send to their audio thread with time information included. (i.e. fades on audio clips).
- [ ] 5. Decide how audio should be routed. Reaper is a good inspiration here.
  - Identify the core algorithms needed to create this audio-routing algorithm.

## Second step - modules
The modules will be separated into six separate crates. Each of these can be worked on in parallel once the first steps of been taken.

`rusty-daw-algorithms` - Core algorithms that are shared between crates and DAWs. This will include the algorithms identified in the first steps. Possible
algorithms include:
  - [ ] gain
  - [ ] de-clicking
  - [ ] summing
  - [ ] scene graph
  - [ ] panning
  - [ ] dithering
  - [ ] resampling
  - [ ] time-stretching
  - [ ] accurate time-keeping
  - [ ] tempo & time signatures
  - [ ] multithreading
  - [ ] swing tempo
 
 `rusty-daw-io` - A cross-platform IO library for connecting to system inputs/outputs and MIDI devices. We will focus only on JACK for now, and possibly
 ASIO on Windows if Windows doesn't support JACK.
 - [ ] JACK audio
 - [ ] JACK MIDI
 
 `rusty-daw-widgets` - A set of reusable widgets for the `tuix` GUI library.
 - [ ] peak/rms meters
 - [ ] waveform rendering
 - [ ] envelope rendering
 - [ ] oscilloscope (Possibly not minimum-viable)
 - [ ] spectrometer (Possibly not minimum-viable)
 
 `rusty-daw-plugin-host` - An abstraction library that handles hosting multiple plugin standards. Only focus on the minimum-viable product for now.
 - [ ] vst2
 - [ ] lv2 (Possibly not minimum-viable)
 
 `rusty-daw-encode` - A library for encoding/decode from several audio file formats. Look at potential existing libraries for this.
 - [ ] wav
 - [ ] resampling while reading
 - [ ] resampling and dithering while rendering
 
 `rusty-daw-midi` - An abstraction library that handles several different MIDI standards.
 - [ ] possibly define our own MIDI standard that can translate to other standards
 - [ ] midi
 - [ ] MPE (Possibly not minimum-viable)
 
 ## Third step
 - [ ] Work together to create an example minimum-viable DAW.
 
 ## Fourth step
 - [ ] Hooray, we did it! Decide which features to focus on next.

[`Ardour`]: https://ardour.org/timing.html
