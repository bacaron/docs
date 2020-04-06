!!! warning
    This is a draft

## Anatomical (T1w) preprocessing.

This page demonstrate common steps used to preprocess anatomical magnetic resonance imaging data (T1-weighted or T1w) on brainlife.io The goal of this turorial is to show how process anatomical data for successive analyses – volumetric analyses from T1w meausres, combination of T1w and diffusion-weighted MRI (dMRI) or functional neuroimaging data (fMRI) pipelines.

This tutorial will use a combination of skills developed in the introduction-to-brainlife tutorial (https://brainlife.io/docs/tutorial/introduction-to-brainlife/). If you have not read this, or you are not comfortable staging, processing, archiving and viewing data on brainlife.io, please go back and follow that tutorial before beginning this one.

### 1. Set the orientation of the brain.

The first step is to make sure the anatomy of the brain 'looks right.' This means that the coordinate system for the T1w image is stored rotated such that the brain appears up-right and centered when opened in the major visualization toolboxes that the scientific community uses (i.e., generally we look at images where the nose faces towards right-hand side in the sagittal plane, and forward in the axial and coronal planes). This rotation involves changing information in the T1w data files and headers. Beside agreeing to the standard convetions, this orientation is important because it assures that the data inside the T1w iamges will be "indexed,"  or read, appropirately allowing different software libraries for neuroimaging data analysis to  read the data, interpret the position and dimensions of individual three-dimensional pixels (i.e., voxels). During this process oftentimes the neck is removed from the original MRI data, so to fit the image within an adequate volume.

Useful information about MRI image orientation can be found at: 
  - https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Orientation%20Explained 
  - http://gru.stanford.edu/doku.php/mrTools/coordinateTransforms

### 2. Anterior Commissure - Posterior Comissure (ACPC) alignment.

The next step is to align the anatomical image to the point where two white matter tracts cross hemispheres: the anterior commissure and posterior commissure (ACPC). The anterior commissure connects the two temporal lobes and is located below the anterior portion of the corpus callosum (i.e. fornix), the large white matter tract connecting the two hemispheres. The posterior commissure is located just behind the cerebral aqueduct, an important connection between the third and fourth ventricles. This step will move the image in a way that effectively "centers" the image. Specifically, the image will be moved so that a horizontal line drawn from the front to the back of the brain will pass through both landmarks, and a horizontal line drawn from the temporal lobes and a vertical line from the top of the brain to the bottom bisect at the midpoint of the two landmarks. A majority of the following processing steps require the anatomical image to be aligned to the ACPC plane, including Freesurfer parcellation.

### 3. Freesurfer Brain Parcellation - Generation.

The next step is perform anatomical preprocessing and divide (i.e. parcellate) the brain into different regions and tissue types (i.e. white matter, gray matter) using Freesurfer. These regions represent different gray matter (i.e. cortical) landmarks such as the motor and somatosensory cortices, and are typically derived from histological properties of these regions. Sometimes, they are derived from functional activation (i.e. fMRI) patterns. Freesurfer will divide the anatomical image into different tissue types (i.e. white matter, gray matter), parcellate the gray matter into known anatomical landmarks, and output statistics regarding the volume and thickness of the different tissue types and anatomical landmarks. These regions are then used for a large number of downstream steps, including white matter tract segmentation. Information regarding the volume and thickness of each region can also be used for group analyses.

Now, let's get to work! The following steps of this tutorial will show you how to:

1. Crop and reorient the anatomical (T1w) image,
1. ACPC align the anatomical (T1w) image, 
1. generate cortical and white matter surfaces and parcellations using Freesurfer,

### Crop and reorient anatomical (T1w) image, & ACPC align the T1w image using FSL-Anat

Helpful information regarding FSL-Anat can be found here: https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/fsl_anat.

1. On the 'Pipelines' tab, click 'Create New Button (plus mark)' to submit a new pipeline.
    * In the search bar, type 'FSL Anat (T1)'
    * Click the app card.
1. In the 'Configuration' section, select the following:
    * Select the boxes for 'reorient' and 'crop'. Leave all other boxes empty. This will reorient and crop the T1 image.
    * For 'template', choose 'MNI152_0.7mm (MNI152_0.7mm)'. This will align the T1 to the MNI152 0.7mm Template. This will be important for NODDI cortex mapping in the diffusion processing tutorials.
1. Leave the input section as is. It should grab all the data from inside your project. When you start working on more complicated datasets, we can go over what to do in regards to dataset tags.
1. In the outputs section, select the boxes for the 'acpc_aligned', 'standard', and 'warp' datatypes. This will automatically archive these outputs to your 'Archives' tab.
    * In the 'Dataset Tags' field, type and enter "fsl_anat" to denote that this was ran with fsl_anat. It will help with future pipelines, as we can index these tags and select only the datasets we want.
    * Select the box for 'Archive all output datasets when finished'
1. Hit 'Submit'
1. Click the button next to 'Offline' on the Pipeline card to launch the pipeline. You'll see the Pipeline tab update with blue and numbers indicating the jobs launched. When jobs finish, you'll start noticing green on the progress bar of the Pipeline card.
1. Once the subject is finished running, you can view the results by going to the 'Processes' tab of that subject and clicking the 'eye' icon to the right of the dataset
    * Choose 'fsleyes' as your viewer

You can also generate QA-related images using fsl's slicer functionality within this app: https://brainlife.io/app/5e88c72d952fefe0a07abfb6.

Once you're happy with the alignment, you can move onto Freesurfer parcellation generation!

### Freesurfer Brain Parcellation - Generation.

1. On the 'Pipelines' tab, click 'Create New Button (plus mark)' to submit a new pipeline.
    * In the search bar, type 'Freesurfer'
    * Click the app card.
1. In the 'Configuration' section, select the following:
    * Select the boxes for 'hippocampal' and 'hires'. Leave all other boxes empty. This will generate a hippocampal segmentation and generate the surfaces using submillimeter resolution of the T1. These will be important later.
1. In the 'Inputs' section, do the following:
    * Check the '+' sign next to the 'anat/t1w' input and type "acpc_aligned". This will filter the datasets to only the "acpc_algined" data generated in the previous step.
    * Leave the box checked stating not to use the anat/t2 datatype
1. In the outputs section, select the boxes for the 'freesurfer'. Deselect the boxes for the two 'parcellation/volume' datatypes. These are redundant and not needed. This will automatically archive these outputs to your 'Archives' tab.
    * In the 'Dataset Tags' field, type and enter "fsl_anat" to denote that this was ran with fsl_anat. It will help with future pipelines, as we can index these tags and select only the datasets we want.
1. Hit 'Submit'.
1. Click the button next to 'Offline' on the Pipeline card to launch the pipeline. You'll see the Pipeline tab update with blue and numbers indicating the jobs launched. When jobs finish, you'll start noticing green on the progress bar of the Pipeline card.
1. Once the subject is finished running, you can view the results by going to the 'Processes' tab of that subject and clicking the 'eye' icon to the right of the dataset
    * Choose 'freeview' as your viewer
        * This will load the following volumes and surfaces: aseg, brainmask, white matter mask, T1, left/right hemisphere pial (cortical), and white (white matter) surfaces.
    * To view the aparc.a2009s segmentation on an inflated surface, do the following:
        * Click File --> Load surface
            * Choose the lh.inflated and rh.inflated surfaces
            * Hit 'OK'
        * Select inflated surface of choice (i.e. left or right hemisphere)
        * Click the drop-down menu next to 'Annotation' and choose 'Load from file'
            * Choose the appropriate hemisphere aparc.a2009s.annot file (lh.aparc.a2009s.annot)
            * Hit 'OK'
            * The aparc.a2009s parcellation should be overlayed on your inflated surface! Now, repeat the process on the other hemisphere.
    
**You have now finished preprocessing your anatomical (T1w) image! You are now ready to move onto the next tutorial: diffusion MRI (dMRI) preprocessing!**
