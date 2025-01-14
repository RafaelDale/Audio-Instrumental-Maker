# Audio Instrumental Maker

## Overview
This Bash script processes audio files to isolate or remove specific frequency ranges, normalize audio, and remove vocals. It utilizes **FFmpeg** for audio processing and provides an interactive command-line interface for user input.

## Features
- **Remove Vocals**: Removes vocals by isolating the difference between left and right audio channels. Note that background vocals will be present as they are typically not centered and therefore not removed.
- **Frequency Processing**:
  - Isolates **low**, **mid**, or **high** frequencies and retain them in the output audio file, note that low or bass frequencies are boosted to 8dB as no boost makes them inaudible.
- **Normalization**: Adjust the volume of the output to standard levels to make quiet parts heard and loud parts slightly quieter.
- **Custom or Default Options**: Use default settings or define custom frequency ranges and output names. (Under testing)

## Requirements
- **FFmpeg**: Ensure FFmpeg is installed and available in your system's PATH.
- **Bash Shell**: The script runs in a Unix-based environment.

## Usage
1. Clone or download the script:
   ```bash
   git clone <repository-url>
   cd <repository-folder>
   ```
2. Run the script:
```
./script.sh
```

3. Follow the interactive prompts:

Input the path to your audio file (e.g., /sdcard/Music/Sample.mp3 or C:/Users/User/Downloads/Music.flac or anything that's audio and is existent on the storage the script is located).

Choose whether to use default settings or define custom options.




# Options

Lower Frequencies: Isolate a range (e.g., 0-80 Hz).

Mid Frequencies: Isolate a range (e.g., 300-1000 Hz).

High Frequencies: Isolate a range (e.g., 10000-20000 Hz).

Normalization: Enable or disable normalization for the output.


# Output

The processed file is saved in the same directory as the input file with a custom or default name.

Temporary files are cleaned up automatically.


# Example

Input

-- **Input file**
```
example.mp3
```

Low frequency range: 0-80 Hz
Normalize: Yes

-- **Output**

Processed file: example_INST.mp3


# License

This script is open-source and free to use.



