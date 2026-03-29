# drc-encode

CLI tool for making movies easier to hear by adding a balanced stereo track optimized for lower-volume playback.

This project is based on the work shared by Reddit user `smernt` in the [r/PleX thread](https://www.reddit.com/r/PleX/comments/9rc7sp/thought_id_share_some_ffmpeg_scripts_i_made_to/).

The goal here is to make that workflow easier to use as a simple command-line tool.

The current processing settings are based on my personal preference. Over time, the tool will be expanded to support more variations like the ones `smernt` showed in that post.

This tool exists because movies with 5.1 or 7.1 audio are often difficult to use in apartments, shared homes, or while children are sleeping. Dialogue can be too quiet while music and action scenes jump out too loudly.

## Requirements

- `ffmpeg` must be installed and available in your `PATH`
- macOS, Linux, or another environment that can run Bash and FFmpeg

Check that FFmpeg is installed:

```bash
ffmpeg -version
```

If that command fails, install FFmpeg first, then run `drc_encode` again.

## Installation

Make the script executable:

```bash
chmod +x drc_encode
```

You can run it from the project directory:

```bash
./drc_encode movie.mkv
```

Or move it somewhere in your `PATH` so you can run it from anywhere:

```bash
mv drc_encode /usr/local/bin/drc_encode
```

Then run it like this:

```bash
drc_encode movie.mkv
```

## Usage

Run the script directly:

```bash
./drc_encode <input-file> [language-code]
```

Examples:

```bash
./drc_encode movie.mkv
./drc_encode movie.mkv eng
```

## What It Does

- Creates a new stereo AAC track from the first audio stream
- Raises quieter parts of the soundtrack and evens out volume swings so dialogue is easier to hear and loud scenes are less jarring
- Copies existing video streams
- Copies existing audio streams
- Copies existing subtitle streams
- Writes the output as `<original-name>_Stereo_DRC.<ext>`

If encoding succeeds:

- the original input file is deleted
- a matching `.srt` file is renamed to match the new output filename

## Notes

- The optional `language-code` is written to the new DRC audio track metadata
- If no language code is provided, the new track is titled `2.0 Stereo DRC`
- If a language code is provided, the new track is titled `<language-code> 2.0 Stereo DRC`
