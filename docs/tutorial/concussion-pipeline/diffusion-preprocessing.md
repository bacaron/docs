!!! warning
    This is a draft

## Diffusion-weighted MRI preprocessing.

This page demonstrates common steps used to preprocess diffusion magnetic resonance imaging (dMRI) data on brainlife.io. The goal of this tutorial is to show you how to process diffusion data for successive analyses, including **microstructural region analysis**, **diffusion tractography**, and **structural connectivity**. This tutorial will be mostly be using [MrTrix3 Preproc](https://brainlife.io/app/5a813e52dc4031003b8b36f9) for diffusion preprocessing, [FSL BET](https://brainlife.io/app/5c63a9c3f2362b00318046c2) for brainmask generation, [FSL DTIFit](https://brainlife.io/app/5c1d67383e9c170177733d3f) and [NODDI AMICO](https://brainlife.io/app/5bc7ac1244f3980027a7aafd) for microstructral modelling.

This tutorial will use a combination of skills developed in the [Introduction tutorial](https://brainlife.io/docs/tutorial/introduction-to-brainlife/) and the [Anatomical Preprocessing tutorial](https://brainlife.io/docs/tutorial/t1w-preprocessing/) you recently completed. If you haven't read our introduction to brainlife, or if you're not comfortable staging, processing, archiving, and viewing data on brainlife.io, please go back through that tutorial before beginning this one.

### 1. Anatomical preprocessing.

The first step of diffusion preprocessing often involves processing the anatomical images. In order to guarantee that any generalizations regarding location made from the preprocessed diffusion data is anatomically-informed, we must have both of our anatomical (T1w or T2w) images and our diffusion MRI images **aligned**. One way we can make this easier for [MrTrix3 Preprocessing](https://brainlife.io/app/5a813e52dc4031003b8b36f9) is by aligning the anatomical images in such a way that the center of the brain is centered in the image. We refer to this as **ACPC-aligned**, as we are aligning the data to the **anterior commissure-posterior comissure plane**. This is the first step in dMRI preprocessing, and it is typically done with the [HCP ACPC Alignment (T1w)](https://brainlife.io/app/5c61c69f14027a01b14adcb3) app. However, we accomplished this in the 'Anatomical preprocessing' tutorial by using the [FSL Anat](https://brainlife.io/app/5e3c87ae9362b7166cf9c7f4). Once we've centered our anatomical image, we can move onto diffusion MRI preprocessing.

### 2. Diffusion preprocessing 

The next step in diffusion preprocessing is to actually preprocess the diffusion data. dMRI data, like any other MRI dataset, is quite noisy. Just like fMRI, dMRI data is sensitive to **susceptibility distortions**. This is when regions of the brain look "wavy" or fold inward or outward on itself in images. This is due to how the scanner sweeps across the entire brain during acquisition. Also, dMRI data is very susceptible to motion. Because some scans take a long time, it is nearly impossible for a participant to lie completely motionless. Even the most subtle movements can alter the results in a specific region and make your data less accurate. Finally, movement of the magnetic gradients can induce electromagnetic currents which can affect the measurement in any given location. These electromagnetic currents are called **eddy currents**, and they are due to the fact that electricity and magnetism are part of the same field, otherwise known as the **electromagnetic field**. Thankfully, brainlife.io allows us to correct for all of these common artifacts using [MrTrix3 Preprocessing](https://brainlife.io/app/5a813e52dc4031003b8b36f9).

In addition to these artifacts, dMRI has a very low **signal-to-noise ratio (SNR)**, which directly impacts how well you can fit microstructural models and analyze diffusion data. This means that in any given location, the amount of diffusion signal is very low compared to the "noise" of the scanner. There are methods to lower the amount of noise across the brain and thus increase the SNR. [MrTrix3 Preprocessing](https://brainlife.io/app/5a813e52dc4031003b8b36f9) has the capability to identify and remove noise using principal components analysis (PCA) and Rician analysis.

On top of these issues, dMRI data is sensitive to other artifacts and non-regularities, including **gibbs ringing and bias field inhomogeneities**, which can affect the quality of the data and how well measures can be derived from the data. [MrTrix3 Preprocessing](https://brainlife.io/app/5a813e52dc4031003b8b36f9) can deal with all of these issues for us!

Finally, once the dMRI data is preprocessed and cleaned, [MrTrix3 Preprocessing](https://brainlife.io/app/5a813e52dc4031003b8b36f9) will align the dMRI data to the anatomical data. This will ensure that any analyses we do with the dMRI data will be anatomically-informed and biologically-relevant.

Useful information about the preprocessing pipeline that [MrTrix3 Preprocessing](https://brainlife.io/app/5a813e52dc4031003b8b36f9) is designed to run -- Diffusion Parameter EStimation with Gibbs and NoisE Removal (DESIGNER) -- can be found in this [original Neuroimage paper](https://www.ncbi.nlm.nih.gov/pubmed/30077743). **(<---- fixed link, double-check this is the correct pub)**

### 3. Diffusion microstructural modelling

Once the dMRI data has been cleaned and aligned to the anatomical (T1w) image, the next step is to start fitting diffusion-based models to the data. The first, and most widely-reported model, is the Diffusion Tensor (DTI) model, which attempts to model how much and in what direction water moves in the brain using a **tensor**. Regions in which water is moving **anisotropically**, or in the same direction, will have a tensor that is very cigar-shaped. Regions in which water is moving **isotropically**, or equally in all directions, will have a spherical-shaped tensor. This model outputs four different measurements: **axial diffusivity** (how strong water movement is in the primary direction of movement); **radial diffusivity** (how strong water movement is in the non-primary directions of movement); **mean diffusivity** (how strong water movement is in any direction); and **fractional anisotropy** (how strong water movement is and how directional that movement is). Fractional anisotropy (FA) and mean diffusivity (MD) are the most widely reported measures. These measures can be used for group anaylses and in tractography/tractometry pipelines, along with cortical white matter mapping pipelines. For this tutorial, we will use [FSL DTIFit](https://brainlife.io/app/5c1d67383e9c170177733d3f) to fit the DTI model to our preprocessed data.

More sophisticated models take advantage of a feature of new acquisition protocols to distinguish the tissue response of different tissue sources, including axons, dendrites, and glial cells. This property is multiple b-value (i.e. gradient strength), also known as **multi-shell**, acquisitions. These types of acquistions use multiple strengths of diffusion gradients to measure the differential properties of tissues under magnetic gradients. One such model that does this to measure the density and orientation of neurites is the Neurite Orientation Dispersion Density Imaging (NODDI) model. This model separates the diffusion signal into multiple compartments that correspond to the different types of diffusion signals given by different tissue types. The three main outputs from the NODDI model are neurite density index (i.e. NDI, ICVF), orientation dispersion (i.e. OD, ODI), and an isotropic volume fraction (ISOVF). NDI is a measure of the density of axons and dendrites among many other biological properties, including tau protein density. ODI is a measure of how much "spreading" or "fanning" there is between neurites and can be useful for identifying tissue damage from repetitive head impacts. ISOVF is a measure of the CSF really, and can be used as a potential index of neuroinflammation. To fit this model, we will use [NODDI AMICO](https://brainlife.io/app/5bc7ac1244f3980027a7aafd).

Now, let's get to work! The following steps of this tutorial will show you how to:

1. ACPC align the anatomical (T1w) image (already done), 
2. preprocess the dMRI data using MrTrix3 Preprocessing,
3. Generate a brainmask of the preprocessed dMRI image for DTI and NODDI fitting
3. and fit the diffusion tensor (DTI) and NODDI models to the preprocessed dMRI data.

### ACPC-align anatomical (T1w) image.

If you've completed the anatomical tutorial, you do not need to re-align the anatomical image. Move onto the next step: preprocessing the dMRI image!


### Preprocess diffusion MRI data with mrtrix3 preproc.

1. On the 'Pipelines' tab, click 'Create New Button (+)' to submit a new pipeline.
    * In the search bar, type 'mrtrix3 preproc'
    * Click the app card.
1. In the 'Configuration' section, select the following:
    * Select the boxes for 'denoise', 'degibbs', 'eddy', 'bias', 'ricn' and 'acpc'. This will denoise, removal the gibbs ringing artifact, perform topup and eddy correction, perform bias and rician background noise removal, and align the DWI to the acpc_aligned T1.
    * For 'reslice', type '1'. This will reslice the data to 1mm isotropic resolution.
    * Leave rest as defaults.
1. 1. In the 'Inputs' section, do the following:
    * Check the '+' sign next to the 'anat/t1w' input and type "acpc_aligned". This will filter the datasets to only the "acpc_algined" data generated in the previous tutorial.
    * For the first 'dwi' datatype, choose the 'AP' tagged dMRI image. 
    * For the second, first deselec  the box indicating not to use the second dwi input and choose the dMRI image with the 'PA' tag. These are important for topup and eddy to correct for acquistion-related artifacts.
1. In the outputs section, select the boxes for the 'dwi: preprocessed'.
    * In the 'Dataset Tags' field, type and enter "fsl_anat" and "1mm reslice" to denote that this was ran aligned to an image from fsl_anat and that it was resliced to 1mm isotropic
1. Hit 'Submit'
1. Click the button next to 'Offline' on the Pipeline card to launch the pipeline. You'll see the Pipeline tab update with blue and numbers indicating the jobs launched. When jobs finish, you'll start noticing green on the progress bar of the Pipeline card.
1. Once the app is finished running, view the results by clicking the 'eye' icon to the right of the dataset
    * Choose 'fsleyes' as your viewer
    * Only have the file titled 'dwi.nii.gz' selected in the viewer

Once you're happy with the results, you can move onto making the brainmask for fitting the DTI and NODDI models!

### Create a brainmask of your newly preprocessed dMRI data

1. On the 'Pipelines' tab, click 'Create New Button (+)' to submit a new pipeline.
    * In the search bar, type 'fsl brain extraction' and choose the one corresponding to DWI.
    * Click the app card.
1. In the 'Configuration' section, select the following:
    * For 'fthreshold', type and enter 0.2. This will generate a very liberal brainmask
    * Leave the rest as default.
1. In the 'Inputs' section, do the following:
    * Check the '+' sign next to the 'dwi' input and type "preprocessed". 
    * In the "Dataset Tags" field, type and enter "1mm reslice" and "fsl_anat". This will filter the datasets to only the preprocessed dMRI image generated above.
1. In the outputs section, deselect the box for the 'dwi: brain_extracted' and select the box for "mask: dwi"
    * In the 'Dataset Tags' field, type and enter "fsl_anat" and "1mm reslice" to denote that this was generated from the preprocessd dMRI image.
1. Hit 'Submit'
1. Click the button next to 'Offline' on the Pipeline card to launch the pipeline. You'll see the Pipeline tab update with blue and numbers indicating the jobs launched. When jobs finish, you'll start noticing green on the progress bar of the Pipeline card.

Once that's done, you can move onto fitting the diffusion tensor (DTI) model!

### Fit the DTI model to the preprocessed dMRI data.

1. On the 'Pipelines' tab, click 'Create New Button (+)' to submit a new pipeline.
    * In the search bar, type 'fsl dtifit' and choose the new one (not old).
    * Click the app card.
1. In the 'Configuration' section, select the following:
    * Leave everything as defaults. We want to use a single shell for the tensor fit as multi-shell fits might lead to issues in the tensor fit.
1. In the 'Inputs' section, do the following:
    * Check the '+' sign next to the 'dwi' input and type "preprocessed". 
    * In the "Dataset Tags" field, type and enter "1mm reslice" and "fsl_anat". This will filter the datasets to only the preprocessed dMRI image generated above.
    * Deselect the box for 'mask' indicating that the datatype isn't being used. Click the '+' sign nex to the name and type and enter "dwi". This will filter to the brainmask generated above.
1. In the outputs section, add the tag "wm" and "fsl_anat" and "1mm reslice" to indicate that this tensor is to be used in white matter tracking (we will discuss later) and that it was from the dMRI image that was aligned to the output from fsl-anat and that was resliced to 1mm isotropic.
1. Once a subject is finished running, view the results by clicking the 'eye' icon to the right of the dataset
    * Choose 'fsleyes' as your viewer

Once you're finished setting up the pipeline, you can move onto the NODDI mapping!

### Fit the NODDI model to the preprocessed dMRI data.

1. On the 'Pipelines' tab, click 'Create New Button (+)' to submit a new pipeline.
    * In the search bar, type 'noddi amico' and choose the 'non_dtiinit' version.
    * Click the app card.
1. In the 'Configuration' section, select the following:
    * Leave everything as defaults.
1. In the 'Inputs' section, do the following:
    * Check the '+' sign next to the 'dwi' input and type "preprocessed". 
    * In the "Dataset Tags" field, type and enter "1mm reslice" and "fsl_anat". This will filter the datasets to only the preprocessed dMRI image generated above.
    * Deselect the box for 'mask' indicating that the datatype isn't being used. Click the '+' sign nex to the name and type and enter "dwi". This will filter to the brainmask generated above.
1. In the outputs section, add the tag "wm" and "fsl_anat" and "1mm reslice" to indicate that this NODDI is to be used in white matter tracking (we will discuss later) and that it was from the dMRI image that was aligned to the output from fsl-anat and that was resliced to 1mm isotropic. Also, deselect the mask output from being archived as we are inputting our own mask and don't need to output a new one.
1. Once a subject is finished running, view the results by clicking the 'eye' icon to the right of the dataset
    * Choose 'fsleyes' as your viewer

**If you're happy with the results, then you have successfully finished preprocessing your dMRI data! You're now ready to move onto the next tutorial: diffusion tractography!**
