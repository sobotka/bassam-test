**What it does?**

The “Default” view transform, also known as the specification transfer curve for sRGB was designed for displays and **never designed to be used for raytracing / rendering**.

This set of transforms adds a photographic view transform for Blender. It is the ACES Shaper LMT that grabs ten stops below middle grey
and six and a half stops over middle grey. Middle grey is pegged at 0.18, which maps to approximately 0.6
display referred. Simply by using it your can elevate your work by an order of a magnitude. To quote Alex Fry:

> "With modern physically-based renderers, realism suffers if things aren't set up in a way that mirrors the real > world. Therefore, no matter how physically accurate your renderer is, if you build a universe where the sun 
> outside, and a character inside both have to live in the narrow band between 0->1, nothing will respond
> realistically.”

**How do I use it?**

Git clone this repository, backup your */blender/bin/[VERSION]/datafiles/colormanagement* (*C:\Program Files\Blender Foundation\Blender\[VERSION]\datafiles\colormanagement* on windows) directory, and replace or link the repository to its place. The view is currently located in the colour management panel of the Scene panel in Properties.

Given that the dynamic range of capture of your imagery is immediately augmented by an order of a magnitude, you will have to adjust your lighting to compensate. This means adding significantly more light to achieve normal real-world values. When lighting under the default sRGB display transform, your lighting was artificially tamped down, and as a result there is a very good chance that your lighting was bogus, as well as many of your material decisions.

**What are the transforms?**

**Views**
 * *Wide Dynamic Range View* This is the primary view mapping middle grey to 0.18 which ends up approximately 0.60 scene referred linear.
 the basic view covers sixteen and a half stops of latitude, with ten stops down and six and a half stops above
 middle grey.
 * *Raw* This is an untransformed dump of the scene referred values to the display. Effectively meaningless.
 * *sRGB* This is the sRGB specification transfer curve. Called *Default* in Blender's default configuration,
 renamed here for clarity. It covers approximately eight and a bit stops of latitude, mapping middle grey 0.18 to
 approximately 0.467 scene referred. It covers six and a bit stops down to approximately two and a bit stops over
 middle grey. It is unsuitable for a rendering view.
 
**Looks**
 * *None* This removes any additional transforms on the view.
 * *Scaled sRGB Grey variants* Some folks were having fits with the fact that the default view maps 0.18 middle grey to 0.606 in the display referred domain. This means that if you load an sRGB texture into Cycles, the middle grey point would be shifted away from the usual sRGB point of 0.466 that imagers expected. To alleviate this mental strain that is easily shifted during grading, the sRGB scaled grey variants were added as "training wheels". This allows an imager to confidently look at the various scaled looks to check on sRGB texture ratio mappings.
 * *Desaturation Basic* This view adds a convergence point towards display referred white. The curve towards white begins at scene referred value 3.0 and converges all primaries towards white as their intensity approaches scene referred 16.291. This emulates a basic filmic / DSLR "blow out" and helps shape all colours to desaturate correctly, as opposed to without any desaturation / convergence / unity point.
 * *Desaturation Contrast 1.6-2.4* This is a range of looks that, in addition to the basic desaturation described above, adds on an additional power curve to add contrast. This is designed to give imagers a basic idea as to what a grade might look like when modified atop of the basic view with desaturation.
 * *Greyscale on Desaturation* This is a greyscale preview of the desaturated look helpful for evaluating contrast and other details in some contexts.
 * *Greyscale* This is a greyscale preview without the desaturation look applied. As above.
 * *False Colour Basic* This is a false colour "heat map" of exposure. The colour scheme is as follows:
   * ```Low Clipping                 = Black    Scene Referred Linear value below 0.0001762728758.```
   * ```Ten Stops Down               = Purple   Scene Referred Linear value 0.0001762728758.```
   * ```Seven Stops Down             = Blue     Scene Referred Linear value 0.001404109349.```
   * ```Four Stops Down              = Cyan     Scene Referred Linear value 0.01124714399.```
   * ```Two Stops Down               = Green    Scene Referred Linear value 0.04456791864.```
   * ```Middle Grey                  = Gray     Scene Referred Linear value 0.18009142.```
   * ```Two Stops Over               = Green    Scene Referred Linear value 0.7196344767.```
   * ```Four Stops Over              = Yellow   Scene Referred Linear value 2.883658483.```
   * ```Five and a half Stops Over   = Red      Scene Referred Linear value 8.150007644.```
   * ```High Clipping                = White    Scene Referred Linear value above 16.29174024.```

**Eeek! Now my render looks awful!**

When you use the Wide Dynamic Range View LUT you may believe that your rendered output looks flat, washed out and desaturated. This is because the view has a logarithmic contrast curve which does not resemble the contrasted look of the default sRGB transform. The goal of this view is to roughly provide a perceptually uniform view of the data to evaluate the density across the full sixteen and a half stop range. With this log view you'll be able to judge your lighting better, and determine whether your scene is properly exposed or not.

It may take some time to develop an eye for the data presented in the basic view. To this end, there are a series of "Contrast" looks that increase the contrast into the more typically expected ranges. These Looks are not applied directly to your data, but rather simply a transformation of your scene referred data into a rough shape that one might aim for when grading.

The false-color looks can be leveraged to get a visual representation of the exposure of your scene and give an immediate representation as to where your image may be getting clipped at the head and the toe in the display referred transform.

The transforms are designed to generate imagery that will be handed off to a grade for look application.

**Help! Grading is tricky!**

The following is a brief list of cautionary bits for imagers while working on scene referred data:
 * Blender desperately needs independent colour transform controls on every UI element. Using this set of LUTs should make it abundently clear.
 * For the past decade or so, Blender's basic mixing node has been broken with regards to certain AdobePDF blend modes, as they are strictly display referred. Screen, Burn, Dodge, Overlay, and a few others are strictly display referred blend modes, and will not work on scene referred data.
 * Lift, Gamma, and Gain is a strictly display referred tool. It will not work with scene referred data.
 * Curves. While curves work fine, without the specific colour transformation selection, it is very tricky. Middle
grey lives at 0.18, which gives you approximately 18% of the UI's interface region to adjust a curve that will
result in tweaking ten stops of your most critical latitude.

**I get it, Blender's broken in fundamental ways. Solutions?**

The ASC CDL transform was designed to operate on both display referred and scene linear imagery. It is found in
the Colour Correction node where *Lift, Gamma, Gain* is listed. Change it to the ASC CDL, adjust Power up to
compensate for the log viewing in the basic view version, and possibly tweak Slope down. Offset will literally
offset values up or down, which will likely be done in very small increments. Adjust your lighting if you can. It should also be noted that the UI representation for the CDL is very limited. Due to operating on scene referred data, you may find that you will have to push Power up beyond the UI interface sliders. This can be acoomplished by using the data entry values, as opposed to the sliders. Also note that operating on the scene referred range may require extremely granular controls when using Slope and Offset for example. The [Shift] key may help here when trying to gradually adjust a value in the numerical inputs.
