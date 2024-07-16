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

- 44x 180 seconds, bin 1x1, Astronomik H-alpha 6nm, at -10°C (excluding 32x discarded)
- 86x 180 seconds, bin 1x1, Astronomik OII 6nm, at -10°C

Total integration time is 6 hours and 30 minutes.

Calibration Frames:
- 26x Flats per filter
- 26x Flat-Darks per filter

## Image Processing Walkthrough

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Acorn](https://flyingmeat.com/acorn/).

### 1. Analyzing the Data

First, I screened all light and calibration frames in *Blink*. There, I saw a couple of minor issues caused by wind gusts. But what troubled me more was that the subframes of a certain imaging session looked “milky”.

Then I opened *SubframeSelector*, specified camera and image scale, and after ensuring my star detection parameters were reasonable, I measured the subframes.

Unfortunately, the “milky” images had notably less stars (500 vs. 2000 on average). I suspect the cause to be poor transparency, thin clouds, light pollution, or a combination of all. As a result, I discarded the whole session. An hour and a half of Hɑ data went to waste.

Last, I chose the best subframe for each filter and appended `_best` to its filename. This way I can use them as a reference if I need to troubleshoot something later.

### 2. Weighted Batch Preprocessing Script 2.7.3

1. Reset everything
2. Load all lights
3. Load all flats
4. Load all darks, including flat-darks
5. Load the bias
6. Select maximum quality
7. Use *CosmeticCorrection* based on darks
8. No *Drizzle*

See [screenshots](./media/wbpp/).

Total execution time: 00h:16m:29s.

### 3. Load Master Images

Here, I loaded all master images, cloned them and renamed Hɑ to `ha`, OII to `o3`.

### 4. Gradient Extraction

Gradients were not visible but I decided to run the *GradientCorrection* tool on each image. With trial and error I saw the defaults worked well on `ha`. However, `o3` didn’t benefit much so I skipped it.

### 5. Create HOO Image and Do Linear Processing

I used *ChannelCombination* to create a HOO image. Then, on the resulting image, I applied *BlurXTerminator* in a correct-only mode.

Next, was *SpectrophotometricColorCalibration* in narrowband-mode. On top of that, I ran *NarrowbandNormalization* script with {BlendMode: Mode1; BlendAmount: 0.6; the rest at defaults}.

Then came *NoiseXTerminator* with {Denoise: 0.5; Detail: 0.15}.

### 6. Create Starless and Stars-Only Images

### 7. Starless Processing

### 8. Stars-Only Processing

### 9. Blend Starless with Stars-Only and Do Final Touches

## Final Image

![FinalImage](./media/NGC-7000-hoo_v2.jpeg?raw=true)

