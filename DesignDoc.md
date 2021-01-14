# RustyDAW: High-level Design

RustyDAW is a collection of tools for creating Digital Audio Workstations (DAWs).

RustyDAW is a community-driven project. If you are interested in participating, feel free to join the [RustAudio Discord](https://discord.com/invite/8rPCp9Q), channels `#rusty-daw` and `#rusty-daw-gui`.

## Objective

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

## Scope

**SECTION TODO.**

As stated above, we want a "modular ecosystem" of DAW tools, but let's strive for a "reference implementation" of a full DAW as a proof-of-concept.

What should the ideal end-goal be? What would we be happy with releasing as a "1.0" version? A full Ableton Live / Bitwig-like DAW? List the scope of that end-goal project here: goals and non-goals. This may include general ideas of what we'd like to split into different modules and crates, such as "the audio system" (which may be split up further) and "the GUI system" (which may also be split up further).

You don't have to list every fine-grained feature, device, and effect here, but the general project goal/idea should be there.

After that, we can identify what a minimum-viable DAW might look like that is a step in that direction, and start a new design doc for it.

Some stuff to think about:

Goals

 - Identify the core algorithms that can be shared across separate DAW projects.
 - Identify key issues and learn from the mistakes of past open-source projects.
 - Create the individual modules needed to create the minimum-viable product.
 - Work together to create an example minimum-viable DAW.
 - Implement more features from there as we like.
 - GPLv3 everything, unless agreed upon otherwise beforehand.

Non-Goals

 - This project will not deal with specific UI design details, only broad design concepts and reusable widgets.
 - Every file and plugin standard does not need to be implemented for a minimum-viable product. Identify which standards are absolutely necessary first.
 - Specific plugins like synthesizers, filters, delay, reverb, etc, should not be included in the minimum-viable product. Only focus on the core backend-algorithms for now.

Basically, a list of requirements and "anti-requirements". You can also create a list of use-cases ("user journeys") here, if desired.

## Design

**SECTION TODO.**

Once we know what we want to build towards, let's create a high-level overview of our proposed "1.0" design.

This can include subsections on architecture decisions, GUI mockups, how we handle time, what information different event types hold, our design for a DSP graph, how we handle inputs and outputs, how we will communicate between the audio/GUI/other threads, etc. etc. etc.

This doesn't need to go into crazy fine-grained detail, that's what other more-specific design docs are for.

Sequence diagrams, system diagrams, and data flow diagrams can be useful to illustrate your ideas.

## Alternatives considered

**SECTION TODO.**

During discussion, we'll probably end up generating a bunch of ideas that we ultimately shoot down for whatever reason. List those ideas and those reasons here.
