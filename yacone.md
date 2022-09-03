# GSoC-2021 With Shaka Streamer
<div align='center' display='flex'>
	<img src="https://raw.githubusercontent.com/google/shaka-streamer/master/docs/source/shaka-streamer-logo.png">
	<!-- <img src="https://upload.wikimedia.org/wikipedia/commons/0/08/GSoC_logo.svg"> -->
</div>

## Project Information:
### Project Name: [Shaka Streamer Enhancements](https://summerofcode.withgoogle.com/projects/#6382405939625984)
#### Mentor: [Joey Parrish](https://github.com/joeyparrish)<br>Student: [Omer Yacine](https://github.com/meryacine)

## Project Description:
Shaka Streamer is a tool that simplifies media transcoding and packaging by utilizing FFmpeg and Shaka Packager and managing the command line arguments that should be fed to them using simple and reusable configuration files.

### The two features concerning this project:

- Adding a **`PeriodConcat`** node that will make Shaka Streamer capable of producing multi-period DASH manifests using the single-period manifests outputted by Shaka Packager(which yet doesn't support multi-period DASH). A workaround can be done for HLS since it doesn't support periods, by using **`#EXT-X-DISCONTINUITY`** tag as a separator between different periods.
- Bundling hermetic copies of FFmpeg and Shaka Packager and allowing the user to specify whether Shaka Streamer should use the hermetic versions or the system-wide versions instead.

## Work Done:
- [Multi-period for DASH (Merged)](https://github.com/google/shaka-streamer/commit/e6152fdfe175dcf97a733cc5765ce95fcdaa7d26)
This commit manipulated the structure of the input configuration to let it accept the new multi-period option. The field `multiperiod_inputs_list` was added to the input configuration. This commit also implemented the DASH period concatenation, in which, the streamer would manage multiple PackagerNodes and concatenate their output when they finish using python's native `xml` module.
- [Multi-period for HLS (Merged)](https://github.com/google/shaka-streamer/commit/a8ec6fc9c294da2ec7a72470c2cee910c3cde50b)
In this PR, a minimal HLS parser was implemented. It searches only for the tags we care about and keep the rest of the playlist untouched. After all the master and media playlists are parsed and represented as python objects, all the media playlists are then sorted out into the appropriate type(video-audio-subtitles) and get fed to one of the four concatenation methods. These methods will make the best concatenation choices based on a playlist's attributes: codec, resolution, channel layout, and language.
- [Bringing Windows Support (Merged)](https://github.com/google/shaka-streamer/commit/9a7e0ec1d5e0b5c3b67b2df97c68c0e43e0d86b7)
This commit has introduced a new `Pipe` object that would abstract POSIX pipes(which are made using the `os.mkfifo()` system call), Windows pipes(implemented as two named pipes between two processes and a server thread that moves the data between the two pipes), and file paths. With all the platform-dependent code being isolated away in a new file(`pipe.py`), the rest of the project could then use the `Pipe` object without needing to know the platform it operates on.
- [Offering hermetic platform-specific binaries (Merged)](https://github.com/google/shaka-streamer/pull/87)
This PR implements the integration between Shaka Streamer and the new PyPi package [**`shaka-streamer-binaries`**](https://pypi.org/project/shaka-streamer-binaries/). It also has a build script to automate building **`shaka-streamer-binaries`** for all our supported platforms, and a download script to download and sort out the latest versions of the binaries we need for the building process.
### Other relating changes:
- [Adding `channel_layouts` field to the pipeline config (Merged)](https://github.com/google/shaka-streamer/commit/5a5b53d3d301925d984efac1a9a5a5d7ebf5379d)
This commit helped simplifying the multi-period for HLS task. Prior to this, the implementation of the audio concatenation for HLS was more complex and would sometimes produce multiple media playlists with the same attributes but different period characteristics in some periods, which an HLS player could not differentiate between. In this commit, the field `channel_count` was removed and replaced with `channel_layouts` which offered more audio encoding flexibility.
#### All my PRs to Shaka Streamer can be found [here](https://github.com/google/shaka-streamer/pulls?q=author%3Ameryacine).
## TODO:
- [x] Writing documentation about the new package **`shaka-streamer-binaries`** in Shaka Streamer's docs.
- [x] Writing documentation on how to use multi-period concatenation and the prerequisites to carry out a successful concatenation.
- [x] Integrating the Streamer's multi-period support with Shaka Packager's HTTP PUT support.
- [x] Use `posixpath` module instead of `os.path` for period concatenation as these manifests will be used on the web on windows.