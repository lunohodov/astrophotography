# White Walker burning – Wolf-Rayet 134

## Image Acquisition

I acquired the data in traveler-mode from a Bortle-3 site. It took me two years and two telescopes to gather the data for this object.

Telescopes:

- T1: Sky-Watcher Evostar 72ED, 336mm (x0.8 reducer) at F4.6
- T2: Askar FRA400, 280mm (x0.7 reducer) at F3.9

Camera:

- Model: ZWO ASI533MM-Pro
- Pixel Size: 3.75𝞵m
- Pixel Array: 3008 x 3008
- Pixel Resolution: 2.31 arc-sec/pixel (T1) and 2.77 arc-sec/pixel (T2)
- Cooling: -35C below ambient

Light Frames:

- 50x 300 seconds, bin 1x1, Astronomik H-ɑ 6nm, at -10°C
- 377x 180 seconds, bin 1x1, Astronomik H-ɑ 6nm, at -10°C
- 377x 180 seconds, bin 1x1, Astronomik OIII 6nm, at -10°C

Total integration time is 58 hours and 21 minutes.

## Image Processing Walkthrough

All processing is done in [PixInsight](https://pixinsight.com).

## Preprocessing

### Analyzing the Data

I screened all calibration and light frames in Blink. There were minor issues caused by wind gusts and/or clouds. I trashed around 15-20 frames.

### Weighted Batch Preprocessing Script (v2.8.9)

1. Reset everything
2. Load all lights
3. Load all flats
4. Load all darks (excluded flat-darks, though present)
5. Load the bias
6. Select maximum quality
7. Use *CosmeticCorrection* with Autodetect (HotSigma=15, ColdSigma=3)
8. Set the Output Pedestal to automatic
9. No drizzle (decided to do it manually)

See [screenshots](./media/). Total execution time: 01h:43m:31s.

## Processing

### The Plan

First, I wanted to settle on my processing goals. For that, I created a HOO image using ChannelCombination and looked at it. Here what I came up with:

- Accent on the shell of ionized oxygen. Show as much filament structure as possible.
- Reveal the depths of the hydrogen nebulosity.
- Hydrogen signal dominates the field, however I’ve collected plenty of oxygen signal. Try to emphasize it.

### Examine the Master Frames

I loaded the master frames for H-alpha and Oxygen III and renamed to `ha` and `oiii` respectively.

The `ha` channel was OK. However, the `oiii` has a few black spots where the readout showed pixel values of 0. To visualize the issue, I used the following PixelMath:

- R: $T
- G: iif($T > 0, $T, 1)
- B: $T
- Destination’s color space set to `RGB Color`

Running the above PixelMath on both channels, showed a lot of black pixels (now in green). After some experimentation with BatchStatistics and manual calibration (ImageCalibration), it became clear the automatic Output Pedestal Settings option wasn’t enough. I changed it to `Literal value = 100` and ran WBPP again.

This fixed the issue. 



