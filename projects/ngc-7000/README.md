# NGC 7000 - North America Nebula

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
- Sensor Resolution: 2.31 arc-sec/pixel
- Cooling: -35C below ambient

Light Frames:

- 171x 180 seconds, bin 1x1, Astronomik Ha 6nm, at -10°C
- 190x 180 seconds, bin 1x1, Astronomik OIII 6nm, at -10°C
- 132x 300 seconds, bin 1x1, Astronomik SII 6nm, at -10°C

Total integration time is 29 hours and 3 minutes.

## Preprocessing

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Pixelmator Pro](https://www.pixelmator.com/pro/).

### Analyzing the Data

First, I screened all light and calibration frames in *Blink*. There were minor issues caused by wind gusts but what troubled me more was that the subframes of an entire Ha imaging session looked “milky”. I ended up discarding an hour and a half of Ha data.

### Weighted Batch Preprocessing Script (v2.8.9)

1. Reset everything
2. Load all lights
3. Load all flats
4. Load all darks (excluded flat-darks, though present)
5. Load the bias
6. Select maximum quality
7. Use *CosmeticCorrection* with Autodetect (HotSigma=15, ColdSigma=3)
8. Set the Output Pedestal to 200 DN
9. No drizzle (done manually)

See [screenshots](./media/wbpp/).

Total execution time: 01h:10m:27s.

## Processing

### Load and Examine All Master Frames

I loaded the master frames for all channels and renamed to `H`, `O`, and `S` respectively.

### The Plan

Before diving into processing the data, I had to decide what story I want my image to tell.

For that, I created a Hubble palette (SHO) image using ChannelCombination and looked at it. After some time, I came up with the following:

- Optimize the visibility of the emission bands to show nebula’s chemical composition.
- Accent on the ionization fronts of the star formation regions in the Cygnus Wall.
- Reveal detail in the interstellar dust separating the North America Nebula and the Pelican Nebula.

### Drizzle Integration

I knew my data was under-sampled so I decided to improve sampling with DrizzleIntegration.

First, I had to determine the drizzle scale. I opened SubframeSelector and measured the FWHM for a set of calibrated frames for each filter. All values were way below 2.5 px, suggesting a drizzle scale of 3 (as per DrizzleIntegration’s documentation). However, since BlurXterminator (BXT) was trained on 2x-drizzled data, and a 3x drizzle would have resulted in a very large file, I opted for a scale of 2.

Then, I experimented with DrizzleIntegration and different parameters. I enabled “Fast drizzle” and used a ROI to significantly speed up testing. 

After each run, I checked the resulting `drizzle_weights` map where:

- I applied boosted STF and visually inspected the map for “blotches” and consistency.
- I used Statistics to check coverage (mean, minimum and maximum values). Higher values are better.

In the end I settled for the following parameters for 2x drizzle: 

- Scale: 2
- Drop shrink: 0.8
- Kernel function: Gaussian
- The rest at defaults

I also decided to use a drizzle 3x for a separate close up image of the Cygnus Wall:

- Scale: 3
- Drop shrink: 0.9
- Kernel function: Circular
- The rest at defaults

After running DrizzleIntegration on all channels, I inspected the resulting masters for artifacts. I found none.

Finally, I cropped the images to the same geometry, names them `H`, `S`, and `O` respectively and saved them to disk.

### Correct Cross-Channel Aberrations and Deconvolution

While I planned to process channels separately, running BXT on a color image allows it to “see” all channels simultaneously and apply cross-channel corrections.

Hence, I created a temporary SHO color image using ChannelCombination.

First, I applied BXT in “Correct Only” mode. Then a second time with the following parameters:

- Sharpen Stars: 0.20
- Adjust Star Halos: 0.10
- Automatic PSF: checked
- Sharpen Nonstellar: 0.55

### Split Channels and Correct Gradients

Testing, showed that applying GradientCorrection on separate channels yielded best results. Hence I split the temporary SHO image back to `S`, `H` and `O` and discarded it.

Then applied GradientCorrection to each channel with default parameters where:

- Automatic convergence: checked
- Structure Protection threshold: 0.05 (Ha & SII) and 0.10 (OIII)

### Plate Solve the Channels

With the ImageSolver script, I solved the celestial coordinates of the `H` channel, then used CopyAstrometricSolution to copy them to the other channels.

### Remove Stars

I removed the stars from each channel using StarXTerminator (SXT) with the following parameters:

- Generate Star Image: checked
- Unscreen Stars: unchecked
- Large Overlap: checked

Checked each resulting starless image for artifacts and found none. The star images (`H_Stars`, `O_Stars`, and `S_Stars`) I kept for later.

### Channel Equalization

To equalize the channel intensity, I used LinearFit on starless images with the `H` channel set as reference.

Rationale: In [Part 4](https://www.youtube.com/watch?v=TYAgHnzs67Y) of PixInsight’s official Narrowband workflow tutorial (same object!), the authors assert that when using (intrinsic) ColorCalibration, the stars affect the light measurement and use a starless image copy as reference. Similarly, I applied LinearFit on starless images.

### Make SHO Color Image

Used ChannelCombination to create a SHO color image.

### SHO Narrowband Normalization

Used NarrowbandNormalization with the following parameters:

- Palette: SHO
- Lightness: Off
- SCNR: 0.550
- OIII Boost: 1.15
- SII Boost: 1.0
- Shadow Point: 0.75

### Stretching the SHO Color Image

Initial stretching was done using SetiAstro’s StatisticalStretch script:

- Target Median: 0.15
- Automatic Convergence: checked
- Normalize Image Range: checked
- Linked Stretch: checked

### Reducing Noise in the Starless SHO Image

Even after the v2 release of NoiseXTerminator (NXT), many narrowband workflows apply noise reduction on monochromatic images prior to channel combination. With the new version, this is no longer needed.

Hence, I decided perform noise reduction after channel combination and use NXT’s intensity/color separation mode.

After consulting the documentation and experimenting, I settled for the following parameters:

- Intensity/color separation: checked
- Frequency separation: checked
- Denoise HF Intensity: 0.55
- Denoise HF Color: 0.78
- Denoise LF Intensity: 0.9
- Denoise LF Color: 0.9
- HF/LF Pixel Scale: 6
- Iterations: 2

Then, I saved the resulting image to disk in TIFF format.

### Other Color Image Enhancements

I opened the exported TIFF file in Pixelmator Pro, then applied various enhancements such as Curves, Levels, Saturation, etc.

Then exported the resulting image as `SHO_Starless.tiff`.

### Creating the Stars Image

For the stars image, I used SetiAstro’s “NB to RGB Star Combination Tool” with:

- Ha Stars Image: H_Stars
- OIII Stars Image: O_Stars
- SII Stars Image: S_Stars
- Apply Star Stretch: checked
- Stretch Factor: 5
- Color Boost: 1.15

I named the resulting image as `SHO_Stars`.

### Screening Back the Stars

Switching to PixInsight, I opened `SHO_Starless.tiff` and used the ScreenStars script from Cosmic Photons to blend the stars together with the starless image.

### Exporting the Image for Publication

1. Used the ICCProfileTransformation script to embed the `sRGB IEC61966-2.1` color profile.
2. Cropped the image to more pleasing dimensions.

## Final Image

![FinalImage](./media/NGC7000_SHO.jpeg?raw=true)

