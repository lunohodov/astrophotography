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

All processing is done in [PixInsight](https://pixinsight.com) with final touches in [Pixelmator Pro](https://www.pixelmator.com/pro/).

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

I loaded the master images for all channels, cloned them and renamed the clones to `H`, `O`, and `S` respectively.

### Gradient Extraction

I skipped gradient correction because I could not see any gradients.