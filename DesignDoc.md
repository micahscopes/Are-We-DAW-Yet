# RustyDAW Design Document

RustyDAW is a collection of tools for creating Digital Audio Workstations (DAWs).

RustyDAW is a community-driven project. If you are interested in participating, feel free to join the [RustAudio Discord](https://discord.com/invite/8rPCp9Q), channels `#rusty-daw` and `#rusty-daw-gui`.


# Objective

*TL;DR: we want a solid, open-source, more-modular audio ecosystem, and we think Rust enables us to provide it.*

We believe the current state of audio software development is slow and bogged down by old technology, and the standards too closed. A DAW is a unique project: it's a large and complex one, but if successful, it can drive change for many different technologies and define new standards. It's important for this work to be done **in the open** to avoid the mistakes of other technologies/standards, and to accelerate the pace of innovation.

Why not contribute to an open-source DAW that already exists? We believe that the current state of open-source DAWs is lacking:
 - DAWs are inherently complex. Many different concepts must be taken into account simultaneously, and work together.
 - Existing DAWs rely on older technologies and standards, such as C++. While fine for the task of writing audio software, C++ makes it trickier to create a maintainable codebase (cross-platform support, ease of writing modular code, etc.).
 - People have different tastes in workflow, but having a monolithic DAW architecture locks projects into a specific use-case. With a **more modular** design, developers can more easily swap out components/libraries to suit the needs of *their* particular use-case, and a whole ecosystem of DAWs with different workflows can be acheived.

We believe Rust to be the perfect language for this project because of its design philosophy:
 - No garbage collection, so still (more easily) audio-safe
 - Cross-platform by default: works on Windows, Mac OS, and Linux, across multiple different CPU architectures, without much compilation fuss
 - The modules and crates system makes it easy to split your code into distinct modular components, and `cargo` handles all of the compilation for you
 - Rust's safety guarantees will significantly reduce the occurrence of DAWs crashing


# Scope for MVP (minimum viable product)

While our long term goals are to grow into a fully-featured DAW with a featureset that competes with existing commercial DAWs like FL, Live, and Bitwig, we need to limit the scope to achieve minimum viable product (mvp) release.

### Goals
Note these goals are for a specific software application that will be the "flagship" of the RustyDAW project. However, the backend engine will be designed to allow any developer to easily use the same engine for whatever GUI interface/workflow they wish to create.

* Cross-platform (Mac, Windows, and Linux)
* Multi-track timeline with audio clips. Audio clips can added, moved, removed, sliced, and copied freely around the timeline
* Load wav, aac, flac, mp3, pcm, and ogg vorbis files as audio clips (afforded to us by the [`Symphonia`] crate)
* Export to wav file
* Realtime effects on audio clips (MVP will include gain, crossfade, pitch shift (doppler), and reverse)
* Robust audio graph
* Grid snapping controls on timeline
* Tempo selection
* Loop range on timeline
* Control clips on the timeline (like MIDI / automation clips). MVP features will include:
    * Clips can be added, moved, removed, sliced, and copied freely around the timeline
    * Drop-down to select MIDI CC that this "automation node" will be attached to.
    * Looping options for control clips
* Piano roll editor for control clips. MVP features will include:
    * Select scale (12 EDO by default)
    * Paint, move, remove, and resize notes in the piano roll (FL style)
    * Grid snap with ability to select grid size (bar, 8th note, 16th note, triplet, etc)
    * Button/keyboard shortcut for simple quantization of notes to the grid
    * A bitwig-like per-note velocity/pan editor on the bottom
* Track headers on each track in the timeline. MVP features will include:
    * Track name. User double-clicks to rename.
    * Fader & pan controls
    * Solo, Mute, and arm record buttons
    * Change color of header/track for organization
    * Drop-down to select where to route the track in the mixer (master or effect bus)
    * Ability to select whether the track headers appear to the left or right side of the timeline
* Mixer view. MVP features will include:
    * Use the same names and track colors as the track headers for each track
    * Add/remove effect busses
    * Rename effect busses
    * Fader & pan controls
* Host VST2 plugins (GUI prefered, but not mvp).
* Built-in MVP plugins will include
    * A basic gain & pan plugin
    * A basic digital clipping plugin
* Settings
    * Interface for selecting audio hardware input/outputs
    * Interface for selecting MIDI devices. Note that we only need to support basic MIDI keyboard functionality for mvp.

### Non-Goals
To keep the scope manageable with such a small team, we will NOT focus on these features for the mvp:

* Export mp3, ogg vorbis, aac, etc.
* Audio clip effects such as time stretching and non-doppler pitch shifting
* Streaming audio files from disk
* Recording audio or MIDI data
* Audio-clip level automation of pitch
* Send effect routing
* Sidechain routing to plugins
* Grouped tracks
* Non-4/4 time signatures and time signature changes
* Project tempo automation
* Curved automation nodes on control clips (only linear automation for now)
* Quantization features in piano roll
* Per-note modulation
* Edit/view multiple control clips at once in piano roll
* Custom time marks & chord view on timeline
* Sample browser (but do support dragging and dropping from system folder)
* Plugin browser
* Preset browser
* "Live" style clip launcher
* A full suite of built-in synth and effect plugins
* An "FL Patcher"-like system
* A "Live/Bitwig" style of grouping together effects and splitting them by mid/side, multiband, etc.
* Custom application themes
* Hosting LV2, VST3, and AU plugins
* Plugin sandboxing
* Advanced loop recording
* Swing tempo


# Backend Design (MVP)


## Time Keeping

Keeping accurate track of time of events is crucial. However, automated tempos, swing tempos, and different audio device samplerates poses a challenge. The proposed solution lies in the [`rusty-daw-time`] repo.

The crate is designed as such:

### Intended workflow:
1. The GUI/non-realtime thread stores all events in `MusicalTime` (unit of beats).
2. The GUI/non-realtime thread creates a new `TempoMap` on project startup and whenever anything about the tempo changes. It then sends this new `TempoMap` to the realtime-thread.
3. When the realtime thread detects a new `TempoMap`, all processors with events stored in `MusicalTime` use the `TempoMap` plus the `SampleRate` to convert each `MusicalTime` into the corresponding discrete `SampleTime` (or sub-samples). It keeps this new `SampleTime` for all future use (until a new `TempoMap` is recieved).
4. When playback occurs, the realtime-thread keeps tracks of the number of discrete samples that have elapsed. It sends this "playhead" to each of the processors, which in turn compares it to it's own previously calculated `SampleTime` to know when events should be played.
5. Once the realtime thread is done processing a buffer, it uses the `TempoMap` plus this `SampleRate` to convert this playhead into the corresponding `MusicalTime`. It then sends this to the GUI/non-realtime thread for visual feedback of the playhead.
6. When the GUI/non-realtime thread wants to manually change the position of the playhead, it sends the `MusicalTime` that should be seeked to the realtime thread. The realtime thread then uses it, the `TempoMap`, and the `SampleRate` to find the nearest (floored) sample to set as the new playhead.

### Time Keeping Data types:
- `MusicalTime` - unit of "musical beats" - internally represented as an `f64`
- `Seconds` - unit of time in seconds - internally represented as an `f64`
- `SampleTime` - unit of time in number of discrete samples - internally represented as an `i64`
- `TempoMap` - a map of all tempo changes in the project (like an automation track) - (internal representation to be decided)


## Hardware I/O

We will store this functionality in the [`rusty-daw-io`] repo.

We will likely use [`cpal`] for mvp. However, it should be noted that we may not use it forever. There are two huge issues we take with [`cpal`]:
* It returns a list of available devices as an array of all possible configurations of devices, instead of just listing the supported options per-device.
* Its architecture does not support duplex audio well. As such, mvp will only support audio output with no input.

After mvp, we will look into either improving our own fork of cpal, or start our own cpal-like crate.

* Create an easy-to-use interface that any GUI system can use to present available devices to the user, and then select and apply those settings. Whenever "apply" is selected, we will take the easy route and restart the whole audio engine (as opposed to trying to seamlessly update the existing one).
* Spawn a realtime callback closure with all input/output buffers neatly packaged in the arguments. Cpal already has done most of the work for us here, but there are known issues with syncing input and output buffers together. As such, mvp will only support audio output, not input.
* Add detection and connecting to MIDI devices. This will be the most difficult part, as `cpal` does not handle this. For mvp, we only need to focus on simple MIDI messages like note on/off velocity, and CC controls.


## Audio Graph

An audio graph is an algorithm that takes individual "nodes" of audio/control processors and arranges them in the order they should be processed. The audio graph may also provide information on how to best utilize multi-threading on the CPU (although multithreading is not mvp).

For mvp, there will be these types of nodes in the graph:
* `TimelineTrack` - A single track in the timeline that outputs audio and control buffers (before any effects except for internal audio-clip effects). See the "Timeline Engine" section below for more details.
* `InternalPlugin` - A single internal effect or synth plugin
* `VST2Plugin` - A single VST2 plugin
* `GainNode` - A node that applies gain onto a signal. This will use de-clicking strategies.
* `PanNode` - A node that applies panning onto a signal. This will use de-clicking strategies.
* `DelayNode` - A node that applies delay compensation onto a signal
* `SumNode` - A node that sums signals together. This will use de-clicking strategies.

## Sampler Engine

We will store this functionality in the [`rusty-daw-sampler`] repo.

This will act as a sampling engine for playing audio clips (whether on a timeline or a clip launcher), as well as any future sampling plugins. 

The proposed interface will look like this:

* Ability to create a `PCMResource` type, which is an *immutable* [`basedrop`] smart pointer to raw samples in memory. The immutability reflects the non-destructive nature of this engine. This type will also store the sample-rate of the data. These raw samples provided by the user of this crate should be:
    * Uncompressed (this crate will not handle encoding/decoding)
    * De-interleaved (each channel in its own buffer)
    * Preferably in the original bit depth (this crate *will* handle automatically converting to f32/f64 on the fly)
    * Preferably in the original sample rate, regardless of the project's sample rate. A key design of this sampling engine is the ability to only need one resampling pass to do any type of effects like pitch-shifting or time stretching (as opposed to two or more if the data is converted to the project's sample rate on load, which will have a noticeable drop in quality)
