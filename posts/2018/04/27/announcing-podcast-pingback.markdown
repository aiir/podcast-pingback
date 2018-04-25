# Announcing Podcast Pingback

The podcast industry is continuing to grow at an impressive rate. But one thing
that is frequently cited as holding it back is the difficulty with which
accurate metrics can be obtained.

Several apps have implemented their own measurement systems, but each exist
within their respective walled garden. None of these systems are interoperable
and therefore lack offering a comparable or complete picture of podcast
listening.

So, based on a previous
[exploratory idea](https://medium.com/togglebit/how-we-all-can-help-podcasts-thrive-but-nothings-quite-that-easy-e583bb5ffe79),
the team at [Aiir](https://www.aiir.com/) have developed
"[Podcast Pingback](/specification/1)", an intentionally simplistic method
for reporting listening activity back to a podcast publisher.

Our work at Aiir is focussed around building innovative, easy to use products,
to enable broadcasters to manage content whilst growing audience and revenue,
all in one place. This means we know first-hand how making our clients audio
more accountable is crucial for the future of podcasting. And if this is true
for broadcasters exploring podcasting as a new revenue stream, it follows that
it's even more important for those podcast innovators where it's one of their
only revenue streams.

The grand ambition is simple; to get an open, interoperable analytics method
embedded in as many clients and used by as many publishers as possible. It's a
big one, and there's a [huge elephant in the room](https://www.apple.com/), but
we have some core beliefs that drive this ambition:

* We believe that listeners will understand the benefit in their favourite
  podcast creators knowing more about their audience, to improve their content
  and attract investment, and will support the inclusion of anonymous and
  controllable analytic measurement in their apps.
* We believe that podcast app developers appreciate the value in tracking
  interactions in an open, interoperable standard, to learn more about their
  users and improve their apps and to ensure the healthy future of the content
  their software relies on.

You might be thinking "great time to try and launch a snooping protocol to a
suspicious public"? We take privacy concerns very seriously, but there's no
getting around the fact that one of the desires for these analytics is to
support the "evil" act of money making to support creators.

Spot and read advertising are unlikely to go away anytime soon and the arguably
more controversial personalised or heavily targeted advertising is out of scope.
The method revolves around an anonymous unique identifier derived by the client
and the inclusion of a listener profile is optional and intended to be under a
user consent agreement. The only way it can be obtained is by a client supplying
it, which would raise a broader question of how the client obtained it before
even passing it to the publisher. Finally, a "forget me" user flow was included
to ensure a listener is always in control.

[Check out the specification now](/specification/1). It relies on being adopted
by publishers and software developers alike. If both points ring true and we see
the adoption of a standard we can move the whole industry forward with a more
complete view of the consumption of our content.

We're keen to receive feedback and engage the community in the future
development of this spec. Please feel free to look at and fork our GitHub
repository for the site and submit pull requests, or
[get in touch](mailto:andy@aiir.com?subject=Podcast%20Pingback) to discuss the
specification further.
