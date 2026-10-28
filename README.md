## Colmap Photogrammetry


![colmap_photogram_01](/images/Screenshot_09.png)

## 

## 

## Colmap Photogrammetry



This walkthrough is meant for a local build & python environment, with a well-defined photo sequence for SfM pipelines.
The result from this run still has artefacts and inaccuracies, which can be significantly reduced by point cloud cleanups and higher resolution settings. 



### Prerequisites



Project folder hierarchy:

```
    project root folder
    │
    ├── images
    └── masks
```


• a clean photo sequence, ideally focus-stacked in /images, a corresponding mask set with the exact names of their original photos in /masks 

• colmap 4.2.x with CUDA, download here: github.com/colmap/colmap/releases

• NVIDIA Driver (Windows x64) 552.xx

• Microsoft Visual C++ Redistributable x64: 14.40 (2015-2020)

• CUDA Toolkit 12.4

• recommended: MeshLab \& CloudCompare

#

### Command Chain

<br>
<br>

1. *set your project path:*
   
   set "DATASET=[PATH_TO_PROJECT]"

<br>
<br>

2. colmap feature_extractor ^
   
    --database_path "%DATASET%/database.db" ^
   
    --image_path "%DATASET%/images" ^
   
    --ImageReader.mask_path "%DATASET%/masks"

   
<br>
<br>

3. *these settings are specific for a DISC3D scan but works for most cases with a clean camera path:*

   
   colmap sequential_matcher ^
   
    --database_path "%DATASET%/database.db" ^
   
    --SequentialMatching.overlap 10 ^
   
    --SequentialMatching.loop_detection 1


<br>
<br>

4. mkdir "%DATASET%/sparse"

<br>
<br>
   
5. colmap mapper ^
   
    --database_path "%DATASET%/database.db" ^
   
    --image_path "%DATASET%/images" ^
   
    --output_path "%DATASET%/sparse"

<br>
<br>

6. mkdir "%DATASET%/dense"

<br>
<br>

7. colmap image_undistorter ^
   
    --image_path "%DATASET%/images" ^
   
    --input_path "%DATASET%/sparse/1" ^
   
    --output_path "%DATASET%/dense" ^
   
    --output_type COLMAP


#

This results in a Dense Point Cloud in /dense, these were the unedited results:



![colmap_photogram_01](/images/Screenshot_01.png)


![colmap_photogram_01](/images/Screenshot_02.png)


![colmap_photogram_01](/images/Screenshot_03.png)

#

<br>
<br>

8. *this will provide the photometric as well as the geometric data, both are required for a clean photogrammetric model:*


    colmap patch_match_stereo ^
   
     --workspace_path "%DATASET%/dense" ^
   
     --workspace_format COLMAP ^
   
     --PatchMatchStereo.geom_consistency true ^
   
     --PatchMatchStereo.filter true ^
   
     --PatchMatchStereo.filter_min_ncc 0.15 ^
   
     --PatchMatchStereo.filter_min_triangulation_angle 3 ^
   
     --PatchMatchStereo.filter_min_num_consistent 3 ^
   
     --PatchMatchStereo.max_image_size 1400



   *the max_image_size value can be set significantly higher for a better resolution, depending on the available system RAM*

 
<br>
<br>
      
9. colmap stereo_fusion ^
      
    --workspace_path "%DATASET%/dense" ^
      
    --workspace_format COLMAP ^
   
    --input_type geometric ^
   
    --output_path "%DATASET%/dense/fused.ply" ^
   
    --StereoFusion.mask_path "%DATASET%/masks" ^
   
    --StereoFusion.min_num_pixels 5 ^
   
    --StereoFusion.max_reproj_error 1 ^
   
    --StereoFusion.max_depth_error 0.01 ^
   
    --StereoFusion.max_normal_error 10 ^
   
    --StereoFusion.check_num_images 50 ^
   
    --StereoFusion.max_image_size 1400 ^
   
    --StereoFusion.num_threads 4 ^
   
    --StereoFusion.cache_size 4
    
<br>
<br>

10.  *at this point I recommend a point cloud cleanup, eg in CloudCompare, in order to prevent artefacts and floating geometry*
    
<br>
<br>
    
11.  *here the actual mesh is generated:*
    

      colmap poisson_mesher ^
     
        --input_path "%DATASET%\dense\fused.ply" ^
     
        --output_path "%DATASET%\dense\object_mesh.ply" ^
     
        --PoissonMeshing.trim 7
<br>
<br>

#
     
![colmap_photogram_01](/images/Screenshot_04.png)


![colmap_photogram_01](/images/Screenshot_05.png)
     
#

<br>
<br>

13. *this model can get really really large, in my case I aimed for a smaller mesh resolution with these flags:*


      colmap poisson_mesher ^
    
       --input_path "%DATASET%/dense/fused.ply" ^
    
       --output_path "%DATASET%/dense/object_mesh.ply" ^
    
       --PoissonMeshing.depth 11 ^
    
       --PoissonMeshing.trim 7

<br>
<br>

14. mkdir "%DATASET%/dense/textured"

<br>
<br>

15. *the cool part, where textures is added per face from the photo sequence:*


      colmap mesh_texturer ^
    
       --workspace_path "%DATASET%/dense" ^
    
       --input_path "%DATASET%/dense/object_mesh.ply" ^
    
       --output_path "%DATASET%/dense/textured"

<br>
<br>

#

![colmap_photogram_01](/images/Screenshot_06.png)


![colmap_photogram_01](/images/Screenshot_07.png)


![colmap_photogram_01](/images/Screenshot_08.png)
