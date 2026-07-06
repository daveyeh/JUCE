# Local JUCE patches (AIX Labs)

Local modifications applied on top of upstream JUCE. Re-apply after a JUCE upgrade.

## modules/juce_audio_plugin_client/juce_audio_plugin_client_AU_1.mm

**`getSupportedLayoutTagsForBus` — align the published layout-tag list with the AU's
reported channel capabilities (`channelInfo`).**

Named CoreAudio tags (Mono, Stereo, …) are published for a layout the bus supports
*only if* its channel count appears in the AU's reported channel capabilities
(AUChannelInfo) for that direction; negative capability entries are wildcards and lift
the restriction. The `DiscreteInOrder` tag is emitted only for
`bus->getCurrentLayout().size()`.

Why not the two simpler behaviours:

- *Stock JUCE* publishes every layout reachable via any multi-bus reconfiguration. On
  plugins where a mono main bus is only valid after also reconfiguring a sidechain
  (Intuition, Multi-Band Gate), that publishes Mono while AUChannelInfo says `[2,2]`
  only, and `auvaltool` fails with `Mismatch between reported channel layouts and
  reported numChannels`.
- *Filtering to the bus's current channel count* (the first version of this patch)
  passes auval but broke Mono/Dual Mono insertion in Logic for every simple mono/stereo
  plugin (reported against Drum EQ 2.0.15): the default format is stereo, so the Mono
  tag was never published, and Logic consults this list when inserting mono variants —
  AUChannelInfo `[1,1]` alone is not sufficient.

Keying the list to AUChannelInfo gives both: plugins that advertise `[1,1]` publish Mono
(Logic mono works), plugins that only advertise `[2,2]` don't (auval stays consistent).

Marker: search for `[AIX patch]` in the file.

## modules/juce_audio_processors_headless/processors/juce_AudioProcessor.cpp

**`validateParameter` — suppress `getVersionHint() == 0` assertion under AU.**

Three lines commented out around the `std::call_once`/`jassertfalse` for parameters
without a version hint. Avoids the assertion firing for legacy plugins whose parameters
were authored before version hints were introduced.
