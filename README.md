# Image Timestamper

## In one sentence

Command line Python script that 1) takes a photo or series of images (timelapse), 2) reads existing time data (if any), and 3) writes new time data based on offset values defined by user.

## Why we built this

When shooting images, it is often advantageous to use a 3rd-party GPS logger (e.g. phones because they're often more accurate at recording positioning than camera GPS chips).

Problem is, in order to join images with GPS tracks, the times in each log must match up (or be within 1 second of each other). ([We use Image Geotagger to do this](https://github.com/trek-view/image-geotagger)).

Oftentimes, cameras use the default camera time (time set by user on camera) to assign a timestamp a `DateTimeOriginal` metadata value to the image.

This is problematic when clocks move or moving between countries with different timezones where time is not updated. On phones this is less of a problem because they use remote time clock (NTP servers) to keep an accurate time.

We've also noticed some camera stitching software completely rewrites the `DateTimeOriginal` of images during stitching, replacing it with the datetime the image was stitched (which is often many days later).

Finally, other software can sometimes completely strip metadata from files (e.g ffmpeg when converting videos to frames).

Image Timestamper gives users a significant amount of flexibility to edit the timestamps of multiple images (usually timestamps) in one go using the command line.

_Note: if you need to adjust GPS log timestamps see [GPS Track Timestamper](https://github.com/trek-view/gps-track-timestamper)._

## How it works

1. You specify a photo file or series of timelapse photo files
2. You define how timestamps should be assigned
3. The script reads the photo / all of the photos in the directory
4. The script writes new timestamps into photos as `DateTimeOriginal` values.

## Requirements

### OS Requirements

Works on Windows, Linux and MacOS.

### Software Requirements

* Python version 3.6+
* [Pandas](https://pandas.pydata.org/docs/): python -m pip install pandas
* [Exiftool](https://exiftool.org/)

## Usage

```
python image-timestamper.py -m [MODE] [INPUT DIRECTORY PHOTO FILE] [OUTPUT DIRECTORY]
```

* mode (`-m`)
	- `manual`: doesn't require any existing data (**timelapse / photo**). For single photo you must specify start `DateTimeOriginal` using `--start_time` in `YYYY-MM-DD:HH:MM:SS` format. For timestamp must supply `--start_time` and the time offset for subsequent photos using `--interval` in seconds. Timelapse images in the directory specified will be ordered and processed in ascending time order (1-9) if  `DateTimeOriginal` values exist in all images, if no `DateTimeOriginal` values exist (for all images) the script will order and process using ascending filename order (A-Z).
	- `offset`: requires `DateTimeOriginal` (**timelapse / photo**). Must specify time `--offset` in seconds that should be applied to existing `DateTimeOriginal` (timelapse / photo) values. Can be positive or negative.
    - `inherit`: requires `GPSDateTime` value (**timelapse / photo**). Inherits `DateTimeOriginal` (photo / timelapse) from `GPSDateTime`.
	- `reverse`: requires `DateTimeOriginal` (**timelapse / photo**). Inherits `GPSDateTime` from `DateTimeOriginal`. **WARNING:** `GPSDateTime` is a much more accurate value for time, as this is reported from GPS atomic clocks. It is very unlikely you want to use this mode, unless your intention is to spoof the image time.
	
## Output

The photo file(s) outputted in specified directory will contain all original metadata, but with updated timestamps from script for `DateTimeOriginal` values.

## Quick start 

_Note for Windows users_

It is recommended you place `exiftool.exe` in the script directory. To do this, [download exiftool](https://exiftool.org/), extract the `.zip` file, place `exiftool(-k).exe` in script directory, and rename `exiftool(-k).exe` as `exiftool.exe`

If you want to run an existing exiftool install from outside the directory you can also add the path to the exiftool executable on the machine using either `--exiftool-exec-path` or `-e`.

_Note for MacOS / Unix users_

Remove the double quotes (`"`) around any directory path shown in the examples. For example `"OUTPUT_1"` becomes `OUTPUT_1`.

**Take a single photo (`INPUT/MULTISHOT_0611_000000.jpg`) and tag the image with `DateTimeOriginal` = `2020-01-01:00:00:01` then output (to directory `OUTPUT_1`)**

```
python image-timestamper.py -m manual --start_time 2020-01-01:00:00:01 "INPUT/MULTISHOT_0611_000000.jpg" "OUTPUT_1"
```

**Take a directory of images (`INPUT`) and tag the first image with `DateTimeOriginal` = `2020-01-01:00:00:01` with each subsequent photo in order having a time offset by 10 seconds for `DateTimeOriginal` value then output (to directory `OUTPUT_1`)**

```
python image-timestamper.py -m manual --start_time 2020-01-01:00:00:01 --interval 10 "INPUT" "OUTPUT_1"
```

**Take a single photo (`INPUT/MULTISHOT_0611_000000.jpg`) and subtract 5 minutes onto `DateTimeOriginal` value then output (to directory `OUTPUT_2`)**

```
python image-timestamper.py -m offset --offset -300 "INPUT/MULTISHOT_0611_000000.jpg" "OUTPUT_2"
```

**Take a directory of images (`INPUT`) and and subtract 5 minutes onto all images `DateTimeOriginal` value then output (to directory `OUTPUT_2`)**

```
python image-timestamper.py -m offset --offset -300 "INPUT" "OUTPUT_2"
```

**Take a single photo (`INPUT/MULTISHOT_0611_000000.jpg`) and inherit the `DateTimeOriginal` value from the images reported `GPSDateTime` then output (to directory `OUTPUT_3`)**

```
python image-timestamper.py -m inherit "INPUT/MULTISHOT_0611_000000.jpg" "OUTPUT_3"
```

**Take a directory of images (`INPUT`) and inherit all `DateTimeOriginal` values from the images reported `GPSDateTime` then output (to directory `OUTPUT_3`)**

```
python image-timestamper.py -m inherit "INPUT" "OUTPUT_3"
```

**Take a single photo (`INPUT/MULTISHOT_0611_000000.jpg`) and the image `GPSDateTime` match the value reported for its `DateTimeOriginal` then output (to directory `OUTPUT_4`)**

```
python image-timestamper.py -m reverse "INPUT/MULTISHOT_0611_000000.jpg" "OUTPUT_4"
```

**Take a directory of images (`INPUT`) and make each images `GPSDateTime` match the value reported for its `DateTimeOriginal` then output (to directory `OUTPUT_4`)**

```
python image-timestamper.py -m reverse "INPUT" "OUTPUT_4"
```

## Support 

We offer community support for all our software on our Campfire forum. [Ask a question or make a suggestion here](https://campfire.trekview.org/c/support/8).

## License

Image Timestamper is licensed under a [GNU AGPLv3 License](/LICENSE.txt).