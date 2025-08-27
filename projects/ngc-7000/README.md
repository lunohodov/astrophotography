# NGC 7000 - North America Nebula (The Great Blue)

## Image Acquisition 

I acquired the imaged in traveller-mode on a Bortle-3 site.

Telescope Specification:

- Model: Sky-Watcher Evostar 72ED
- Aperture: 72mm
- Focal Length: 336mm (with 0.8x reducer)
- F-ratio: 4.6

Camera Specification:

- Model: ZWO ASI533MM-Pro
- Pixel Size: 3.75𝞵m
- Pixel Array: 3008 x 3008
- Pixel Resolution: 2.31 arc-sec/pixel
- Cooling: -35C below ambient

Light Frames:

- 171x 180 seconds, bin 1x1, Astronomik H-ɑ 6nm, at -10°C
- 190x 180 seconds, bin 1x1, Astronomik OII 6nm, at -10°C
- 132x 300 seconds, bin 1x1, Astronomik SII 6nm, at -10°C

Total integration time is 29 hours and 3 minutes.

## Image Processing Walkthrough

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Pixelmator Pro](https://www.pixelmator.com/pro/).

### Analyzing the Data

First, I screened all light and calibration frames in *Blink*. There were minor issues caused by wind gusts but what troubled me more was that the subframes of an entire H-ɑ imaging session looked “milky”.

I opened *SubframeSelector*, specified camera and image scale. After ensuring my star detection parameters were reasonable, I measured the subframes.

It turned out the “milky” images had notably less stars (500 vs. 2000 on average). I suspect the cause to be poor transparency, thin clouds, light pollution, or a combination of all. As a result, I discarded an hour and a half of H-ɑ data.


### Weighted Batch Preprocessing Script 2.7.3

1. Reset everything
2. Load all lights
3. Load all flats
4. Load all darks (excluded flat-darks, though present)
5. Load the bias
6. Select maximum quality
7. Use *CosmeticCorrection* with Autodetect (HotSigma=4, ColdSigma=3)
8. Set an Output Pedestal of 200 (DN)
9. Drizzle 1x, Drop Shrink 0.90, Square

See [screenshots](./media/wbpp/).

Total execution time: 01h:10m:27s.

### The Plan

Before diving into processing the data, I had to decide what story I want my image to tell.

For that, I created a SHO image using ChannelCombination and looked at it. After some time, I came up with the following:

- Optimize the visibility of the emission bands to show nebula’s chemical composition.
- Accent on the ionization fronts of the star formation regions in the Cygnus Wall.
- Reveal detail in the interstellar dust separating the North America Nebula and the Pelican Nebula.

### Load and Examine All Master Frames

I loaded the master frames for all channels, cloned them and renamed the clones to `H`, `O`, and `S` respectively.

### Correcting the Stars

I wanted to fix unequal FWHMs and asymmetric star halos using BlurXTerminator (BXT) in correct-only mode.

However, I wanted BXT to benefit from being able to “see” all channels simultaneously. For this I used ChannelCombination to create a SHO image. Then ran BXT in correct-only mode.

Then I split the RGB channels again, replacing the existing `S`, `H` and `O` images with the corrected results.

In addition, I made a copy of `H`, the dominant channel, named it `L`, and put it aside.

### Remove the Stars

I removed the stars from all channels using StarXTerminator (SXT), with Unscreen Stars option disabled, and kept the starry images for later use.

### Channel Equalization

I applied LinearFit to the starless `H` and `S` images using `O` as reference.

### Gradient Correction

I skipped gradient correction because I could not see any gradients.

### Create Color Images

Loaded the `S`, `H`, and `O` images into ChannelCombination and created a SHO image.

In addition, used SetiAstro’s Perfect Palette Picker to create a Foraxx and a second SHO image. Named them `Final_Foraxx` and `Final_SHO` respectively. 

Used ScreenTransferFunction to examine the linked STF. It looked reasonable.

### Color Calibration & Normalization

This is a confusing one for me.

On one hand debating on colors makes no sense in false-color imaging.

On the other hand, SpectrophotometricColorCalibration has a narrowband-mode (used for HOO) and PixInsight’s team promotes using ColorCalibration for SHO images. In addition, there is the palette polarization technique.

Given the above, I decided to test the following:

- NarrowbandNormalization only.
- ColorCalibration with the entire nebula as white reference and structure detection disabled.
- Background neutralization without full calibration.

### Deconvolution

Applied BXT to the `L` image with:

- Sharpen Stars: 0%
- Manual PSF: 2.38
- Sharpen Non-stellar: 75%

### Noise Reduction

Applied NoiseXTerminator to `SHO`, `Final_SHO`, `Final_Foraxx`, and `L` images with:

- Intensity/color and Frequency separation unchecked
- Denoise: 0.75
- Iterations: 4

### Create Narrowband Stars

Used SetiAstro’s NB to RGB Star Combination script where:

- use `H_stars` and `O_stars` images
- Green Channel Blend Ratio: 0,3
- Star Stretch Factor: 4
- Color Boost: 1.25

### Stretch the Starless Color Image

In ScreenTransferFunction, I applied a linked auto-stretch and copied the values in HistogramTransformation (HT). With `SHO` selected, I opened HT’s preview and adjusted the shadow- and midtones sliders so that the normalized values in the sky background areas stayed around `0.1`. Then applied to `SHO`.

### Stretch the Starless Luminance Image