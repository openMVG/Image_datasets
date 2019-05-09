OpenMVG spherical image samples
===============================

- Flat (11 indoor images)
- School (4 outdoor images)

Here some command line instructions to run OpenMVG pipeline on 360 images content:

```console
$dataset=<i.e. FULL_PATH to 360_Flat/images>
$dataset_out=<i.e.FULL_PATH to 360_Flat>

# Initialize the scene (spherical camera model)
openMVG_main_SfMInit_ImageListing -i $dataset -o $dataset_out/matches -f 1 -c 7

# Compute of the features (you can add -n <THREAD_COUNT> to be faster)
openMVG_main_ComputeFeatures -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches

# Compute the image matches (notice the angular mode, more adapted for spherical images)
openMVG_main_ComputeMatches -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches -g a

# Compute of the camera motion and structure of the scene
openMVG_main_IncrementalSfM -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches -o $dataset_out/reconstruction

#
# MVS (optional)
#

# Export the scene to a cubic scene (there is no open source MVS tool that consider spherical images as input)
# - each sphere is exported as 6 pinholes images
#
openMVG_main_openMVGSpherical2Cubic -i $dataset_out/reconstruction/sfm_data.bin -o $dataset_out/reconstruction/cubic

# Convert the scene from OpenMVG to OpenMVS data format
openMVG_main_openMVG2openMVS -i $dataset_out/reconstruction/cubic/sfm_data_perspective.bin -o $dataset_out/reconstruction/cubic/scene.mvs -d $dataset_out/reconstruction/cubic/openmvs_images

# Run OpenMVS to obtain a dense point cloud
DensifyPointCloud $dataset_out/reconstruction/cubic/scene.mvs
```
