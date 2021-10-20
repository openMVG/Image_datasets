OpenMVG Drone image samples
===============================

- Palm_Desert_Micro (21 GPS images)

# Associated demos (OpenMVG 2.0)
- Processing with or without GPS priors
- Processing with image similarity (VLAD or PRE-EMPTIVE matching)

## SFM without GPS prior but adjust the scene to GPS datum by Rigid adjustment to GPS coordinates

```console
dataset=<i.e. FULL_PATH to Palm_Desert_Micro>
dataset_out=<i.e.FULL_PATH Palm_Desert_Micro_output>

mkdir $dataset_out

# Initialize the scene
openMVG_main_SfMInit_ImageListing -i $dataset -o $dataset_out/matches -d <PATH to sensor_width_camera_database.txt> -c 3

# Compute of the features (you can add -n <THREAD_COUNT> to be faster)
openMVG_main_ComputeFeatures -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches

# Compute the image matches
openMVG_main_ComputeMatches -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches/matches.putatives.bin

# Filter matches
openMVG_main_GeometricFilter -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches/matches.putatives.bin -g f -o $dataset_out/matches/matches.f.bin

# Compute of the camera motion and structure of the scene
openMVG_main_SfM --sfm_engine "INCREMENTAL" -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches -o $dataset_out/reconstruction

# Rescale to GPS coordinates
openMVG_main_geodesy_registration_to_gps_position -i $dataset_out/reconstruction/sfm_data.bin -o $dataset_out/reconstruction/sfm_data_adjusted.bin
# Visualization in PLY
openMVG_main_ConvertSfM_DataFormat -i $dataset_out/reconstruction/sfm_data_adjusted.bin -o $dataset_out/reconstruction/sfm_data_adjusted.ply

# Load sfm_data_adjusted.ply and file GPS_position.ply to see side by side (priors and what SFM found)
# Note preview could be 'strange' due to floating precision and large coordinates

```

## Using GPS prior for pair selection and SFM prior (optimize in GPS XYZ coordinates)

```console
mkdir $dataset_out

# Initialize the scene with GPS prior
openMVG_main_SfMInit_ImageListing -i $dataset -o $dataset_out/matches -d <PATH to sensor_width_camera_database.txt> -c 3 --use_pose_prior

# Initialize a View Graph for image matching using image GPS positions and X neighbors (here X equals 10)
openMVG_main_ListMatchingPairs -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches --gps_mode --neighbor_count 10 -o $dataset_out/matches/pairs.txt

# Compute of the features (you can add -n <THREAD_COUNT> to be faster)
openMVG_main_ComputeFeatures -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches

# Compute the image matches
openMVG_main_ComputeMatches -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches/matches.putatives.bin --pair_list $dataset_out/matches/pairs.txt

# Filter matches
openMVG_main_GeometricFilter -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches/matches.putatives.bin -g f -o $dataset_out/matches/matches.f.bin

# Compute of the camera motion and structure of the scene (scene will be in GPS prior coordinates)
openMVG_main_SfM --sfm_engine "INCREMENTAL" -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches -o $dataset_out/reconstruction --prior_usage
# Note: Ignoring the --prior_usage (will ignore GPS coordinates priors -- so you could register to GPS position manually using openMVG_main_geodesy_registration_to_gps_position)
# Note preview could be 'strange' due to large difference of coordinates with the GPS position and SFM point cloud that will be displayed side by side

```

## Using VLAD similarity image pair selection (demo -- no GPS consideration)

```console
mkdir $dataset_out

# Initialize the scene with GPS prior
openMVG_main_SfMInit_ImageListing -i $dataset -o $dataset_out/matches -d <PATH to sensor_width_camera_database.txt> -c 3

# Compute of the features (you can add -n <THREAD_COUNT> to be faster)
openMVG_main_ComputeFeatures -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches

# Compute a VLAD dictionary, compute a VLAD descriptor for each image, and retrieve the closest images by using VLAD similarity
openMVG_main_ComputeVLAD -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches --pair_file $dataset_out/matches/vlad_pairs.txt
# Notes:
# Visualize for each image, the closest image retrieved, you can open $dataset_out/matches/retrieval_matches_matrix.svg
# Visualize the retrieved adjacency matrix, you can open  $dataset_out/matches/retrieval_matches_matrix.svg

# Compute matches for the selected VLAD image pairs
openMVG_main_ComputeMatches -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches/matches.putatives_vlad.bin --pair_list $dataset_out/matches/vlad_pairs.txt

# Filter matches
openMVG_main_GeometricFilter -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches/matches.putatives_vlad.bin -g f -o $dataset_out/matches/matches.f.bin

# Compute of the camera motion and structure of the scene (scene will be in GPS prior coordinates)
openMVG_main_SfM --sfm_engine "INCREMENTAL" -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches/ -o $dataset_out/reconstruction

```

## Using Preemptive matching for image pair selection (demo -- no GPS consideration)

```console
mkdir $dataset_out

# Initialize the scene with GPS prior
openMVG_main_SfMInit_ImageListing -i $dataset -o $dataset_out/matches -d <PATH to sensor_width_camera_database.txt> -c 3

# Compute of the features (you can add -n <THREAD_COUNT> to be faster)
openMVG_main_ComputeFeatures -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches

# Compute matches with preemptive matching to select only some image pairs
# i.e here we keep per image 4000 features with the largest scale
openMVG_main_ComputeMatches --preemptive_feature_count 4000 -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches/matches.putatives_preemptive_V1.bin --pair_list $dataset_out/matches/preemptive_pairs.txt

# Compute matches for the selected preemptive pairs
openMVG_main_ComputeMatches -i $dataset_out/matches/sfm_data.json -o $dataset_out/matches/matches.putatives_preemptive_V2.bin --pair_list $dataset_out/matches/preemptive_pairs.txt

# Filter matches
openMVG_main_GeometricFilter -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches/matches.putatives_preemptive_V2.bin -g f -o $dataset_out/matches/matches.f.bin

# Compute of the camera motion and structure of the scene (scene will be in GPS prior coordinates)
openMVG_main_SfM --sfm_engine "INCREMENTAL" -i $dataset_out/matches/sfm_data.json -m $dataset_out/matches/ -o $dataset_out/reconstruction

```