* An enum `ShiftMode` (non-exhaustive) that contains the following options:
    * `None` - No pitch shifting/time stretching.
    * `Doppler(f64`) - Speed up/slow down the effective playback speed by this factor. It is up to the user to calculate the value they need to pass in.
    (Goals after mvp include time stretching, pitch *and* time streching, formant shifting, and clip-level automation on each of these)
* An enum called `InterpQuality` (interpolation quality, non-exhaustive)
    * Optimal2x - The "optimal 2x" algorithm as described in the [`deip`] paper
    * Linear
* A method that fills rendered samples into a given buffer. This method will have the following arguments:
    * An *immutable* reference to a `PCMResource`. The immutability reflects the non-destructive nature of this engine.
    * `buffer: &mut[f32]` - the buffer of samples to fill (single channel)
    * `channel: u16` - the channel of the `PCMResource` to use
    * `frame: SampleTime(i64)` - the frame in the `PCMResource` where the copying will start. Note this may be negative. The behavior of this method should be to fill in zeros wherever the buffer lies outside of the resource's range.
    * `sub_frame: f64` - the fractional sub-sample offset to add to the previous `frame` argument. This engine handles interpolation.
    * `shift_mode: ShiftMode` - described above
    * `interp_quality: InterpQuality` - described above
    * `reverse`: whether or not the audio should be reversed
