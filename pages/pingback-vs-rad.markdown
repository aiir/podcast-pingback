# Podcast Pingback vs NRP RAD

## Feed vs ID3

Podcast Pingback advertises one or more data receiving endpoints in the RSS feed
for a podcast. RAD advertises an endpoint in the ID3 tag of the podcast audio.

* The ability to include a specific ID3 tag in the audio content is potentially
  an added step and complication in the audio production process and may fall on
  staff who otherwise have little interest or technical involvement in this
  level of podcasting specifics. The approach of Pingback splits the
  dependencies between production and distribution, the implementation can be
  entirely downstream of the audio content production.

* Decisions have to be made about which bits of the audio content are of
  interest ahead of time, to mark up. Conversely Pingback collects abstract data
  until such a point that a point of interest is identified and the data can be
  mined. If an area of interest is discovered later with RAD, a new audio file
  must be issued and the process would be reliant on as many podcast clients as
  possible picking up the new audio feed to get reliable analytics.

* RAD only works with clients that support reading ID3 tags. This is
  particularly a concern with web browsers (e.g. player embeds in a website)
  which do not have this functionality natively. This may be resolved in a
  future update by defining a secondary way of identifying areas of interest in
  the audio in a way the browser can parse, but this then creates more than one
  source of truth for the standard.

## Server Implementation

Pingback requires an endpoint that accepts all events and then, potentially at
a point in the future, those events need to be analysed to make sense of the
data. Conversely, RAD events are already pre-defined, so the process simply
totals the incoming events.

* For similar reasons mentioned in the Feed vs ID3 section, this means the focus
  of the analytics have to be decided ahead of time and baked in to the audio
  file, but this does make parsing the received events simpler.

* Pingback allows any part of the audio to be tracked, and decisions to be made
  at any point in time about what events are of interest, but it does require
  additional computation against the received events.

## Client Implementation

Pingback defines a basic list of events to be reported, there's no content to be
parsed ahead of time by the client to know what events to report. Conversely,
RAD requires the client to parse a list of events in the ID3 tag and then
monitor for those events to occur.

* The client implementation is more complicated for RAD than for Pingback.
* In balance, the server implementation for Pingback is more complicated.
* Comparing the number of unique clients hoped to adopt one of these standards
  compared to broadcaster/service provider server implementations, we feel
  placing the complexity on the server side is in the best interests of the
  wider community, reducing potential errors in implementation and increasing
  adoption.
