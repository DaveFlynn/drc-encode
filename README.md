# drc-encode

CLI tool for making movies easier to hear by adding a balanced stereo track optimized for lower-volume playback.

This project is based on the work shared by Reddit user `smernt` in the [r/PleX thread](https://www.reddit.com/r/PleX/comments/9rc7sp/thought_id_share_some_ffmpeg_scripts_i_made_to/).

The goal here is to make that workflow easier to use as a simple command-line tool.

The current processing settings are based on my personal preference. Over time, the tool will be expanded to support more variations like the ones `smernt` showed in that post.

This tool exists because movies with 5.1 or 7.1 audio are often difficult to use in apartments, shared homes, or while children are sleeping. Dialogue can be too quiet while music and action scenes jump out too loudly.

## Requirements

- `ffmpeg` must be installed and available in your `PATH`
- `ffprobe` must be installed and available in your `PATH`
- `jq` must be installed and available in your `PATH`
- macOS, Linux, or another environment that can run Bash and FFmpeg

Check that FFmpeg is installed:

```bash
ffmpeg -version
```

Check that `jq` is installed:

```bash
jq --version
```

If either command fails, install the missing dependency first, then run `drc_encode` again.

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
./drc_encode <input-file> [language-code] [options]
```

Examples:

```bash
./drc_encode movie.mkv
./drc_encode movie.mkv --interactive
./drc_encode movie.mkv --tracks 2,1 --profile stereo-drc
```

Options:

- `--interactive`, `-i`: show detected audio streams and prompt for which tracks to encode
- `--tracks <list>`: encode selected audio tracks using the displayed 1-based track numbers, for example `--tracks 1,2`
- `--profile <profile>`: choose the encode profile; currently only `stereo-drc` is supported
- `[language-code]`: optional legacy override for the generated track language metadata

## What It Does

- Creates new stereo AAC tracks from the selected source audio streams
- Raises quieter parts of the soundtrack and evens out volume swings so dialogue is easier to hear and loud scenes are less jarring
- Places generated tracks before the original audio tracks
- Preserves language metadata on generated tracks when the source stream has it
- Copies existing video streams
- Copies existing audio streams
- Copies existing subtitle streams
- Writes the output as `<original-name>_Stereo_DRC.<ext>`

If encoding succeeds:

- the original input file is deleted
- a matching `.srt` file is renamed to match the new output filename

## Notes

- If no options are provided, the first audio stream is encoded with the default `stereo-drc` profile
- Generated track titles are derived from the source title or language when available
- The optional `language-code` argument is kept for backward compatibility, but normal usage should rely on source stream metadata