* A similar method as above, but optimized for stereo signals

While streaming samples from disk is an eventual goal of this crate, it will not be part of the mvp.


## Control Data

We will like eventually use our own custom control spec instead of standard MIDI for our internal plugins/future custom plugin format. This is for the following reasons:

* The MIDI standard doesn't support all the features we want like:
    * Non 12-TET scales
    * Per-note modulation of pitch, volume, pan, etc (This is afforded by the MPE extension, though)
    * Fine control over sample-accurate audio-rate modulation
    * MIDI wants to take control of aspects we believe should be left to the DAW/plugin spec instead, such as plugin parameters.
* The MIDI2 standard is more promising, but there are several issues we take with it:
    * The spec is huge and complicated.
    * We would have to rely on a third party organization to include any potential features in the future.
    * It heavily prioritizes being fully backwards compatible with MIDI. We have some doubts on how acheivable this actually is.
    * MIDI2 wants to take control of aspects we believe should be left to the DAW/plugin spec instead, such as plugin parameters.
    * MIDI2 is new and has little adoption. We have no idea if it will even be successful in the long run.

However, it is unclear at the moment of what a spec should look like. So for MVP, we will only focus on how the DAW stores control information and how it assembles this information into MIDI for use with VST2 plugins. If we succesfully do this, it will become clearer what needs to be done for any of our internal plugin specs.

### Piano Roll Notes

To support non-western 12-TET scales, the piano roll needs to store information about the current scale it is working with. We will use an index map instead of simply storing the pitch of each note in the note itself. This is so the piano roll can easily transpose notes in any scale, and it allows the user to experiment with different scale types and tuning with the same note information. Later (not mvp), notes with pitches that lie outside the current scale will be supported using MPE.

