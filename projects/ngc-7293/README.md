# NGC 7293 - Helix Nebula

## Image Acquisition 

Images are acquired in traveller-mode on a Bortle-3 site.

Telescope Specification:

* Model: Sky-Watcher Evostar 72ED
* Aperture: 72mm
* Focal Length: 420mm
* F-ratio: 5.7

Camera Specification:

* Model: ZWO ASI533MC-Pro
* Pixel Size: 3.75𝞵m
* Pixel Array: 3008 x 3008
* Pixel Resolution: 1.85 arcsec/pixel
* Cooling: -35C below ambient

Light Frames:

* 57 x 300 seconds, bin 1x1, Optolong L-eXtreme, 0°C

Total of 4 hours and 40 minutes.

Calibration Frames:

* Master Bias, bin 1x1, 0°C, unity gain, offset 70
* Master Dark(s), bin 1x1, 0°C, unity gain, offset 70
* Flats, bin 1x1, 0°C, unity gain, offset 70


## Image Analysis

Blink showed the data looks OK. I had to remove some subs because of clouds or elongated stars caused by wind gusts.

I decided to use SubframeSelector to gain a better understanding the data.

Open SubframeSelector, then enter camera and image scale data in the System Parameters section. Switch to Star Detection Preview mode and applied globally (Structure layers = 10, rest at defaults).

Use the resulting `structure_map` as mask (red) of the original view to verify there are the "right" number of stars. In addition, do the same but with `stars` view as mask.

Now that we have reasonable star detection parameters, measure the subframes. Chose the best image and append `_best` to it's filename to mark it for use as a reference frame.


## Image Processing

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Acorn](https://flyingmeat.com/acorn/).

### WBPP v2.6.2

* Reset all
* Load all lights
* Load all flats
* Load master dark
* Load master bias
* Select the preset for maximum quality and no compromises
* Set keyword to `NIGHT`
* Reference image to auto
* Set cosmetic correction for all frames
* Enable Local Normalization
* Enable Image Integration
* Enable Drizzle (Scale = 2, Drop Shrink = 0.9)
* Enable Autocrop

See [the screenshots](./media/wbpp/).

Executed in 14m and 23s - no errors.


## Setup Master RGB Image

Open the RGB master image and change its identifier to `work`.

### Remove Color Gradients in the Master RGB Image

Use DynamicBackgroundExtraction (Tolerance = 1, Default sample radius: 75px) to remove color gradients.

### Color Calibration

The `work` RGB image somehow lost its astrometric solution so I had to manually run the ImageSolver script.

Run SpectrophotometricColorCalibration with:

* White Reference = Average Spiral Galaxy
* QE Curve = Ideal QE Curve
* Red Filter = Sony CMOS R-UVIRCut / Opt. L-eXtreme
* Green Filter = Sony CMOS G-UVIRCut / Opt. L-eXtreme
* Blue Filter = Sony CMOS B-UVIRCut / Opt. L-eXtreme

### Deconvolution on Master RGB Image

Run the PSFImage script on our `work` image to get the X and Y FWHM star sizes: 5.16 and 4.64. Average for PSF Diameter.

Experiment with BlurXTerminator settings, then run with PSF Diameter = 4.9.

Run NoiseXTerminator with Denoise = 0.5.

### Take the Image Nonlinear and Normalize Color

Take the image nonlinear with HistogramTransformation or Bill Blanshan's unlinked stretch script.

Use StarXTerminator to go starless. Check the Unscreen Stars option.

Remove residual star artefacts with CloneStamp.

Run NarrowbandNormalization (Palette = HOO, Lightness = Off, BlendMode 3 and Blend Amount = 0.6, Oiii Boost = 1).

Rename `work` to `hoo`.

Open the ForaxxPaletteUtility script, set the number of channels to 2, configure for single image without stars, then for both Ha and Oiii select our `hoo` image. Execute.

Name the resulting image `foraxx`.

With PixelMath, blend `hoo` and `foraxx` using the following expression `0.5*hoo + 0.5*foraxx`. Rename resulting image to `work`.

### Create Color Masks

Use Bill Blanshan's RedMask script to isolate the red areas in the image. Blur, then apply CurvesTransformation to enhance.

With Bill Blanshan's BlueMask script, create a blue mask. Enhance with CurvesTransformation and blur.

Create a yellow mask with Bill Blanshan's YellowMask script. Blur, enhance with CurvesTransformation, then blur again.

Create a GAME mask to focus on the brighter portions of the nebula.

### Finish Processing the RGB Work Image

Apply the YellowMask, adjust with CurvesTransformation. Then run LocalHistorgamEqualization (Kernel Radius = 30, Contrast Limit = 2, Amount = 0.5, 8-bit histogram).

Apply the RedMask. Enhance with CurvesTransformation. Then run LocalHistorgamEqualization (Kernel Radius = 100, Contrast Limit = 2, Amount = 0.4, 8-bit histogram). Run HDRMultiscaleTransform (Number of layers = 8, Scaling Function = Gaussian 11, check Preserve hue and Lightness mask).

Apply the BlueMask, adjust with CurvesTransformation. Then run LocalHistorgamEqualization (Kernel Radius = 100, Contrast Limit = 2, Amount = 0.3, 12-bit histogram).

Apply the GAME mask and enhance with CurvesTransformation.

Remove the mask. Open GeneralizedHyperbolicStretch and apply judiciously to do a final stretch.

Run global NoiseXTerminator with Denoise = 0.7.


### Process the Stars

Adjust the stars-only image to reduce intensity and size, then boost the saturation using CurvesTransformation.


### Add the Stars Back

Use PixelMath and a screening equation:

    ~((~starless)*(~stars))


## Final Processing

Save the image as TIFF to disk. Open with Acorn and apply a final touch with
Curves and Levels to the image.

---

# Final Image

![FinalImage](./media/NGC-7293.jpeg?raw=true)

