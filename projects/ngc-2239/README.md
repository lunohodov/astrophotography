# NGC-2237 - Rosette Nebula

The Rosette Nebula is a HII region located near one end of a giant molecular cloud in the constellation of Monoceros.

This distinctive nebula, which some claim looks like a skull, has a hole in the middle that creates the illusion of its rose-like shape.

A survey with the Chandra X-ray Observatory revealed the center of the nebula hosts an open cluster of very bright and extremely hot stars. These stars, scattered within a dense molecular cloud, emit UV radiation that heats the surrounding gas to temperatures approximately ranging from 1 to 10 million C°. The stellar winds from the stars, have created the hole seen in the middle of the nebula. The pressure from these winds on the interstellar matter causes new stars to form.

The Rosette Nebula is about 5200 light years away from Earth and measures roughly 130 light years in diameter.

## Image Acquisition 

I took the images in traveller-mode on a Bortle-3 site.

Telescope Specification:

* Model: Sky-Watcher Evostar 72ED
* Aperture: 72mm
* Focal Length: 336mm (with 0.8x reducer)
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

### Blink

Initial evaluation with Blink shows good image data. I removed  a couple of low quality subframes due to tracking and focus errors.

### Subframe Selector

To better understand the data, I decided to analyse further with SubframeSelector. In System Parameters, I entered my image scale and camera gain.

First, I want to test star detection and refine if needed:

1. With Add Files, choose a single light frame
2. Switch the routine to Star Detection Preview (Structure layers = 10, the rest at its defaults) and apply globally
3. Apply the resulting `structure_map` as mask to the original view and verify the number of stars looks “right”
4. Apply the resulting `stars` as mask to the original view and verify the number of stars looks “right”

With reasonable star detection parameters, I add the files I want, switch back to Measure Subframes routine and apply globally.

PSF Signal Weight and Stars are used as the two determinant factors for quality with the following expressions by filter:

- Astronomik Lum3: `Stars >= 2600 && PSFSignal >= 0.13`
- Optolong L-eXtreme:
- IDAS NB1:



## Image Processing

All processing is done in [PixInsight](https://pixinsight.com) with final touches done in [Acorn](https://flyingmeat.com/acorn/).

### WBPP v2.7.0

For each filter, I had a separate WBPP run.

* Reset all
* Load all lights
* Load all flats
* Load all darks
* Select the preset for maximum quality and no compromises
* Reference image to auto
* Set cosmetic correction for all frames
* Enable Local Normalization
* Enable Image Integration
* Enable Drizzle (Scale = 1, Drop Shrink = 0.9)
* Enable Autocrop

Here diagnostic screenshots per filter:

- Astronomik Lum3
- Optolong L-eXtreme
- IDAS NB1