The "scale" will be internally stored as:
* `MusicalScale`
    * The pitch of the root note, stored as an `f64`. For standard western 12-TET tuning, this will be 440.0Hz (C4).
    * A `Vec<f64>` of how each note (including the root note, but not the octave note) in the scale relates to the root note as a ratio of pitch (1.0 being the same pitch as root, 2.0 being an octave above the root.) The length of this Vec will also tell the piano roll how many notes there are in the scale. For example, a Vec for standard western 12-TET equal tempered tuning would look like this:
    ```
        1.0,       // 2 ^ (0/12)
        1.059463,  // 2 ^ (1/12)
        1.122462,  // 2 ^ (2/12)
        1.189207,  // 2 ^ (3/12)
        1.259921,  // 2 ^ (4/12)
        1.33484,   // 2 ^ (5/12)
        1.414214,  // 2 ^ (6/12)
        1.498307,  // 2 ^ (7/12)
        1.587401,  // 2 ^ (8/12)
        1.681793,  // 2 ^ (9/12)
        1.781797,  // 2 ^ (10/12)
        1.887749,  // 2 ^ (11/12)
    ```

See the "Time Keeping" section above for an explanation of how `MusicalTime`, `SampleTime`, and the `TempoMap` come into play.

For MVP, the DAW will internally store each individual note in this format:
* `MusicalNote`
    * The `MusicalTime` when this note is ON.
        * Whenever this value is changes, or whenever the `TempoMap` changes, the playback engine will convert this into the corresponding discrete `SampleTime`. The playback engine uses this new `SampleTime` and compares it to the `SampleTime` of the current playhead to know the exact sample that this event should be triggered on.
    * The `MusicalTime` when this note is OFF.
        * The same `SampleTime` strategy as described in the entry above.
    * The octave of the note, stored as an `i8`. For example, in 12-TET, a value of 0 means the octave with root note C4, 1 is the octave with root note C5, -1 is the octave with root note C3, etc.
    * The index of the note in the scale stored as a `u16`. For example, in 12-TET, a value of 0 means C, a value of 1 means C#, a value of 2 means D, etc.
    * The initial velocity of the note (stored as `f64` in the range [0.0, 1.0])
    * The initial pan of the note (stored as `f64` in the range [-1.0, 1.0], where 0 is center)

We will not be dealing with polyphonic aftertouch, micro pitch expression, or other per-note automation in this MVP.

### Automation Nodes

We will only use standard MIDI CC automation lanes in this MVP. While the eventual goal is to have something more flexible with a custom internal control/plugin spec, MIDI CC lanes are still required for compatibility with 3rd party plugin formats such as VST2/VST3/LV2/AU.

For MVP, each node in the automation lane will be stored in this format:
* `AutomationNode`
    * The `MusicalTime` when this node occurs.
        * Whenever this value is changes, or whenever the `TempoMap` changes, the playback engine will convert this into the corresponding discrete `SampleTime`. The playback engine uses this new `SampleTime` and compares it to the `SampleTime` of the current playhead to know the exact sample that this event should be triggered on.
    * The "curve" of this node. This will be a non-exhaustive enum. MVP options will include:
        * `Linear`: Linear automation between this node and the next
        * `Step`: Constant automation from this node until the next one. When the next node is reached, the value immediately jumps to the value of that new node.
    * The "value" of the node stored as an `f64`. This will only be in the normalized range [0.0, 1.0] to allow automation clips to easily be copied and moved between lanes.

## Timeline Engine

We will store this functionality in the [`rusty-daw-timeline`] repo.

The goal of this engine is to take information like audio clips and control clips, and turns them into a sort-of "virtual instrument" that takes the playhead time (in `SampleTime`) as input, and outputs buffers of audio, control, and MIDI data. As such, each "track" in the timeline will act like a single node in the "AudioGraph" (explained in the AudioGraph section above).

For mvp, this crate will include these data structures that can be added to an individual `TimelineTrack` struct:

* `AudioClip`
    * An immutable [`basedrop`] smart pointer to a `PCMResource`. The immutability reflects the non-desctructive nature of this engine. There will only be one sample resource per audio clip.
    * The time in `Seconds` where this audio clips starts in the raw samples
    * The `ShiftMode` to use (pitch & time stretching). This is explained in the "Sampler" section above.
    * The `InterpQuality` to use (interpolation quality). This is explained in the "Sampler" section above.
    * Any information on clip fades. The structure of this data is still yet to be determined.
    * A `bool` on whether or not the audio should be reversed
