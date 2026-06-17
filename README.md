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
./drc_encode movie.mkv --tracks 1 --profile stereo
./drc_encode movie.mkv --tracks 1 --profile nightmode
./drc_encode movie.mkv --tracks 1 --keep-original
./drc_encode movie.mkv --reorder-audio 2,4,3,1 --keep-original
./drc_encode movie.mkv --remove-audio 2 --keep-original
./drc_encode movie.mkv --tag-audio 2:title=Commentary --keep-original
```

Options:

- `--interactive`, `-i`: show detected audio streams and prompt for track selection
- `--tracks <list>`: encode selected audio tracks using the displayed 1-based track numbers, for example `--tracks 1,2`
- `--profile <profile>`: choose the encode profile
- `--reorder-audio [order]`: reorder existing audio tracks without creating a new DRC track
- `--remove-audio [list]`: remove existing audio tracks without creating a new DRC track
- `--tag-audio <spec>`: rewrite existing audio metadata without creating a new DRC track
- `--keep-original`: keep the original input file after a successful encode
- `[language-code]`: optional legacy override for the generated track language metadata

## Profiles

Available profiles:

- `stereo-drc`: the default opinionated low-volume profile. It applies compression, loudness normalization, and a volume boost.
- `stereo`: plain AAC stereo mix down without DRC processing.
- `nightmode`: center-channel-focused surround mix down for dialogue-heavy low-volume listening.

`nightmode` expects a surround source because it uses surround channel names like `FC`, `FL`, `FR`, `LFE`, `SL`, and `SR`. The script will reject `nightmode` for mono or stereo source tracks.

Interactive mode asks for tracks first, then asks for a compatible profile if `--profile` was not already provided:

```text
Detected audio streams:
1. a:0  eng  aac  5.1     -  "English"
2. a:1  eng  aac  stereo  -  "Commentary"

Select audio tracks to process, in output order [1]:
> 1,2

Available profiles for selected tracks:
1. stereo-drc
2. stereo

Select profile [stereo-drc]:
> stereo-drc
```

The profile list only includes profiles that work for all selected tracks. For example, `nightmode` is only offered when every selected track is surround audio.

## Track Selection

Track numbers use the list shown by `drc_encode`, not FFmpeg's internal zero-based numbering.

For example, if interactive mode shows:

```text
Detected audio streams:
1. a:0  eng  aac  5.1     -  "English"
2. a:1  eng  aac  stereo  -  "Commentary"
```

Then `--tracks 1` means the displayed first track, which is FFmpeg audio stream `a:0`.

Selecting more than one track creates one generated DRC track for each selected source track in the order you provide:

```bash
./drc_encode movie.mkv --tracks 1,2 --profile stereo-drc
```

That produces:

```text
1. DRC from track 1
2. DRC from track 2
3. original track 1
4. original track 2
```

Commentary tracks are handled the same as any other selected audio track. If you select a commentary track, a DRC version of the commentary track will be generated. If you only want to preserve commentary as an original track, do not include it in `--tracks`.

## Audio Management

The audio management options are remux-only operations. They copy video and subtitle streams, copy selected audio streams, and do not create new DRC/profile tracks.

Use `--reorder-audio` to change the order of existing audio tracks:

```bash
./drc_encode movie.mkv --reorder-audio 2,4,3,1 --keep-original
```

The reorder list must include every audio track exactly once. If the file has four audio tracks, `2,4,3,1` is valid, but `2,4,3` will fail because track `1` is missing.

If you omit the order, the script displays the detected audio tracks and prompts for the new order:

```bash
./drc_encode movie.mkv --reorder-audio --keep-original
```

Use `--remove-audio` to remove one or more existing audio tracks:

```bash
./drc_encode movie.mkv --remove-audio 2 --keep-original
./drc_encode movie.mkv --remove-audio 2,4 --keep-original
```

If you omit the list, the script displays the detected audio tracks and prompts for which tracks to remove:

```bash
./drc_encode movie.mkv --remove-audio --keep-original
```

The script will not allow removing every audio track.

Use `--tag-audio` to rewrite audio metadata by displayed track number:

```bash
./drc_encode movie.mkv --tag-audio 2:title=Commentary --keep-original
./drc_encode movie.mkv --tag-audio 2:language=eng --keep-original
./drc_encode movie.mkv --tag-audio 2:title=Commentary,language=eng --keep-original
```

You can repeat `--tag-audio` for multiple tracks:

```bash
./drc_encode movie.mkv \
  --tag-audio 1:title=Main_Audio \
  --tag-audio 2:title=Commentary \
  --keep-original
```

`--tag-audio` can also be combined with `--reorder-audio` or `--remove-audio`. The track number in the tag spec always refers to the original displayed audio track number from the input file.

## Batch Usage

You can run `drc_encode` in a shell loop:

```bash
for file in *.mkv; do
  ./drc_encode "$file" --tracks 1 --profile stereo-drc
done
```

This processes each file one at a time. Avoid `--interactive` in batch loops unless you want to answer the track-selection prompt for every file.

Be careful with `--tracks 1` in mixed batches. It always means the first detected audio track in each file, so it may select the wrong language if different files use different audio ordering.

## Re-running On Processed Files

Generated tracks are inserted before all existing audio tracks.

If an original file starts as:

```text
1. English 5.1
2. Commentary
```

And you run:

```bash
./drc_encode movie.mkv --tracks 1
```

The output becomes:

```text
1. English Stereo DRC
2. English 5.1
3. Commentary
```

If you later run `drc_encode` again on that output and select track `3`, the new generated commentary track will be inserted at the front:

```text
1. Commentary Stereo DRC
2. English Stereo DRC
3. English 5.1
4. Commentary
```

For now, select all tracks you want to generate in a single run when possible:

```bash
./drc_encode movie.mkv --tracks 1,2 --profile stereo-drc
```

## What It Does

- Creates new stereo AAC tracks from the selected source audio streams
- Raises quieter parts of the soundtrack and evens out volume swings so dialogue is easier to hear and loud scenes are less jarring
- Places generated tracks before the original audio tracks
- Preserves language metadata on generated tracks when the source stream has it
- Copies existing video streams
- Copies existing audio streams
- Copies existing subtitle streams
- Writes the output with a profile-specific suffix, such as `<original-name>_Stereo_DRC.<ext>`, `<original-name>_Stereo.<ext>`, or `<original-name>_NightMode.<ext>`

If encoding succeeds:

- the original input file is deleted by default
- a matching `.srt` file is renamed to match the new output filename by default

Use `--keep-original` to keep the original input file. When that option is used, a matching `.srt` file is copied instead of renamed.

## Notes

- If no options are provided, the first audio stream is encoded with the default `stereo-drc` profile
- Generated track titles are derived from the source title or language when available
- The optional `language-code` argument is kept for backward compatibility, but normal usage should rely on source stream metadata
