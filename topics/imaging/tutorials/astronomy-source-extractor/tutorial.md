---
layout: tutorial_hands_on

title: Astronomy Source extractor
questions:
  - How do I partition an image into regions based on which object they are nearest to (Voronoi segmentation)?
  - How should images be preprocessed before applying Voronoi segmentation?
  - How can I overlay two images?
  - How can Voronoi segmentation be used to analyze spatial relationships?
  - How can extracted image properties be used to categorize identified objects? 
objectives:
  - How to perform Voronoi Segmentation in Galaxy.
  - How to extract a single channel from an image. 
  - How to overlay two images. 
  - How to count objectives in a layer map. 
time_estimation: 1H
key_points:
- Voronoi segmentation is a simple algorithm, chosen to give a starting point for working with image segmentation. 
- This tutorial exemplifies how a Galaxy workflow can be applied to data from several different domains. 
requirements:
  -
    type: "internal"
    topic_name: imaging
    tutorials:
      - imaging-introduction

contributions:
  authorship:
    - Andrei-EPFL
  funding:
    - fiesta
    - oscars
tags:
- imageanalysis
- astronomy

---

In astronomy and large-scale sky surveys, a key objective is to identify individual celestial bodies—such as stars and galaxies—in sky images to enable further detailed scientific analysis. For example, DESI Legacy Survey have "photographed" a third of the sky, whereas DESI is measuring spectra of individual galaxies selected from all the photometrically detected targets.
The Galaxy source-extractor tool is based on SEP which is a Python library built from the core routines of SExtractor (Source Extractor), which is widely used in astronomy for detecting and measuring sources in astronomical images. 

In order to detect sources, the background must be estimated and then all groups of pixels with a minimum area of
```markdown
minarea = 5 (default value of the number of pixels)
```
 with a flux larger than an absolute threshold at a pixel (j,i):
```markdown
thresh * err[j,i]
```
are selected as sources, where 
```markdown
thresh = 1.5
err_option = float_globalrms
```
i.e. by default the error is the global RMS of the background.

In case
```markdown
err_option = none
```
the value of ```thresh``` is use directly as an absolute threshold.

This tutorial will explain the data requirements for the tool, the main tool parameters and the output products of the tool. For more detailed instructions one can consult: https://sep.readthedocs.io/en/v1.0.x/index.html, https://ui.adsabs.harvard.edu/abs/1996A%26AS..117..393B/abstract and https://www.astromatic.net/software/sextractor/ .

### Application to bioimage analysis


### Application to Earth Observation


### Application to Astronomy

![Sky image](../../images/voronoi-segmentation/sky_image_IMAGE.png "Sky image.")

![Segmented sky image](../../images/voronoi-segmentation/sky_image_OVERLAY_VORONOI.png "Segmented sky image.")

Thes original image is published by [Legacy Surveys / D. Lang (Perimeter Institute)](https://www.legacysurvey.org/acknowledgment/) and can be downloaded from the [official website](https://www.legacysurvey.org/viewer/jpeg-cutout?ra=53.16216667&dec=-27.79149167&layer=ls-dr10&pixscale=0.262&size=1200). The Legacy Surveys are described in {% cite legacy-survey-astronomy %}.

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}


## Data requirements 
Source-extractor requires one single image. For astronomy, this is a sky-image with luminous sources. Nonetheless, two other files could provided as well: a mask and a filter. Masking is an important process in astronomy, since very bright stars strongly influence the estimation of the background. Therefore, astronomers create geometric masks to remove the pixels affected by bright stars. 

Lastly, the filter is used to smooth out the image to help improve the detection of faint, extended objects, but may not be helpful very crowded data. The ```Filter Case``` tool parameter sets whether the filter file is needed or not.


**Image:** 
- Preferrably lighter objects on a darker background.
- Format: single channel, 2D array, .tiff or .fits. 

**Mask:** 
- True values, or numeric values greater than ```maskthresh``` tool parameter, are considered masked.
- Format: single channel, 2D array, .tiff or .fits. 

**Filter kernel:** 
- 2D array.
- Format: .txt, readable by
```python
import numpy
numpy.loadtxt(filter_file)
```
The default kernel is:
```markdown
1 2 1
2 4 2
1 2 1
```
and can be stored as shown in a text file.