* `PianoRollClip`
    * A vec of `MusicalNote`s (described in the "Piano Roll Notes" section above)
    * The `MusicalScale` to use (described in the "Piano Roll Notes" section above)
* `AutomationClip`
    * A vec of `AutomationNode`s (described in the "Automation Nodes" section above)

All mutable methods that add or change data in these clips must also include the `TempoMap` of the project as an argument. This is so all events in `MusicalTime` are correctly converted into the corresponding `SampleTime` for use with the playback engine (as described in the "Time Keeping" section).

In addition, the `TimelineTrack` struct will have a method that updates all events to the new `SampleTime` when the `TempoMap` of the project changes.

The `TimelineTrack` struct will include a method for "seeking" the playhead to a particular `SampleTime`.

And finally, the `TimelineTrack` will have a "process" method that outputs buffers of audio, control, and/or MIDI data.

## Internal Plugins

We will store this functionality in the [`rusty-daw-plugins`] repo.

MVP will only include very basic effect plugins. These are mostly just to test the GUI design of internal plugins.
* Gain & Pan - pretty self explanatory
* Hard Clipper - hard clipper with (maybe) antialiasing

An EQ may be added since people on this team are working on one anyway.

## Plugin Hosting

We will store this functionality in the [`rusty-daw-plugin-host`] repo.

For MVP, we will only focus on hosting VST2 plugins. Displaying the plugin's GUI would be nice, but is not strictly mvp.

## Backend-Frontend Interface

All communication between the backend and frontend will be handled by lock-free spsc fifo ring buffers. Any crate of this type would work, but we will use the `ringbuf` crate for the sake of using the same xcrate across the project.

To make this work, most data structs will also create an "ID" counterpart when they are created. The frontend holds onto this "ID" counterpart, and sends the actual struct (wrapped in a [`basedrop`] smart pointer) to the backend via a ringbuffer. The internal structure of this "ID" struct will be an immutable [`basedrop`] smart pointer of the struct itself (although if lifetimes prove to be too difficult, we could also try using uniquely-generated u64 IDs with a hashmap).

The actual messages passed between these buffers will be determined by the specific project itself. For this DAW, we will likely have these messages:
* Timeline messages
    * `TimelinePlay` - play the timeline
    * `TimelineStop` - stop playback on the timeline
    * `TimelineSeek(MusicalTime)` - Seek the playhead of the timeline to this position
    * `UpdateTempoMap(TempoMap)` - Update the tempo map of the project
* PCM Resource messages
    * `AddPcmResource(PcmResource)` - Add a pcm resource to storage
    * `RemovePcmResource(PcmResourceID)` - Remove the pcm resource from storage
* Audio Clip messages
    * `AddAudioClip(AudioClip)` - Add a new audio clip to storage
    * `RemoveAudioClip(AudioClipID)` - Remove the audio clip from storage
    * `UpdateAudioClip(AudioClipID, ... parameters ...)` - Update the parameters of this audio clip
* Timeline track messages
    * `AddTrack(TimelineTrack)` - Add a track to the audio graph
    * `RemoveTrack(TimelineTrackID)` - Remove the track from the audio graph
    * `UpdateTrack(TimelineTrackID, ... parameters ...)` - Update the parameters of this track
    * `AddAudioClipToTrack(TimelineTrackID, AudioClipID, MusicalTime)` - Add the audio clip to this track at the given position
    * `RemoveAudioClipFromTrack(TimelineTrackID, AudioClipID)` - Remove the audio clip from this track
    * `UpdateAudioClipInTrack(TimelineTrackID, AudioClipID, MusicalTime)` - Update the position of this audio clip in the track
    * `AddPianoRollClipToTrack(TimelineTrackID, PianoRollClipID, MusicalTime)` - Add the piano roll clip to this track at the given position
    * `RemovePianoRollClipFromTrack(TimelineTrackID, PianoRollClipID)` - Remove the piano roll clip from this track
    * `UpdatePianoRollClipInTrack(TimelineTrackID, PianoRollClipID, MusicalTime)` - Update the position of this piano roll clip in the track
    * `AddAutomationClipToTrack(TimelineTrackID, AutomationClipID, MusicalTime)` - Add the automation clip to this track at the given position
    * `RemoveAutomationClipFromTrack(TimelineTrackID, AutomationClipID)` - Remove the automation clip from this track
    * `UpdateAutomationClipInTrack(TimelineTrackID, AutomationClipID, MusicalTime)` - Update the position of this automation clip in the track
    * `MuteTrack(TimelineTrackID, bool)` - Mute/unmute the particular track
    * `SoloTrack(TimelineTrackID, bool)` - Solo/unsolo the particular track
