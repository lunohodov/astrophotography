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
8. Set an Output Pedestal of 200 (DN)
9. Drizzle 1x, Drop Shrink 0.90, Square

See [screenshots](./media/wbpp/).

Total execution time: 01h:10m:27s.

### 3. Load Master Images

I loaded all master images and cloned them. Renamed the clones to `ha`, `oiii`, and `sii` respectively.

### 4. Gradient Extraction

Gradients were not visible so I skipped gradient correction.

### 5. Create SHO Image and Calibrate Color

I composed a Hubble Palette image with *ChannelCombination* where I put `sii`, `ha` and `oiii` in the Red, Green and Blue channels respectively. The resulting image, I renamed to `sho`.

My goal when calibrating the color was to best show the nebula’s chemical composition.

The following tools were used:

- *BackgroundNeutralization*
- *ColorCalibration* for intrinsic white reference
- *PixelMath* for palette polarization

#### BackgroundNeutralization

*ColorCalibration*, unlike *PhotometricColorCalibration* and *SpectrophotometricColorCalibration*, does not neutralize the background. So I went with this first.

Initially, I used the mode `Rescale as needed` and a background reference covering the darkest area in `sho`. This maximized the dynamic range usage but resulted in less foot-room (i.e. Blacks separation).

In the end, I switched to `Target background` and applied with:

- Target background set to `0.0010000`
- Reference image set to an aggregated image build from 4 previews of the dark areas of `sho` 

#### ColorCalibration

To optimize the visibility of the three emission bands (H-alpha, OIII, and SII) within the nebula, I needed to use a white reference that is an inherent part of the image. In other words – the nebula itself.

One process to allow using an intrinsic white reference is *ColorCalibration*.

However, there are a lot of stars inside the nebula and they affect the light measurement. To fix this, I used *StarXTerminator* to create a starless copy of `sho` and named it `white_reference`.

Then, I applied *ColorCalibration* to `sho` with:

- the white reference set to a preview in `white_reference` covering the nebula.
- the background reference set to a preview in `sho` covering the darkest area.
- disabled Structure Detection, although it’s not needed as `white_reference` has no stars.

#### PixelMath

Since H-alpha (Green) is the strongest of all three emission lines, the image still has a hue towards green. To further balance the colors, I used a technique called palette polarization. This shifts the chromatic representation towards SII (Red) and OII (Blue).

For that I used *PixelMath* to multiply the Red and Blue channels by a factor. However, to ensure the sky background stays neutral i.e. not turn magenta, I had to multiply the light from the objects only and exclude the background pedestal. 

To calculate the pedestal, I created a new independent image from the preview used as the background reference and named it `sho_bg`.

Then, I applied the following *PixelMath* to `sho`:

- R/K: `($T - med(sho_bg)) * k + med(sho_bg)`
- G: `$T`
- B: `($T - med(sho_bg)) * k + med(sho_bg)`
- Symbols: `k = 1.15`

### 6. Deconvolution

Used *BlurXTerminator* with:

- Sharpen Stars: 0.10
- Adjust Star Halos: 0.0
- Automatic PSF enabled
- Sharpen Nonstellar: 0.50

### 7. Create a Synthetic Luminance

To enhance the detail and reduce noise, I decided to employ a synthetic luminance.

For that I combined all the master lights using *ImageIntegration* with:

- Average combination
- Normalization: Additive with scaling
- Weights: SNR
- Disabled outlier rejection
- Rest at defaults

The above resulted in a monochrome image with optimal signal-to-noise ratio. I renamed the image to `lum` and saved it on disk.

Then, I applied *BlurXTerminator* with:

- Sharpen Stars: 0
- Adjust Star Halos: 0
- Automatic PSF
- Sharpen Nonstellar: 0.5

Last, I removed the stars with *StarXTerminator*.

### 8. Go Non-Linear

#### Stretching the SHO image

To blend the synthetic luminance and continue with further processing, I had to stretch both `sho` and `lum` images.

But before that, I extracted the `CIE L*` component of the still linear `sho` image, renamed it to `sho_lum_linear` and iconized it for later.

My plan was to separately process the nebulosity and the stars *after delinearization*. Hence, I wanted to keep the stars smaller and not stretch too much.

For that, I opened the *Statistics* tool and set it to track the current view. Then, I stretched `sho` using *ScreenTransferFunction* and *HistogramTransformation* while making sure the mean values are around the `0.20` mark.

#### Stretching the Luminance Image

The `lum` had to have a similar stretch as `sho`. All tutorials I watched did it freehand but I could not reproduce their results.

To get around this, I *LinearFit*-ed `lum` to the `sho_lum_linear` image I created before. Then stretched `lum` with the *HistogramTransformation* processes directly from `sho`’s History Explorer.

Now, at least to my eye, I had a similar stretch.

### 9. Blend Synthetic Luminance and SHO Images

To blend the synthetic luminance, I applied *ChannelCombination* to `sho` with:

- Color Space: CIE L\*a\*b
- L: `lum`
- all other channels disabled
- inheriting astrometric solution enabled

### 10. Go Starless

Applied *StarXTerminator* to `sho` with:

- Generate Star Image enabled
- Unscreen Stars enabled
- Large Overlap enabled

The above extracted the stars from `sho` into a new image `sho_stars`.

## Final Image

