# NGC 2239 - Rosetta Nebula

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

* 50 x 300 seconds, bin 1, Optolong L-eXtreme, -10°C 
* 446 x 10 seconds, bin 1, Astronomik Lum3, -10°C 
* 318 x 60 seconds, bin 1, IDAS NB1, -10°C 

Total of 6 hours 42 minutes.


## Image Analysis

Blink showed the data looks great overall. I had to remove some subs because of
guiding errors but am quite happy with the data.


## Image Processing

All processing is done in [PixInsight](https://pixinsight.com) with final
touches done in [Acorn](https://flyingmeat.com/acorn/).


### WBPP v2.5.9

I wanted to have more control over the preprocessing, so for each filter, I had
a separate WBPP run.

* Reset all
* Load all lights
* Load all flats
* Load all darks
* Select the preset for maximum quality and no compromises
* Reference image to auto
* Set cosmetic correction for all frames
* Enable Local Normalization
* Enable Image Integration
* Enable Drizzle (Scale = 2, Drop Shrink = 0.9)
* Enable Autocrop


### Create Pristine Master Images

Load the 2x drizzled and auto-cropped masters in PixInsight, rename and save to
disk:

* `pristine_idas.xisf`
* `pristine_lextreme.xisf`
* `pristine_lum3.xisf`

Use StarAlignment to align our the images with each other.

Then use DynamicCrop to select a common crop level and trim the frames. This will void the astrometric solution.

Use the ImageSolver script and save to disk.

Now, these are the images we can use to restart our processing from scratch.