* Piano Roll Clip messages
    * `AddPianoRollClip(PianoRollClip)` - Add a new piano roll clip to storage
    * `RemovePianoRollClip(PianoRollClipID)` - Remove the piano roll clip from storage
    * `UpdatePianoRollClip(PianoRollClipID, ... parameters ...)` - Update the parameters of this piano roll clip
    * `AddMusicalNote(MusicalNote)` - Add a musical note. Note that musical notes can only be created with a `PianoRollClipID`, so the note is tied only to that specific piano roll clip.
    * `RemoveMusicalNote(MusicalNoteID)` - Remove the musical note
    * `UpdateMusicalNote(MusicalNoteID, ... parameters ...)` - Update the parameters of this musical note
* Automation Clip messages
    * `AddAutomationClip(AutomationClip)` - Add a new automation clip to storage
    * `RemoveAutomationClip(AutomationClipID)` - Remove the automation clip from storage
    * `UpdateAutomationClip(AutomationClipID, ... parameters ...)` - Update the parameters of this automation clip
    * `AddAutomationNode(AutomationNode)` - Add a new automation node. Note that automation nodes can only be created with a `AutomationClipID`, so the node is tied only to that specific automation clip.
    * `RemoveAutomationNode(AutomationNodeID)` - Remove the automation node
    * `UpdateAutomationNode(AutomationNodeID, ... parameters ...)` - Update the parameters of this automation node
* VST2 Plugin messages
    * `AddVst2Plugin(VstPlugin)` - Add a new VST2 plugin to the audio graph
    * `RemoveVst2Plugin(VstPluginID)` - Remove the VST2 plugin from the audio graph
    * `UpdateVst2Plugin(VstPluginID, ... parameters ...)` - Update parameters about this VST2 plugin
    * `OpenVst2PluginWindow(VstPluginID, ... parameters ...)` - Open the window for this VST2 plugin
    * `CloseVst2PluginWindow(VstPluginID, ... parameters ...)` - Close the window of this VST2 plugin
* Audio Graph messages
    * `AddGainNode(GainNode)` - Add a gain node to the audio graph
    * `RemoveGainNode(GainNodeID)` - Remove the gain node from the audio graph
    * `UpdateGainNode(GainNodeID, ... parameters ...)` - Update parameters about this gain node
    * `AddPanNode(PanNode)` - Add a pan node to the audio graph
    * `RemovePanNode(PanNodeID)` - Remove the pan node from the audio graph
    * `UpdatePanNode(PanNodeID, ... parameters ...)` - Update parameters about this pan node
    * `AddDelayNode(DelayNode)` - Add a "delay compensation" node to the audio graph
    * `RemoveDelayNode(DelayNodeID)` - Remove the delay node from the audio graph
    * `UpdateDelayNode(DelayNodeID, ... parameters ...)` - Update parameters about this delay node
    * `AddSumNode(SumNode)` - Add a "summing" node to the audio graph
    * `RemoveSumNode(SumNodeID)` - Remove the sum node from the audio graph
    * `UpdateSumNode(SumNodeID, ... parameters ...)` - Update parameters about this sum node
    * `UpdateAudioGraph(... parameters ...)` - This is where the order and connection of nodes in the audio graph gets updated. This tells the audio graph in which order (and which threads) it should process each node.

Note that there are no messages about changing hardware devices. This is because the entire backend will be reconstructed in that case.


# GUI Design (MVP)

*TODO*

[`Symphonia`]: https://github.com/pdeljanov/Symphonia
[`cpal`]: https://github.com/RustyDAW/cpal
[`rusty-daw-time`]: https://github.com/RustyDAW/rusty-daw-time
[`rusty-daw-io`]: https://github.com/RustyDAW/rusty-daw-io
[`rusty-daw-audio-clip`]: https://github.com/RustyDAW/rusty-daw-audio-clip
[`basedrop`]: https://github.com/glowcoil/basedrop
[`deip`]: https://github.com/BillyDM/Awesome-Audio-DSP/blob/main/deip.pdf