<!-- Add links to top -->

Music Metadata <!-- omit in toc -->
===============================================

(C) Tim Curtis 2024  

The source for this section is from the [MPD Documentation.](https://mpd.readthedocs.io/en/latest/protocol.html#tags)

### Track tags supported by MPD

Tags marked in __Bold__ are the subset used by moOde.

|Tag|moOde|Description|
| :- | :- | :- |
|__artist__|yes|the artist name. Its meaning is not well-defined; see “composer” and “performer” for more specific tags|
|artistsort|no|same as artist, but for sorting. This usually omits prefixes such as “The”|
|__album__|yes|the album name|
|albumsort||same as album, but for sorting|
|__albumartist__|yes|on multi-artist albums, this is the artist name which shall be used for the whole album. The exact meaning of this tag is not well-defined|
|albumartistsort|no|same as albumartist, but for sorting|
|__title__|yes|the song title|
|titlesort|no|same as title, but for sorting|
|__track__|yes|the decimal track number within the album|
|__name__|yes|a name for this song. This is not the song title. The exact meaning of this tag is not well-defined. It is often used by badly configured internet radio stations with broken tags to squeeze both the artist name and the song title in one tag|
|__genre__|yes|the music genre|
|mood|no|the mood of the audio with a few keywords|
|__date__|yes|the song’s release date. This is usually a 4-digit year|
|__originaldate__|yes|the song’s original release date|
|__composer__|yes|the artist who composed the song|
|composersort|no|same as composer, but for sorting|
|__performer__|yes|the artist who performed the song|
|__conductor__|yes|the conductor who conducted the song|
|work|no|a work is a distinct intellectual or artistic creation, which can be expressed in the form of one or more audio recordings|
|ensemble|no|the ensemble performing this song, e.g. “Wiener Philharmoniker”|  
|movement|no|name of the movement, e.g. “Andante con moto”|
|movementnumber|no|movement number, e.g. “2” or “II”|
|showmovement|no|If this tag is set to “1” players supporting this tag will display the work, movement, and movementnumber instead of the track title|
|location|no|location of the recording, e.g. “Royal Albert Hall”|
|grouping|no|“used if the sound belongs to a larger category of sounds/music” (from the IDv2.4.0 TIT1 description)|
|__comment__|yes|a human-readable comment about this song. The exact meaning of this tag is not well-defined|
|__disc__|yes|the decimal disc number in a multi-disc album|
|label|no|the name of the label or publisher|
|musicbrainz_artistid|no|the artist id in the MusicBrainz database|
|__musicbrainz_albumid__|yes|the album id in the MusicBrainz database|
|musicbrainz_albumartistid|no|the album artist id in the MusicBrainz database|
|musicbrainz_trackid|no|the track id in the MusicBrainz database|
|musicbrainz_releasegroupid|no|the release group id in the MusicBrainz database|
|musicbrainz_releasetrackid|no|the release track id in the MusicBrainz database|
|musicbrainz_workid|no|the work id in the MusicBrainz database. There can be multiple values for some of these tags. For example, MPD may return multiple lines with a performer tag. A tag value is a UTF-8 string|

### Other Metadata

The response to MPD lsinfo and similar commands may include other metadata as listed below in addition to track tags.

|Tag|moOde|Description|
| :- | :- | :- |
|__duration__|yes|the duration of the song in seconds; may contain a fractional part|
|Time|no|like duration, but as integer value. This is deprecated and is only here for compatibility with older clients. Do not use|
|Range|no|if this is a queue item referring only to a portion of the song file, then this attribute contains the time range in the form START-END or START- (open ended); both START and END are time stamps within the song in seconds (may contain a fractional part). Example: 60-120 plays only the second minute; “180 skips the first three minutes|
|__Format__|yes|the audio format of the song (or an approximation to a format supported by MPD and the decoder plugin being used). When playing this file, the audio value in the status response should be the same|
|__Last-Modified__|yes|the time stamp of the last modification of the underlying file in ISO 8601 format. Example: “2008-09-28T20:04:57Z”
|added|no|the time stamp when the file was added in ISO 8601. A negative value means that this is unknown/unavailable. Example: “2023-11-25T13:25:07Z”|
