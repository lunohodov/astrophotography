# SH2 132 - Lion Nebula

## Image Acquisition 

Images are acquired in traveller-mode on a Bortle-3 site.

Telescope Specification:

* Model: Sky-Watcher Evostar 72ED
* Aperture: 72mm
* Focal Length: 336mm (0.8x reducer)
* F-ratio: 4.6

Camera Specification:

* Model: ZWO ASI533MC-Pro
* Pixel Size: 3.75𝞵m
* Pixel Array: 3008 x 3008
* Pixel Resolution: 2.3 arcsec/pixel
* Cooling: -35C below ambient

Light Frames:

* 324 x 120 seconds, bin 1x1, IDAS-NB1, 0°C
* 57 x 30 seconds, bin 1x1, IDAS-NB1, 0°C

Total of 11 hours 16 minutes taken in five imaging sessions.

Calibration Frames:

* Master Bias, bin 1x1, 0°C, unity gain, offset 70
* Master Dark(s), bin 1x1, 0°C, unity gain, offset 70
* Flats, bin 1x1, 0°C, unity gain, offset 70


## Image Analysis

Blink showed the data looks OK. I had to remove some subs because of clouds or elongated
stars caused by wind gusts.

I decided to also use SubframeSelector to have a better understanding of taken
images.

Opened SubframeSelector, then entered camera and image scale data in the
System Parameters section. Switched to Star Detection Preview mode and applied
globally (Structure layers = 10, rest at defaults). After that, I used the resulting
StructuresMap as mask (red) of the Original view to verify there are the "right"
number of stars.

Rejected all subframes with less than 5000 stars.

Open ImageIntegration. 

./2023-08-17: LIGHTS, FLATS
./2023-08-18: LIGHTS,
./2023-08-19: LIGHTS, FLATS
./2023-08-28: LIGHTS, FLATS
./2023-09-09: LIGHTS, FLATS, DARKFLATS
./2023-10-14: LIGHTS, FLATS



## Image Processing

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Acorn](https://flyingmeat.com/acorn/).

### WBPP v2.6.2

I could not acquire flats for one of the imaging sessions. However, because I
left the imaging train assembled, I decided to reuse flats taken the day before.

* Reset all
* Load all lights
* Load all flats
* Load all darks
* Select the preset for maximum quality and no compromises
* Set keyword to `NIGHT`
* Reference image to auto
* Set cosmetic correction for all frames
* Enable Local Normalization
* Enable Image Integration
* Enable Drizzle (Scale = 2, Drop Shrink = 0.9)
* Enable Autocrop

See [the screenshots](./media/wbpp/).

Executed in 1h 49m - no errors.

### Setup Master RGB Image

Open the RGB master image and change its identifier to `work`.


### Remove Color Gradients in the Master RGB Image

Use DynamicBackgroundExtraction to remove the color gradient where Tolerance = 10.


### Run Deconvolution on Master RGB Image

Run the PSFImage script on the RGB image to get the X and Y FWHM star sizes:
4.16 and 4.30. Sizes are pretty close.

Experiment with BlurXTerminator settings, then run it with PSF Diameter = 4.2.

Run NoiseXTerminator with Denoise = 0.5.


### Take the Image Nonlinear and Process

Take the image nonlinear with HistogramTransformation or Bill Blanshan's
unlinked stretch script.

Use StarXTerminator to go starless. Check the Unscreen Stars option.

Remove residual star artefacts with CloneStamp.

Run the NarrowbandNormalization script with Palette = HOO, Lightness = Ha,
Blend Mode 2 and Blend Amount = 0.6, OIII Boost = 1.

Open the ForaxxPaletteUtility script, set the number of channels to 2, check
option for single Foraxx image without stars, then in both Ha and Oiii select
our `work` image. Execute.

Do an SCNR to the green with 0.8 amount, then do global CurvesTransformation.

#### Create Color Masks

Use Bill Blanshan's RedMask script to isolate the red areas in the image. Use
CurvesTransformation to enhance then blur a bit. Do a final tweak of
CurvesTransformation.

Use Bill Blanshan's BlueMask script. Use CurvesTransformation to enhance,
then blur and do a final touch of CurvesTransformation.

Create a GAME mask to focus on the brighter portions of the nebula.

### Finish Processing the RGB Master Image

Apply the BlueMask, adjust with CurvesTransformation. Then run
LocalHistorgamEqualization with Kernel Radius = 120, Contrast Limit = 2, Amount
= 0.2 and 8-bit histogram.

Apply the RedMask. Enhance with CurvesTransformation. Then run
LocalHistorgamEqualization with Kernel Radius = 120, Contrast Limit = 2, Amount
= 0.4 and 8-bit histogram.

Remove the mask.

Run a global HDRMultiscaleTransform with Number of Layers = 7, Scaling Function
= Gaussian (11), and Lightness Mask checked.

Apply global NoiseXTerminator with Denoise = 0.7.

Apply GAMEMask and enhance with CurvesTransformation. Then run
LocalHistorgamEqualization with Kernel Radius = 64, Contrast Limit = 2, Amount =
0.3 and 8-bit histogram.

Use RangeSelection to create a RangeMask selecting most part of the nebula.
Invert the mask to target the dark area of the image, then apply it. Run NoiseXTerminator with Denoise = 0.7.

Open GeneralizedHyperbolicStretch and apply judiciously to do a final stretch.


### Process the Stars

Adjust the Stars Only image to reduce star intensity and size then boost the saturation using CurvesTransformation.


### Add the Stars Back

Use PixelMath and a screening equation:

    ~((~starless)*(~stars))


## Final Processing

Save the image as TIFF to disk. Open with Acorn and apply a final touch with
Curves and Levels to the image.

---

# Final Image

![FinalImage](./media/Sh2-132.jpeg?raw=true)

