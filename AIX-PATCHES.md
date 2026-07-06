# Local JUCE patches (AIX Labs)

Local modifications applied on top of upstream JUCE. Re-apply after a JUCE upgrade.

## modules/juce_audio_plugin_client/juce_audio_plugin_client_AU_1.mm

**`getSupportedLayoutTagsForBus` — restrict only the `DiscreteInOrder` tag to the current
bus channel count.**

Apple's `auvaltool` reports `Mismatch between reported channel layouts and reported
numChannels` when `DiscreteInOrder` tags for other channel counts appear in the bus's
layout list; stock JUCE emits them for every count up to the bus maximum. The patch emits
the `DiscreteInOrder` tag only for `bus->getCurrentLayout().size()`.

The named CoreAudio tags (Mono, Stereo, …) are deliberately left unfiltered, exactly as
stock JUCE publishes them. An earlier version of this patch filtered those too, which
broke Mono/Dual Mono insertion in Logic (reported against Drum EQ 2.0.15): Logic consults
the published layout list when inserting the mono variants, and the Mono tag was missing
because the bus's default format is stereo. AUChannelInfo alone is not sufficient for
Logic — the Mono named tag must be present.

Marker: search for `[AIX patch]` in the file.

## modules/juce_audio_processors_headless/processors/juce_AudioProcessor.cpp

**`validateParameter` — suppress `getVersionHint() == 0` assertion under AU.**

Three lines commented out around the `std::call_once`/`jassertfalse` for parameters
without a version hint. Avoids the assertion firing for legacy plugins whose parameters
were authored before version hints were introduced.
