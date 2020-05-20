# Globe Design

[Globe](https://github.com/microsoft/globe) is a library me and my colleagues pulled out
of an internal codebase in order to provide it to the open source audience, but also to
eventually improve it as it is based on code which has not been maintained and perhaps
designed to a standard we want to eventually reach.

The purpose of the library is to provide an Intl-like mechanism, but one which takes into
an account the operating system date and time formatting settings.

Intl by itself takes perfect care of date and time localization, and we would use it bare
if that was all we needed, but for a scenario where our code is running in Electron and
not the browser. It is still JavaScript, but it is running in a runtime which also allows
exposing native code functions making the runtime more flexible than the browser. One of
the native APIs we make use of is one which provides our code with date and time format
settings as configured in the operating system. This information is not available in
browser JavaScript APIs, so localization efforts in the browser end with Intl and indeed
that's where we end too, when our code is running in the browser. When running on the
desktop, though, there's further we can go to make the application feel more native. One
of the improvements we can make is to query these operating system date and time format
settings and construct our Intl options such that the resultant formatted string follows
the same pattern the operating system uses when formatting date and time. Hopefully,
makes the user experience a tiny bit better and bridges the gap between our web-based
application and the native desktop experience a bit more.

How this is done is as follows:

1. Query operating system date and time format strings
2. Construct an Intl options object to mimic them
3. Format date and time values this way to honor operating system date and time format

Unfortunately, things are rarely this easy and we are dealing with a few complications:

1. Operating systems make the date and time formatting strings configurable

   The user is free to reorder the various date and time tokens in any way they please
   and as a result the space of possible date and time format patterns is infinite.
   Not only can users mix and match the date and time format tokens, they can also
   interleave the pattern with verbatim strings to be included in the final pattern.
  
2. Operating systems provide a limited set of formatting strings not suitable for all
   cases

   As a part of the application user interface design, relative date and time labels
   are a great way to present date and time information in a way that's quick to use
   and contextual. The operating systems we support provide some variants of date and
   time formats which differ in length and the amount of information contained, but
   there are cases where none of these is a good match. As an example, all date format
   strings usually include the year token, but it often makes sense to exclude the
   year value from the date label if it matches the current year.
   
Due to #1 above, it is impossible to fully parse and represent the format strings in
a more structured way. Due to #2, the aforementioned is pretty much necessary as the
strings need to be processed to enable format options which enable formatting in ways
which are not captured in the system-provided formattign strings, but are still in
their "spirit" for a lack of a better term.
 
Fully parsing these formatting strings is impossible due to the various verbatim
tokens users may choose to include in the formatting strings, so we've quickly ruled
that option our. However, not all is lost, as the fraction of people who tweak these
settings to very non-standard values is incredibly small and that largely limits the
pool of possible date and time formatting strings the operating system may provide
even including the user adjustments. In theory, the common set of date and time format
strings across all cultures and with the most common tweaks is still pretty small.
(Dozens, at most low hundreds of patterns? Depending on what we want to support.)

For formatting strings which fall out of this small common set, we can either opt
out of formatting the date and time values so that the operating system formatting
settings are honored altogether, or we can opt out of processing them further to do
the desired transformations (such are dropping the year as mentioned above) and
instead use them verbatim. This way users with common patterns can benefit from the
further processing which enables the UI niceties such as relative labels, but users
with uncommon patterns are not harmed in any way for having chosen unusual patterns.
At worst, they either get default browser behavior even on desktop or they get semi-
respective behavior (in terms of the patterns but not the transformations on them).

The users who do have common formatting strings benefit from Globe eventually
recognizing these formatting strings and providing structured information which
allows the developers to do the transformations such as removing the year where
desired.

As of current, Globe does this in what we consider a slightly hacky way and we plan
on refactoring the library in time to not only simplify it, but also improve it so
that more common formatting patterns are correctly implemented and tested and the
less common ones are not incorrectly interpreted resulting in broken labels.

## Status

The design of the refactor is being worked out and I will update this document with
our progress as we approach the final design document.
