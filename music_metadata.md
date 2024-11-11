<!-- Add links to top -->

Music Metadata <!-- omit in toc -->
===============================================

(C) Tim Curtis 2024  

The source for this section is from the [MPD Documentation](https://mpd.readthedocs.io/en/latest/protocol.html#tags) in the development branch.

### Tags supported by MPD:

Tags colored in Orange are the subset used by moOde.

__<span style="color:#d35400">artist:</span>__ the artist name. Its meaning is not well-defined; see “composer” and “performer” for more specific tags.  
__artistsort:__ same as artist, but for sorting. This usually omits prefixes such as “The”.  
__<span style="color:#d35400">album:</span>__ the album name.  
__albumsort:__ same as album, but for sorting.  
__<span style="color:#d35400">albumartist:</span>__ on multi-artist albums, this is the artist name which shall be used for the whole album. The exact meaning of this tag is not well-defined.   
__albumartistsort:__ same as albumartist, but for sorting.  
__<span style="color:#d35400">title:</span>__ the song title.  
__titlesort:__ same as title, but for sorting.  
__<span style="color:#d35400">track:</span>__ the decimal track number within the album.  
__<span style="color:#d35400">name:</span>__ a name for this song. This is not the song title. The exact meaning of this tag is not well-defined. It is often used by badly configured internet radio stations with broken tags to squeeze both the artist name and the song title in one tag.  
__<span style="color:#d35400">genre:</span>__ the music genre.  
__mood:__ the mood of the audio with a few keywords.  
__<span style="color:#d35400">date:</span>__ the song’s release date. This is usually a 4-digit year.  
__<span style="color:#d35400">originaldate:</span>__ the song’s original release date.  
__<span style="color:#d35400">composer:</span>__ the artist who composed the song.  
__composersort:__ same as composer, but for sorting.  
__<span style="color:#d35400">performer:</span>__ the artist who performed the song.  
__<span style="color:#d35400">conductor:</span>__ the conductor who conducted the song.  
__work:__ “a work is a distinct intellectual or artistic creation, which can be expressed in the form of one or more audio recordings”.  
__ensemble:__ the ensemble performing this song, e.g. “Wiener Philharmoniker”.    
__movement:__ name of the movement, e.g. “Andante con moto”.      
__movementnumber:__ movement number, e.g. “2” or “II”.    
__showmovement:__ If this tag is set to “1” players supporting this tag will display the work, movement, and movementnumber` instead of the track title.  
__location:__ location of the recording, e.g. “Royal Albert Hall”.  
__grouping:__ “used if the sound belongs to a larger category of sounds/music” (from the IDv2.4.0 TIT1 description).  
__<span style="color:#d35400">comment:</span>__ a human-readable comment about this song. The exact meaning of this tag is not well-defined.  
__<span style="color:#d35400">disc:</span>__ the decimal disc number in a multi-disc album.  
__label:__ the name of the label or publisher.  
__musicbrainz_artistid:__ the artist id in the MusicBrainz database.  
__<span style="color:#d35400">musicbrainz_albumid:</span>__ the album id in the MusicBrainz database.  
__musicbrainz_albumartistid:__ the album artist id in the MusicBrainz database.  
__musicbrainz_trackid:__ the track id in the MusicBrainz database.  
__musicbrainz_releasegroupid:__ the release group id in the MusicBrainz database.  
__musicbrainz_releasetrackid:__ the release track id in the MusicBrainz database.  
__musicbrainz_workid:__ the work id in the MusicBrainz database.
There can be multiple values for some of these tags. For example, MPD may return multiple lines with a performer tag. A tag value is a UTF-8 string.  

### Other Metadata

The response to lsinfo and similar commands may contain song tags and other metadata, specifically:

__<span style="color:#d35400">duration:</span>__ the duration of the song in seconds; may contain a fractional part.  
__Time:__ like duration, but as integer value. This is deprecated and is only here for compatibility with older clients. Do not use.  
__Range:__ if this is a queue item referring only to a portion of the song file, then this attribute contains the time range in the form START-END or START- (open ended); both START and END are time stamps within the song in seconds (may contain a fractional part). Example: 60-120 plays only the second minute; “180 skips the first three minutes.  
__<span style="color:#d35400">Format:</span>__ the audio format of the song (or an approximation to a format supported by MPD and the decoder plugin being used). When playing this file, the audio value in the status response should be the same.  
__<span style="color:#d35400">Last-Modified:</span>__ the time stamp of the last modification of the underlying file in ISO 8601 format. Example: “2008-09-28T20:04:57Z”
added 13: the time stamp when the file was added in ISO 8601. A negative value means that this is unknown/unavailable. Example: “2023-11-25T13:25:07Z”
