![colmap_photogram_01](/images/Screenshot_09.png)

## 

## 

## Colmap Photogrammetry



This walkthrough is meant for a local build & python environment, with a well-defined photo sequence for SfM pipelines



### Prerequisites


• colmap 4.2.x with CUDA, download here: github.com/colmap/colmap/releases

• NVIDIA Driver (Windows x64) 552.xx

• Microsoft Visual C++ Redistributable x64: 14.40 (2015-2020)

• CUDA Toolkit 12.4

• recommended: MeshLab \& CloudCompare


### Command Chain

1. set your project path:
   
   set "DATASET=[PATH_TO_PROJECt]"

2. colmap feature_extractor ^
   
    --database_path "%DATASET%/database.db" ^
   
    --image_path "%DATASET%/images" ^
   
    --ImageReader.mask_path "%DATASET%/masks"
   

4. these settings are specific for a clean DISC3D scan but works for most cases with a clean camera path

   
   colmap sequential_matcher ^
   
    --database_path "%DATASET%/database.db" ^
   
    --SequentialMatching.overlap 10 ^
   
    --SequentialMatching.loop_detection 1


8. 

