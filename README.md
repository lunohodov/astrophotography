# My Journey Into Astrophotography

Here I share images and learnings I have made on my journey
into Astrophotography.

Bear with me.


## Imaging Projects

- [NGC-3324 - Gabriela Mistral Nebula](projects/ngc-3324)
- [NGC-7000 - North America Nebula](projects/ngc-7000)
- [NGC-7293 - Helix Nebula](projects/ngc-7293)
- [SH2-132 - Lion Nebula](projects/sh2-132)


## Project README Structure

Each imaging project follows a common document structure:

1. **Image Acquisition** — capture setup and data summary
   - Telescope Specification (model, aperture, focal length, f-ratio)
   - Camera Specification (model, pixel size, array, resolution, cooling)
   - Light Frames (count, exposure, binning, filter, temperature)
   - Calibration Frames *(where applicable)* — bias, darks, flats

2. **Image Analysis** *(most projects)* — quality screening with Blink and/or
   SubframeSelector; records rejected frames and reasons

3. **Preprocessing / Image Processing** — step-by-step PixInsight processing log
   covering: WBPP, gradient correction, deconvolution (BlurXTerminator), noise
   reduction (NoiseXTerminator), star removal (StarXTerminator), channel
   equalization, color combination, stretching, mask-based enhancements, star
   processing, and final recombination via PixelMath

4. **Final Processing** *(some projects)* — finishing touches applied in an
   external editor (Acorn or Pixelmator Pro)

5. **Final Image** — embedded JPEG of the finished result