> <comment-title> Checking the metadata of an image </comment-title>
> Tip: You can use the tool {% tool [Show image info](toolshed.g2.bx.psu.edu/repos/imgteam/image_info/ip_imageinfo/5.7.1+galaxy1) %} to extract metadata from your image. The image metadata should have "SizeC = 3". It should not say "SizeC = 3 (effectively 1)", because that means that your image is stored in interleaved, not planar format.
{: .comment}


## Getting data from Zenodo
> <hands-on-title> Data Upload </hands-on-title>
>
> 1. Create a new history for this tutorial. When you log in for the first time, an empty, unnamed history is created by default. You can simply rename it.
> 
>    {% snippet faqs/galaxy/histories_create_new.md %}
> 
> 2. Import {% icon galaxy-upload %} the following dataset from [Zenodo](https://zenodo.org/records/15172302). 
> 
>    ```
>    https://zenodo.org/records/15281843/files/images_and_seeds.zip
>    https://zenodo.org/records/15424465/files/image_and_seed.zip
>    ```
> 
>    - **Important:** Choose the type of data as `zip`.
> 
>    The upload might take a few minutes. 
> 
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
> 3. {% tool [Unzip](toolshed.g2.bx.psu.edu/repos/imgteam/unzip/unzip/6.0+galaxy0) %} the image with the following parameters:
>    - {% icon param-file %} *"input_file"*: `images_and_seeds.zip`
>    - *"Extract single file"*: `Single file`
>    - *"Filepath"*: Choose which data you want to use: 
>        - Cells: `images_and_seeds/cell_image-B2--W00026--P00001--Z00000--T00000--dapi.tiff`
>        - Trees: `images_and_seeds/tree_image_2019_DELA_5_423000_3601000.tiff`
>        - Galaxies: `sky_image_IMAGE.png`
>    
> 4. Rename {% icon galaxy-pencil %} the resulting file as `image`.
>
> 5. Check that the datatype is correct.
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
> 6. {% tool [Unzip](toolshed.g2.bx.psu.edu/repos/imgteam/unzip/unzip/6.0+galaxy0) %} the seed with the following parameters:
>    - {% icon param-file %} *"input_file"*: `images_and_seeds.zip`
>    - *"Extract single file"*: `Single file`
>    - *"Filepath"*: Choose the seed image corresponding to the image you chose in the last step. 
>        - Cells: `images_and_seeds/cell_seeds-B2--W00026--P00001--Z00000--T00000--dapi.tiff`
>        - Trees: `images_and_seeds/tree_seeds_2019_DELA_5_423000_3601000.tiff`
>        - Galaxies: `sky_image_SEED.png`
>
> 7. Rename {% icon galaxy-pencil %} the resulting file as `seeds`.
{: .hands_on}


## Generate an object mask from pixel intensity
In case there should be empty regions without cells in the image, we wish to constrain the single Voronoi regions to roughly the area where a cell is. 
Therefore, we first smooth the image to reduce the influence of noise, and then apply a threshold on the smoothed image to get a binary mask. 
> <comment-title> Mask vs seeds </comment-title>
> The process used to create a mask can also be used to make seeds, as in the 
> [Imaging introduction tutorial]({% link topics/imaging/tutorials/imaging-introduction/tutorial.md %}). 
{: .comment}

> <hands-on-title> Task description </hands-on-title>
> The image has three channels (red, green, blue). To generate a mask, we have to select a channel, for instance channel `0`. 
> 1. {% tool [Convert image format](toolshed.g2.bx.psu.edu/repos/imgteam/bfconvert/ip_convertimage/6.7.0+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `image`
>    - *"Extract series"*: `All series`
>    - *"Extract timepoint"*: `All timepoints`
>    - *"Extract channel"*: `Extract channel`
>        - *"Channel id"* `0`
>    - *"Extract z-slice"*: `All z-slices`
>    - *"Extract range"*: `All images`
>    - *"Extract crop"*: `Full image`
>    - *"Tile image"*: `No tiling`
>    - *"Pyramid image"*: `No Pyramid`
> 
> 2. Rename the output to `single channel image`
> 
> 
> 3. {% tool [Filter 2-D image](toolshed.g2.bx.psu.edu/repos/imgteam/2d_simple_filter/ip_filter_standard/1.12.0+galaxy1) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `single channel image`
>    - *"Filter type"*: `Gaussian`
>        - *"Sigma"*: `3`
> 
> 4. Rename the output to `smoothed image`.
> 
>
> 5. {% tool [Threshold image](toolshed.g2.bx.psu.edu/repos/imgteam/2d_auto_threshold/ip_threshold/0.18.1+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `smoothed image`
>    - *"Thresholding method"*: `Manual`
>        - *"Threshold value"*: `3.0`. Note: This threshold value works well for the cell image, where the background has a very low intensity. For images with a brighter background, the threshold value will need to be adjusted.
> 
> 6. Rename the output to `mask`.
> 
{: .hands_on}

> <comment-title> How many channels does my image have? </comment-title>
> Note: If providing your own image, you can check how many channels your image has with the {% tool [Show image info](toolshed.g2.bx.psu.edu/repos/imgteam/image_info/ip_imageinfo/5.7.1+galaxy1) %} tool.
> The number of channels is listed as, e.g., `SizeC = 3` for the cell image or `SizeC = 3 (effectively 1)` for the tree image.
{: .comment}

> <comment-title> The value of "Sigma" and the "Threshold value" </comment-title>
> Note: Generating a robust mask is harder for images with more noise. 
> Since the tree image has more noise than the cell image, you may have to adjust the value of *"Sigma"* to achieve better results.
> You may also have to adjust the *"Threshold value"* in the last step.
{: .comment}


> <question-title></question-title>
>
> What is the purpose of the smoothing step? 
>
> > <solution-title></solution-title>
> >
> > The purpose of smoothing is to reduce noise. 
> > For seed generation, noise might lead to false seeds where there is no object. 
> > Smoothing also promotes connectedness within an object, where noise might make an object appear as two separate objects. 
> > For mask generation, the same principles apply.
> >
> {: .solution}
>
{: .question}


## Perform Voronoi segmentation based on the seeds 
> <hands-on-title> Task description </hands-on-title>
> We need to assign a unique label to each object in the seed image:
> 1. {% tool [Convert binary image to label map](toolshed.g2.bx.psu.edu/repos/imgteam/binary2labelimage/ip_binary_to_labelimage/0.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Binary Image"*: `seeds`
>    - *"Mode"*: `Connected component analysis`
> 
> 2. Rename the output to `label map`.
>
> 1. {% tool [Compute Voronoi tessellation](toolshed.g2.bx.psu.edu/repos/imgteam/voronoi_tesselation/voronoi_tessellation/0.22.0+galaxy3) %}. We use the label map to perform Voronoi segmentation. 
>    - {% icon param-file %} *"Input Image"*: `label map`
> 
> 2. Rename the output to `tessellation`.
>
{: .hands_on}

> <question-title></question-title>
>
> How does the size of the seeds influence the Voronoi segmentation? 
>
> > <solution-title></solution-title>
> >
> > A Voronoi segmentation partition a plane into regions based on [proximity to each member of a given set of objects](https://en.wikipedia.org/wiki/Voronoi_diagram). The algorithm is the same irregardless of whether the seeds are single points or objects with a spatial extent, but the size of the seeds will certainly alter the segmentation in some way. 
> >
> {: .solution}
>
{: .question}

## Apply the mask and visualize the segmentation
A Voronoi tessellation segments an image into non-overlapping segments that cover the entire image. 
This makes sure that segments do not overlap, but empty spaces between objects will be counted as part of the segment belonging to the nearest item. 
A more accurate segmentation can be achieved by using the mask to reduce the size of the Voronoi segments. 
This can be achieved with the following operation. 
> <hands-on-title> Task description </hands-on-title>
> Combine the tessellation with the seeds and the mask to generate a segmentation that limits the expanse of each segment: 
> 1. {% tool [Process images using arithmetic expressions](toolshed.g2.bx.psu.edu/repos/imgteam/image_math/image_math/1.26.4+galaxy2) %} with the following parameters:
>    - *"Expression"*: `tessellation * (mask / 255) * (1 - seeds / 255)`
>    - In *"Input images"*:
>        - {% icon param-repeat %} *"Insert Input images"*
>            - {% icon param-file %} *"Input Image"*: `tessellation`
>            - *"Variable for representation of the image within the expression"*: `tessellation`
>        - {% icon param-repeat %} *"Insert Input images"*
>            - {% icon param-file %} *"Input Image"*: `seeds`
>            - *"Variable for representation of the image within the expression"*: `seeds`
>        - {% icon param-repeat %} *"Insert Input images"*
>            - {% icon param-file %} *"Input Image"*: `mask`
>            - *"Variable for representation of the image within the expression"*: `mask`
>
> 2. Rename the output to `masked segmentation`.
>
> 3. {% tool [Colorize label map](toolshed.g2.bx.psu.edu/repos/imgteam/colorize_labels/colorize_labels/3.2.1+galaxy3) %}. 
>    - {% icon param-file %} *"Input Image"*: `masked segmentation`
>    - *"Radius of the neighborhood"*: `10` (works well for the cell image; may need adjustment for the tree image)
>    - *"Background label"*: `0`
> 
> 4. Rename the output to `colorized label map`
>
> 5. {% tool [Convert single-channel to multi-channel image](toolshed.g2.bx.psu.edu/repos/imgteam/repeat_channels/repeat_channels/1.26.4+galaxy0) %} with: 
>    - {% icon param-file %} *"Input Image"*: `single channel image`
>    - *"Number of channels"*: `3`
> 
> 6. Rename the output to `multi channel image`
> 
> 7. {% tool [Overlay images](toolshed.g2.bx.psu.edu/repos/imgteam/overlay_images/ip_overlay_images/0.0.4+galaxy4) %} with the following parameters to overlay the Voronoi segmentation on the original image:  
>    - *"Type of the overlay"*: `Linear blending`
>    - {% icon param-file %} *"Image 1"*: `multi channel image`
>    - {% icon param-file %} *"Image 2"*: `colorized label map`
>    - *"Weight for blending"*: `0.5`
>
{: .hands_on}


## Count objects and extract image features
> <hands-on-title> Task description </hands-on-title>
> 1. {% tool [Count objects in label map](toolshed.g2.bx.psu.edu/repos/imgteam/count_objects/ip_count_objects/0.0.5-2) %}. 
>    - {% icon param-file %} *"Input Image"*: `tessellation`
>
> 1. {% tool [Extract image features](toolshed.g2.bx.psu.edu/repos/imgteam/2d_feature_extraction/ip_2d_feature_extraction/0.18.1+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Label map"*: `tessellation`
>    - *"Use the intensity image to compute additional features"*: `Use intensity image`
>        - {% icon param-file %} *"Intensity Image"*: `single channel image`
>    - *"Select features to compute"*: `Select features`
>    - *"Available features"*:
>        - {% icon param-check %} `Label from the label map`
>        - {% icon param-check %} `Max Intensity`
>        - {% icon param-check %} `Mean Intensity`
>        - {% icon param-check %} `Minimum Intensity`
>        - {% icon param-check %} `Area`
>        - {% icon param-check %} `Major axis length`
>        - {% icon param-check %} `Minor axis length`
>
{: .hands_on}


In this last step, we compute the max, min and mean intensity for each image segment, as well as the area and the major and minor axis lengths. 
Depending on the use case, the distribution of these extracted features could reveal different subgroups in the data. 
In this way, the features could be used in to categorize different types of trees, cells, or other items. 
We will now use a scatter plot to explore the data. 


## Visualize segment features with a scatter plot 
> <hands-on-title>Plot feature extraction results</hands-on-title>
> 1. Click on the **Visualize** {% icon galaxy-barchart %} icon of the {% icon tool %} **Extract image features** output.
> 2. Run **Scatter plot (NVD3)** with the following parameters:
>    - *"Provide a title"*: `Segment features`
>    - *"X-Axis label"*: `Major axis length`
>    - *"Y-Axis label"*: `Minor axis length`
>    - *"Column of data point labels"*: `Column 1`
>    - *"Values for x-axis"*: `Column 6`
>    - *"Values for y-axis"*: `Column 7`
>
>    > <question-title></question-title>
>    >
>    > Plot the major axis lengths together with the minor axis lengths. What do you observe?
>    >
>    > > <solution-title></solution-title>
>    > > The major and minor axis lengths appear to be positively correlated, which is not surprising. It means that segments with a major axis that is longer than others typically have a longer minor axis too. 
>    > {: .solution }
>    {: .question}
{: .hands_on}


# Conclusion
This pipeline performs Voronoi segmentation and can be applied to datasets from any field as long as the input data satisfies the input data criteria. 
The steps in this tutorial are also available on Galaxy as a published workflow called [Voronoi segmentation](https://usegalaxy.eu/published/workflow?id=23030421cd9fcfb2). 
