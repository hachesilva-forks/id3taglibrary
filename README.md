Perry’s ID3 Tag Library
=======================

A free open-source ID3v1 and ID3v2 tag parsing utility for MP3 files. Programmed in VB.NET, it has been tested to work with Visual Studio, VB.NET, ASP.NET, Visual Basic for Applications (VBA) and VBScript.

* Supports ID3v1, ID3v2.1, ID3v2.2, ID3v2.3, ID3v2.4.  
* Object oriented architecture - MP3File, ID3v2Tag, Frame, mp3.Artwork, etc.  
* Meticulously built from scratch in accordance with official ID3 spec.  
* Detects ALL ID3v2 frames such as those typically hidden or ignored - PRIV, etc.  

*This library parses ID3 tags on the binary level, true to the ID3 spec, therefore it can read an entire ID3 tag including all frames, not just a handful of common frames.*

![Class Diagrams](http://files.glassocean.net/github/id3taglibrary1.jpg)

Perry's ID3 Tag Viewer implements the current version of the library:

![Perry's ID3 Tag Viewer](http://glassocean.net/media/id3-tag-viewer-1.jpg)

The Tag Viewer is currently not available as an open-source project...

Roadmap
-------

**Tag mapping**

Partial tag mapping is already implemented, but the guys over at HydrogenAudio.org have setup a wiki attempting to unify tag mapping across several tag formats. This is the closest to a tag mapping standard out there so the lib should support it by implementing it. Ideally, one should be able to grab the "Artist" of any file by just calling the wrapper property e.g. mp3.Artist instead of digging through multiple artist frames for a value.

**Tag writing**

First prototype of direct ID3v2 frame data editing/saving has been partially implemented and tested, which involves  splitting and rejoining the file if the saved frame data is larger than the current frame size + padding. This should also include reading/writing custom frame types.

The Tag Viewer should also implement a simple interface for making these types of edits, preferably straight from the DataGrids.

**Batch processing**

A feature for the Tag Viewer; this would allow batch processing of mp3 files, such as changing the artwork for an entire album, removing hidden PRIV frames from several albums, or renaming a selection of files based on their filenames/tag data.

**MPEG stream parsing**

It turns out that the MPEG audio specs are not just freely available on the internet - they're copyrighted. However, in 1998, Predrag Supurovic released the unofficial MPEG HEADER FRAME specs after some extensive internet and source code research. A prototype has been developed in the lib utilizing this well known algorithm [[1]](#references).  

Will search the mp3 file for valid MPEG header frames starting AFTER the ID3v2 tag. The reason is that many false sync signals can be detected in the ID3v2 tag which causes excessive processing since each sync signal must be evaluated to make sure it's a true MPEG header frame.

Results are looking good so far! Here's an output of the valid MPEG header frames being detected in the first 10 KB (after the ID3v2 tag) of an mp3 file:

```
Reading next chunk of 10000 bytes...
Detected MPEG header frame sync byte at array index 458 with a binary value of 11111111111110111110001001100100 or an ASCII value of ÿûâd or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 1503 with a binary value of 11111111111110111110000001100100 or an ASCII value of ÿûàd or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 1947 with a binary value of 11111111111111000100100111000000 or an ASCII value of ÿüIÀ or decoded values of Bitrate: 56000 Sampling Rate: 32000
Detected MPEG header frame sync byte at array index 2547 with a binary value of 11111111111110111110001001100100 or an ASCII value of ÿûâd or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 2835 with a binary value of 11111111111111111001010111110001 or an ASCII value of ÿÿ•ñ or decoded values of Bitrate: 128000 Sampling Rate: 48000
Detected MPEG header frame sync byte at array index 2843 with a binary value of 11111111111111111100101100011100 or an ASCII value of ÿÿË or decoded values of Bitrate: 224000 Sampling Rate: 32000
Detected MPEG header frame sync byte at array index 3592 with a binary value of 11111111111110111110001001000100 or an ASCII value of ÿûâD or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 4637 with a binary value of 11111111111110111110001001000100 or an ASCII value of ÿûâD or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 5123 with a binary value of 11111111111111111011010110011011 or an ASCII value of ÿÿµ› or decoded values of Bitrate: 192000 Sampling Rate: 48000
Detected MPEG header frame sync byte at array index 5682 with a binary value of 11111111111110111110001001000100 or an ASCII value of ÿûâD or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 6727 with a binary value of 11111111111110111110001001000100 or an ASCII value of ÿûâD or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 7682 with a binary value of 11111111111111111110000000111111 or an ASCII value of ÿÿà? or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 7772 with a binary value of 11111111111110111110001001100100 or an ASCII value of ÿûâd or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 8085 with a binary value of 11111111111111011110101110010111 or an ASCII value of ÿýë— or decoded values of Bitrate: 320000 Sampling Rate: 32000
Detected MPEG header frame sync byte at array index 8731 with a binary value of 11111111111111111110101100011111 or an ASCII value of ÿÿë or decoded values of Bitrate: 320000 Sampling Rate: 32000
Detected MPEG header frame sync byte at array index 8817 with a binary value of 11111111111110111110001001000100 or an ASCII value of ÿûâD or decoded values of Bitrate: 320000 Sampling Rate: 44100
Detected MPEG header frame sync byte at array index 9862 with a binary value of 11111111111110111110001001000100 or an ASCII value of ÿûâD or decoded values of Bitrate: 320000 Sampling Rate: 44100
```

The results above are equivalent to the same mp3 file when viewed in Windows Media ASF View 9 Series, a tool for verifying the integrity of media files to ensure they conform with Microsoft's ASF specs.

Looking at the data above we notice several key facts:

* The binary values are parsed to give us the bitrate, sampling rate, etc. on a per-frame basis. VBRI, XING and LAME header parsing should be implemented to support VBR streams without needing to process the entire file.
* Not every MPEG frame will contain the same data (bitrate, sampling rate, etc) because encoding happens on a per-frame basis. This is always the case with VBR encoding, but if we detect bitrate/sampling rate changes in a file that uses CBR encoding, we should ignore those frames because they are technically invalid.
* We cannot determine the song length based on a single MPEG frame; instead we must count all frames (process the entire file) or do an approximation by averaging the first several frames.
* If you count the gap between each index in the data, you will notice it's usually 1044 or 1045. This value happens to be the MPEG frame size for each frame. However, if frames are detected that don't have this computed frame size, we should discard them because they are likely false sync signals.

**Auto-tag & auto-fix**

Automatically fill tag data (artist, album, track, artwork, etc) from online sources (freedb, Amazon, etc).

**Windows Media Player album artwork fix**

Trying to maintain a song library in Windows Media Player can be frustrating, and here's why...

The official ID3 spec treats artwork on a *per-song* basis. Every mp3 file in a folder or album can have it’s own unique artwork embedded in the mp3 file itself so that it’s portable without needing other jpg files, and software such as WinAmp will display the artwork for each mp3 file properly. However, Windows Media Player treats artwork differently, by storing the artwork as hidden jpg files in the same folder as the mp3 files.

Windows Media Player 7 names the hidden jpg files **AlbumArtSmall.jpg**, **AlbumArtLarge.jpg** and **Folder.jpg**. You can only have one set of these in a folder, hence one set of artwork per folder. What if you organized your music in such a way where you had a folder containing hit singles from several different artists? Windows Media Player ends up using the artwork from a single mp3 file for the entire set of mp3 files in that folder.

Windows Media Player 8/9/10/11/12 names the hidden jpg files **AlbumArt_{XXX}_Small.jpg**, **AlbumArt_{XXX}_Large.jpg** and **Folder.jpg**, where XXX is a GUID (long string of text). But what determines these GUIDs, and what is their purpose? Windows Media Player creates the GUIDs to give every single song or album a unique identity in their online database. When playing mp3 files, Windows Media Player can retrieve song information automatically from the online database, which is great if the mp3 files themselves don’t have complete song information stored in the embedded ID3 tags. However, Windows Media Player will also automatically update your mp3 files with this song information retrieved from the online database (an option that is enabled by default) and this data can be inaccurate.

Since the online content is submitted by users like yourself, a small portion of the content in this database contains bad GUIDs for certain songs or albums. The trouble happens when Windows Media Player retrieves a bad GUID from the database, particularly a GUID of {00000000-0000-0000-0000-000000000000}. When this happens, Windows Media Player will generate the album artwork (hidden jpg files) with names consisting of **AlbumArt_{00000000-0000-0000-0000-000000000000}_Small.jpg** and **AlbumArt_{00000000-0000-0000-0000-000000000000}_Large.jpg**, no longer a unique identity.

As was already explained in the above note about Windows Media Player 7, what ends up happening is that you can only have one set of artwork in a folder, thus Windows Media Player ends up using the artwork from a single song or album for the entire set of mp3 files in that folder, and it will overwrite the embedded artwork in those mp3 files, severely damaging your music collection.

To make matters worse, Windows Media Player seems to store the GUID inside of a PRIV frame in the ID3 tag embedded in the mp3 file, so even if you try fixing the embedded artwork, Windows Media Player will see the GUID in the PRIV frame and regenerate the hidden jpg files. The problem spreads around from person to person, likely finds its way back into the online database, and never truly gets fixed because users don’t have a way of correcting mp3 files with bad GUIDs stored in PRIV frames of the ID3 tags:

![GUID in PRIV frames](http://files.glassocean.net/github/id3taglibrary3.jpg)

Users of Microsoft Windows XP/Vista/7 have no doubt seen these before:

![artwork as hidden jpg files](http://files.glassocean.net/github/id3taglibrary2.jpg)

You're bound for trouble when the GUID is all zeros!

Perry's ID3 Tag Library (and Tag Viewer) can detect this problem, and future versions with tag writing capabilities should allow users to fix it. This won't help fix the online database, but at least users will have a way to fix their own music collections by removing the PRIV frames so Windows Media Player stops generating the artwork (jpg files) with bad GUIDs [[2]](#references).

References
----------

[1] [MPEG AUDIO HEADER FRAME](http://www.mpgedit.org/mpgedit/mpeg_format/mpeghdr.htm)  
[2] [Help, I am going INSANE! WMP 12 album art.](http://social.technet.microsoft.com/Forums/windows/en-US/e6ee46cc-f088-4847-a9a2-58fac6888407/help-i-am-going-insane-wmp12-album-art)  
[3] [«Защита» mp3 файлов на amazon.com / Информационная безопасность / Хабрахабр](http://habrahabr.ru/post/134523/)  

History
-------

This project originated as a sub-component for an older project I started in 2010, during which I only knew of a few other libraries, very few for .NET, even fewer for .NET open-source, and literally zero for VB.NET open-source.

Attempting to implement a full spec ID3 tag reader was a huge challenge for me (having no binary computer sciency knowledge at the time) but overall proved to be an incredible learning experience. Not being accustomed to open-source, it's initially difficult to share the results of this hard work so openly, but I've learned that it makes me happier to see just how useful my code might be to others. So please respect the license.

One thing that inspired me to keep me working on this project was an interesting discovery made in part by myself and a Russian blogger who published an article (in Russian) pointing out how digital music retailers such as Amazon use a type of audio steganography to store unique purchase identifiers (information about who purchased the music, with a full name/user account id linked to a purchase id) in the mp3 files by embedding the data inside PRIV frames of the ID3 tags which are ignored by nearly all music software [[3]](#references).
