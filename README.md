# Garrett-Epp-Kleemeyer-Lindenberger-Polk-2020-NeuroImage


* This folder contains the scripts used for the different analysis steps reported in:

	* [Garrett, Epp, Kleemeyer, Lindenberger, & Polk (NeuroImage 2020). Higher performers upregulate brain signal variability in response to more feature-rich visual input](https://www.sciencedirect.com/science/article/pii/S1053811920303232)

* Raw data files for each participant should be stored in the subfolder 'RAW' in A_preproc.

* All names of folders and scripts are prefixed with letters and sometimes numbers. To replicate the results of the paper, scripts should be run in alphabetical and numerical order. All scripts can be found in the scripts folder of the respective analysis step. Results are stored separately in different subfolders. Before running the scripts, the user must specify the path of the root directory  and the Subject ID List in the config file of the respective scripts folders. Results reported in the paper are based on the default ID list specified in these config files.

* If the scripts should be run on different data, please make sure to alter/adapt the names and paths in all of the scripts provided. 

* All standard images and masks should be stored in the folder G_standards_masks. These include:

    | Image/Source | Details |
    | ------ | ------ |
    | [T1 gray matter mask](https://fsl.fmrib.ox.ac.uk/fsldownloads_registration) | See note below |
    | Gray matter common coordinates mask | generated by the script C_GMcommonCoords.m, located in the B_PLS folder |

**NOTE:** avg152_T1_gray_mask_90_2mm.nii was created using the following inputs and parameters, using the base image from the FSL standard/tissuepriors folder:

>> fslmaths avg152T1_gray.img -thrp 45 -bin avg152_T1_gray_mask_90.nii

* All required toolboxes should be stored in the folder E_toolboxes. Due to space limitations, these should be downloaded from the specified websites. Toolbox versions of the original analysis include:

    * [Nifti toolbox](https://de.mathworks.com/matlabcentral/fileexchange/8797-tools-for-nifti-and-analyze-image) 
    * [PLS toolbox](https://www.rotman-baycrest.on.ca/index.php?section=84) 
    * [spm 12](http://www.fil.ion.ucl.ac.uk/spm/)
    * [HMAX Matlab] (https://maxlab.neuro.georgetown.edu/docs/hmax/hmaxMatlab.tar)
	* preprocessing tools - custom script
---
---
**The following sections contain further information regarding scripts for specific analysis steps.**

## Preprocessing

0. **preproc_bash_config.sh & preproc_mat_config.m**

 Set up base path for project folder and Subject List in these files. ProjectPath variable must be set by the user to location of the data folder. Both the mat file and the bash file must be altered. Subject list corresponds to subject IDs. 

1. **A_BET.sh**

 This script will perform brain extraction of anatomical images. Individual parameters are provided in separate file contained in scripts folder.
 
 Input: /FaceHouseData/A_preproc/RAW/<ID>/anat/mprage.nii.gz
 
 Output: /FaceHouseData/A_preproc/data/<ID>/anat/mprage_brain.nii.gz

2. **B_0_design_template.sh**
 
 B_0_design_template.fsf is a template design file used for FEAT. The B_feat_fsl.sh script performs desired preprocessing procedures, such as smoothing and motion correction.
 
 Subject Design File: /FaceHouseData/A_preproc/data/<ID>/fMRI/design_<ID>.fsf
 
 Input: /FaceHouseData/A_preproc/RAW/<ID>/func_nifti.nii.g
 
 Output: /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>.feat

3. **C_BPfilt_detrend.m**
 
 This script performs quadratic detrending (spm_detrend) and bandpass-filtering, using a 8th orderbutterworth filter.
 
 Input: /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>.feat/filtered_func_data.nii.gz
 
 Output: /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt.nii.gz
 
4. **D_ICA.sh**

 This script performs ICA decomposition. 

 Input: /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt.nii.gz
 
 Output: /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt.ica
 
5. **E_FIX_denoising**

 In this folder, all components for performing FIX denoising are stored

 Note: You should run FIX on your own trainingset, i.e. creating a trainingset of your own data (approx. 1/3 of your data). As an example, we provide the component set of one subject (E_FIX_denoising/example_ica/102_ica.ica/*) as an html page together with our decisions about noise components: E_FIX_denoising/example_ica/102_rejcomps.txt

6. **E_0_create_rejcomp_txtfiles**
    
    1/3 of the dataset's ICA components are screened **manually** and a list of noise components is made for later removal. Noise components must be stored in a subject specific text file. Use this template to fill in the artefactual components and execute the template. This creates txt-files (e.g. SUB01_rejcomps.txt) in E_FIX/rejcomps folder. 
 
    **NOTE**: As ICA calculation is probabilistic, the components may change slightly after each run of FSL MELODIC. Therefore, we provide our original ICA results (components' html sites) under /FaceHouseData/A_preproc/D_ICA_results/ together with our denoising decicions: /FaceHouseData/A_preproc/E_FIX_denoising/rejcomps/. 

7. **E_1_Run_FIX**
    
    This script creates an ouput Training dataset (TrainingData.RData object) from the hand-labeled training data set (E_FIX/rejcomps txt files). It then applies artefact cleanup with the help of this training set for the rest of the study subjects.
    Input:  
    1. Feat dirs for feature extraction (/FaceHouseData/A_preproc/data/<ID>/fMRI/$<ID>.feat)
    2. rejcomps text files under /E_FIX_denoising/rejcomps
    3. /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt.nii.gz
    Output: 
    1. TrainingData.RData 
    2. <ID>.feat/fix4melview_TrainingData_thr60.txt
    3. /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt_denoised.nii.gz

  
8. **F_FLIRT_normalization.sh**

 This script performs a three step registration procedure. First, the func nifti is reoriented to std (no registration applied). Then 1. a functional-to-anatomical registration matrix is created (with epi_reg: BBR registration), 2. and an anatomical-to-3mm_MNI matrix. 3. These are then concatenated and the resulting registration matrix is applied to the funtional data.

 Input: /NKI_enFaceHouseDatahanced_rest/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt_denoised.nii.gz
 
 Output: /FaceHouseData/A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt_denoised_MNI3mm_flirt.nii.gz

 
## PLS

1. **A_edit_txtfiles.sh**

 This script adjusts and copies the /B_PLS/template_batchfile.txt file for every subject and adjusts it
 Input: /B_PLS/template_batchfile.txt
 Ouput: /B_PLS/batchfiles/<ID>_batchfile.txt

2. **B_Sessiondatamat_creation**
 
 This script creates session data mat-files for PLS from txt template (/B_PLS/batchfiles/<ID>_batchfile.txt). It creates the structure of the sessiondata.mat files, needed for PLS. This file structure can then be altered to our needs in later steps and filled with different values of interest.

 Template path: /B_PLS/batchfiles/
 
 Output: /B_PLS/B_meanPLS/mean_<PROJECT>_<ID>_BfMRIsessiondata.mat 

3. **C_GMcommonCoords.m**

 This script creates a mask of coordinates including only gray matter (GM) coordinates and commonly activated coordinates for all subjects in the sample called final coordinates. Outputs _GMcommoncoords.mat_.

 Data path (preprocessed nifti): /A_preproc/data/<ID>/fMRI/<ID>_func_feat_BPfilt_denoised_MNI3mm_flirt.nii.gz
 
 Output: G_standards_masks/GM_mask/GMcommoncoords.mat

4. **D_SD_BfMRIsessiondatamat_creation.m**

This script performs the 2nd pre-step for PLS: Fill matrix with SD values for various condtions, runs and blocks. It also adds a fourth condition: SD house - SD face
Input: /B_PLS/B_meanPLS/mean_<PROJECT>_<ID>_BfMRIsessiondata.mat 
Output: /B_PLS/A_SD_PLS/SD_<PROJECT>_<ID>_BfMRIsessiondata.mat 

5. **E_mean_BfMRIsessiondatamat_creation.m**

This script adds a fourth condition to the mean_sessiondata.mats: house - face 
Input: /B_PLS/B_meanPLS/mean_<PROJECT>_<ID>_BfMRIsessiondata.mat 
Ouput: /B_PLS/B_meanPLS/mean_<PROJECT>_<ID>_BfMRIsessiondata.mat (with four conditions instead of three)

## Image metrics
6. **A_FaceHouse_image_properties.m**

This script performs calculation of:
(1) contrast (or “variance”): intensity contrast between neighbouring pixels over an entire 2D image 
(2) correlation: correlation between neighbouring pixel intensities over a 2D image
(3) energy (or “inertia”): the sum of squared elements in the grey-level co-occurrence matrix (GLCM)
(4) homogeneity: measure of the closeness of the distribution of elements in the GLCM to the GLCM diagonal
(5) local entropy (using entropyfilt): This function computes entropy on greyscale intensities from 9 * 9 pixel neighbourhoods, within-image; we then averaged all local neighbourhoods within-image to estimate a single average image entropy per face and house image. 
(6) local standard deviation (stdfilt): using using the same grey scale intensity neighborhood approach as above (9 * *9 pixel), this function computes the standard deviation (SD) for each neighborhood

for each of the input images. 

Input: /F_images_experiment/*
Ouput: /D_figures/*

7. **B_run_hmax_s1_c1_s2_c2.m**

This script runs the hmax model on the input images. 
Input: /F_images_experiment/*
Output: /C_image_metrix/hmax_output/*
     s1, s2, c1, c2
