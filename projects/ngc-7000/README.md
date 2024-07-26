# NGC 7000 - North America Nebula (The Great Blue)

## Image Acquisition 

Images are acquired in traveller-mode on a Bortle-3 site.

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

- 171x 180 seconds, bin 1x1, Astronomik H-alpha 6nm, at -10°C
- 190x 180 seconds, bin 1x1, Astronomik OII 6nm, at -10°C
- 132x 300 seconds, bin 1x1, Astronomik SII 6nm, at -10°C

Total integration time is 29 hours and 3 minutes.

## Image Processing Walkthrough

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Acorn](https://flyingmeat.com/acorn/).

### 1. Analyzing the Data

First, I screened all light and calibration frames in *Blink*. There were minor issues caused by wind gusts but what troubled me more was that the subframes of an entire H-alpha imaging session looked “milky”.

I opened *SubframeSelector*, specified camera and image scale. After ensuring my star detection parameters were reasonable, I measured the subframes.

It turned out the “milky” images had notably less stars (500 vs. 2000 on average). I suspect the cause to be poor transparency, thin clouds, light pollution, or a combination of all. As a result, I discarded an hour and a half of H-alpha data.


### 2. Weighted Batch Preprocessing Script 2.7.3

1. Reset everything
2. Load all lights
3. Load all flats
4. Load all darks, including flat-darks
5. Load the bias
6. Select maximum quality
7. Use *CosmeticCorrection* with Autodetect (HotSigma=4, ColdSigma=3)
8. Drizzle 1x, Drop Shrink 0.90, Square

See [screenshots](./media/wbpp/).

Total execution time: 01h:02m:22s.

### 3. Load Master Images

I loaded all master images and cloned them. Renamed the clones to `ha`, `oii`, and `sii` respectively.

### 4. Gradient Extraction

Gradients were not visible so I skipped gradient correction.

### 5. Create SHO Image and Calibrate Color

I composed a Hubble Palette image with *ChannelCombination* where I put `sii`, `ha` and `oii` in the Red, Green and Blue channels respectively. The resulting image, I renamed to `sho`.

My goal when calibrating the color was to best show the nebula’s chemical composition.

The following tools were used:

- *ColorCalibration* for intrinsic white reference
- *BackgroundNeutralization*
- *PixelMath* for palette polarization

#### ColorCalibration

To optimize the visibility of the three emission bands (H-alpha, OII, and SII) within the nebula, I needed to use a white reference that is an inherent part of the image i.e. the nebula itself.

Hence, I picked *ColorCalibration* because it allows us specify an intrinsic white reference.

However, there are a lot of stars inside the nebula and they affect the light measurement. To fix this, I used *StarXTerminator* to create a starless copy of `sho` and named it `white_reference`.

Next, I applied *ColorCalibration* to `sho` with:

- the white reference set to a preview in `white_reference` covering the nebula.
- the background reference set to a preview in `sho` covering the darkest area.
- disabled Structure Detection, although `white_reference` has no stars.

#### BackgroundNeutralization

*ColorCalibration*, unlike other similar processes such as *PhotometricColorCalibration* and *SpectrophotometricColorCalibration*, does not neutralize the background.

To adjust the background, I applied *BackgroundNeutralization* with background reference set to a preview in `white_reference` covering the darkest area.

#### PixelMath

Since H-alpha (Green) is the strongest of all three emission lines, the image still has a hue towards green. To further balance the colors, I used a technique called palette polarization. This shifts the chromatic representation towards SII (Red) and OII (Blue).

For that I used *PixelMath* to multiply the Red and Blue channels by a factor. However, to ensure the sky background stays neutral i.e. not turn magenta, I had to multiply the light from the objects only and exclude the background pedestal. 

To calculate the pedestal, I created a new independent image from the preview used as the background reference and named it `sho_bg`.

Then, I applied the following *PixelMath* to `sho`:

- R/K: `($T - med(sho_bg)) * k + med(sho_bg)`
- G: `$T`
- B: `($T - med(sho_bg)) * k + med(sho_bg)`
- Symbols: `k = 1.5`

### 6. Deconvolution

Used *BlurXTerminator* with:

- Sharpen Stars: 0.10
- Adjust Star Halos: 0.07
- Automatic PSF enabled
- Sharpen Nonstellar: 0.50

## Final Image

