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

- 171x 180 seconds, bin 1x1, Astronomik H-alpha 6nm, at -10°C
- 190x 180 seconds, bin 1x1, Astronomik OII 6nm, at -10°C
- 132x 300 seconds, bin 1x1, Astronomik SII 6nm, at -10°C

Total integration time is 29 hours and 3 minutes.

## Image Processing Walkthrough

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Acorn](https://flyingmeat.com/acorn/).

### Analyzing the Data

First, I screened all light and calibration frames in *Blink*. There were minor issues caused by wind gusts but what troubled me more was that the subframes of an entire H-alpha imaging session looked “milky”.

I opened *SubframeSelector*, specified camera and image scale. After ensuring my star detection parameters were reasonable, I measured the subframes.

It turned out the “milky” images had notably less stars (500 vs. 2000 on average). I suspect the cause to be poor transparency, thin clouds, light pollution, or a combination of all. As a result, I discarded an hour and a half of H-alpha data.


### Weighted Batch Preprocessing Script 2.7.3

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

### Load Master Images

I loaded the master images for all channels, cloned them and renamed the clones to `ha`, `oiii`, and `sii` respectively.

### Gradient Extraction

I skipped gradient correction because I could not see gradients.

### Separate Stars and Equalize Channels

I decided to separate nebulosity and stars in the linear stage and process them independently.

For that I used *StarXTerminator* on each channel image where:

- Generate Star Image: checked
- Unscreen Stars: unchecked
- Large Overlap: unchecked

I inspected the resulting images for artifacts, found none, then renamed the star images to `ha_stars`, `oiii_stars`, and `sii_stars`respectively.

Next, I had to make the channels statistically matching. I used *LinearFit* to equalize `oiii` and `sii` to `ha`. Please note, that doing this after star removal ensures that signal from the stars will not bias the background.

### Create Stars Color Image

For the stars color image I used *PixelMath* where:

- R: `h`
- G: `h*0.7 + o*0.3`
- B: `o`

Then I used *HistogramTransformation* to balance the colors.

Alternatively, one can use Seti Astro’s *NBtoRGBStarCombination*.

Finally, I applied *BlurXTerminator* with:

- Sharpen Stars: 0.40
- Adjust Star Halos: 0.00
- Sharpen Nonstellar: 0.00

### Create Starless SHO Color Image

I composed a Hubble Palette image with *ChannelCombination* where I put `s`, `h` and `o` in the Red, Green and Blue channels respectively. The resulting image, I renamed to `sho`.

### Starless SHO Deconvolution

I  *BlurXTerminator* with:

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

