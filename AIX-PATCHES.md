# Local JUCE patches (AIX Labs)

Local modifications applied on top of upstream JUCE. Re-apply after a JUCE upgrade.

## modules/juce_audio_plugin_client/juce_audio_plugin_client_AU_1.mm

**`getSupportedLayoutTagsForBus` — filter by current bus channel count.**

Stock JUCE returns every layout the bus could be configured into across any multi-bus
reconfiguration. Apple's `auvaltool` evaluates the layout list per-format and reports
`Mismatch between reported channel layouts and reported numChannels` when, e.g., a Mono
tag appears in a stereo bus's list. The patch restricts the returned tags (both the named
CoreAudio tags and the `DiscreteInOrder` tag) to those whose channel count matches
`bus->getCurrentLayout().size()`. Mono/stereo support is preserved at the AUChannelInfo
and StreamFormat level; only the per-format layout query is filtered.

Marker: search for `[AIX patch]` in the file.

## modules/juce_audio_processors_headless/processors/juce_AudioProcessor.cpp

**`validateParameter` — suppress `getVersionHint() == 0` assertion under AU.**

Three lines commented out around the `std::call_once`/`jassertfalse` for parameters
without a version hint. Avoids the assertion firing for legacy plugins whose parameters
were authored before version hints were introduced.
