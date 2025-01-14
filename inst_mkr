#!/bin/bash
# THANKS ^^
# Function to display a box with a message
p() {
    echo "┌────────────────────────────────────────────────────┐"
    echo "│ $1"
    echo "└────────────────────────────────────────────────────┘"
}

# Help function to display the guide for users
h() {
    echo "Help: Here's a quick guide to understand the options:"
    echo "1. Removing Lower Frequencies: Excludes frequencies below the specified range (e.g., 0-100Hz)."
    echo "2. Removing Mid Frequencies: Excludes mid-range frequencies (e.g., 300-1000Hz)."
    echo "3. Removing Higher Frequencies: Excludes frequencies above a certain threshold (e.g., 10000Hz)."
    echo "4. Isolating Frequencies: Instead of removing, you can isolate a certain frequency range to keep in the output."
    echo "5. Normalization: Adjusts the volume of the final output to standard levels."
    echo ""
    echo "Press 'H' or 'h' at any point to view this help."
}

# Function to prompt the user for input and store the result in a variable
g() {
    local prompt=$1
    local var_name=$2
    while true; do
        p "$prompt (Press 'H' for help)"
        read -p "Input: " input
        if [[ "$input" == "H" || "$input" == "h" ]]; then
            h
            continue
        fi
        eval "$var_name=\"$input\""
        break
    done
}

# Ask for the input file path and check if the file exists
while true; do
    g "Please enter the path of the input MP3 file:" f
    if [ ! -f "$f" ]; then
        p "[ERROR]: Input file does not exist. Please check the path and try again."
    else
        break
    fi
done
d=$(dirname "$f")

# Ask if the user wants to use the default options
g "Would you like to use the default options for convenience? (y/n)" use_defaults

if [ "$use_defaults" == "y" ]; then
    # Use default options
    base_name=$(basename "$f" | sed 's/\.[^.]*$//')
    o="$d/${base_name}_INST.${f##*.}"
    l="y"
    L="0-80"
    m="n"
    M=""
    h="n"
    H=""
    n="n"
else
    # Ask about frequency ranges for processing
    g "Would you like to process lower frequencies? (y/n)" l
    if [ "$l" == "y" ]; then
        while true; do
            g "Please enter the frequency range you want to isolate or remove (e.g., 0-100 Hz):" L
            if [[ "$L" =~ ^[0-9]+-[0-9]+$ ]]; then
                break
            else
                p "[ERROR]: Invalid frequency range format. Please use the format 'low-high' (e.g., 0-100)."
            fi
        done
    fi

    g "Would you like to process mid frequencies? (y/n)" m
    if [ "$m" == "y" ]; then
        while true; do
            g "Please enter the frequency range you want to isolate or remove (e.g., 300-1000 Hz):" M
            if [[ "$M" =~ ^[0-9]+-[0-9]+$ ]]; then
                break
            else
                p "[ERROR]: Invalid frequency range format. Please use the format 'low-high' (e.g., 300-1000)."
            fi
        done
    fi

    g "Would you like to process higher frequencies? (y/n)" h
    if [ "$h" == "y" ]; then
        while true; do
            g "Please enter the frequency range you want to isolate or remove (e.g., 10000-20000 Hz):" H
            if [[ "$H" =~ ^[0-9]+-[0-9]+$ ]]; then
                break
            else
                p "[ERROR]: Invalid frequency range format. Please use the format 'low-high' (e.g., 10000-20000)."
            fi
        done
    fi

    # Ask for output file name
    g "Please enter the name for the output file (without extension):" o
    o="$d/$o.${f##*.}"

    # Ask if normalization is needed
    g "Do you want to normalize the output audio? (y/n)" n
fi

# Processing: Remove vocals (difference between left and right channels)
p "Processing: Removing vocals..."
ffmpeg -i "$f" -af "pan=stereo|c0=c0-c1|c1=c1-c0" "$d/SIL_DIFF.flac" -loglevel quiet

# Processing: Isolate or remove lower frequencies (if specified) and apply +8dB boost
if [ "$l" == "y" ]; then
    p "Processing: Isolating lower frequencies $L and applying +8dB boost..."
    ffmpeg -i "$f" -af "highpass=f=$(echo $L | cut -d'-' -f1),lowpass=f=$(echo $L | cut -d'-' -f2),volume=8dB" "$d/SIL_Lower.flac" -loglevel quiet
fi

# Processing: Isolate or remove mid frequencies (if specified)
if [ "$m" == "y" ]; then
    p "Processing: Isolating mid frequencies $M..."
    ffmpeg -i "$f" -af "highpass=f=$(echo $M | cut -d'-' -f1),lowpass=f=$(echo $M | cut -d'-' -f2)" "$d/SIL_Mids.flac" -loglevel quiet
fi

# Processing: Isolate or remove higher frequencies (if specified)
if [ "$h" == "y" ]; then
    p "Processing: Isolating higher frequencies $H..."
    ffmpeg -i "$f" -af "highpass=f=$(echo $H | cut -d'-' -f1),lowpass=f=$(echo $H | cut -d'-' -f2)" "$d/SIL_Higher.flac" -loglevel quiet
fi

# Combining the processed temporary files
p "Mixing: Combining the processed files..."
if [ "$l" == "y" ] && [ "$m" == "y" ] && [ "$h" == "y" ]; then
    ffmpeg -i "$d/SIL_Lower.flac" -i "$d/SIL_Mids.flac" -i "$d/SIL_Higher.flac" -i "$d/SIL_DIFF.flac" -filter_complex "[0][1][2][3]amix=inputs=4:duration=longest" "$o" -loglevel quiet
elif [ "$l" == "y" ] && [ "$m" == "y" ]; then
    ffmpeg -i "$d/SIL_Lower.flac" -i "$d/SIL_Mids.flac" -i "$d/SIL_DIFF.flac" -filter_complex "[0][1][2]amix=inputs=3:duration=longest" "$o" -loglevel quiet
elif [ "$l" == "y" ] && [ "$h" == "y" ]; then
    ffmpeg -i "$d/SIL_Lower.flac" -i "$d/SIL_Higher.flac" -i "$d/SIL_DIFF.flac" -filter_complex "[0][1][2]amix=inputs=3:duration=longest" "$o" -loglevel quiet
elif [ "$m" == "y" ] && [ "$h" == "y" ]; then
    ffmpeg -i "$d/SIL_Mids.flac" -i "$d/SIL_Higher.flac" -i "$d/SIL_DIFF.flac" -filter_complex "[0][1][2]amix=inputs=3:duration=longest" "$o" -loglevel quiet
fi

# If normalization was requested, apply it
if [ "$n" == "y" ]; then
    p "Normalizing output..."
    ffmpeg -i "$o" -filter:a loudnorm "$o" -loglevel quiet
fi

# Cleanup temporary files
p "Cleaning up temporary files..."
rm -f "$d/SIL_Lower.flac" "$d/SIL_Mids.flac" "$d/SIL_Higher.flac" "$d/SIL_DIFF.flac"

# Done!
p "Done! Your output file is saved as $o."
