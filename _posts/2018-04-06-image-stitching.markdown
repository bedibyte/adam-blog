---
layout: post
title: Image Stitching
date: 2018-04-06 13:32:20 +0300
description: For this post, I am writing about another computer vision algorithm my team and I designed for our senior design project last year, which is an image-stitching algorithm.
img: post-3.png # Add image post (optional)
tags: [Python, OpenCV]
---
For this post, I am writing about another computer vision algorithm my team and I designed for our senior design project last year, which is an image-stitching algorithm. The code is written in Python using OpenCV (with `numpy` and `imutils` installed).

## Features
-----
The image stitching algorithm is able to stitch an unlimited amount of images together in order from left-to-right orientation to create a panorama.

## Procedure
-----
The key steps in designing the code are:
* **Step 1:** Extract features from input images
* **Step 2:** Match correspondences
* **Step 3:** Calculate homography matrix, H with respect to an anchor image
* **Step 4:** Warp all input images individually
* **Step 5:** Add all warped images together

### **Step 1**
The first step in designing the image stitching algorithm is to extract features (specifically, descriptors and keypoints) from all input images.

```Shell
    def stitch(self, images, ratio=0.75, reprojThresh=4.0, showMatches=False):
    
        image_count = len(images)
        kps = []
        features = []

        for i in range(0, image_count):
            (kps_i, features_i) = self.detectAndDescribe(images[i]) 
            kps.append(kps_i) 
            features.append(features_i)
```

<u>Line 1</u> sets up the `stitch` function with `images` as its input. The variable `images` (may consists up to an infinite amount) will be the images that we're going to stitch together. It is important to note that the `images` list must be taken in according to order as the code will stitch these images from left-to-right, starting from the first image on the list to the next.

<u>Line 3</u> calculates the number of input images, while <u>Lines 4 and 5</u> initialize `kps` and `features` as empty matrices. 

Once we have done that, we call the `detectAndDescribe` function on <u>Line 8</u> for all of our input images. This function typically detects keypoints and extracts local invariant descriptors (i.e. SIFT) from all of the images.

Finally, <u>Lines 9 and 10</u> append the results into our `kps` and `features` matrices.

### **Step 2**
    Our next step involves using the gathered information (specifically, `kps` and `features`) from Step 1 to match correspondences between images. 

```Shell
        for i in range(0, image_count - 1):
            M_i = self.matchKeypoints(kps[i], kps[i+1],
                                       features[i], features[i+1], ratio, reprojThresh)
            (matches_i, H_i, status_i) = M_i
            M.append(M_i)
            matches.append(matches_i)
            H.append(H_i)
            status.append(status_i)
```

<u>Lines 2 and 3</u> above call the `matchKeypoints` function to match the features in our images. This method will be explained in more detail later at the bottom part of this post. The output of this function gives us a list of keypoint matches (`matches_i`), the homography matrix (`H_i`) which is derived from the RANSAC algorithm, and status (`status_i`) which is a list of indexes that indicates the keypoints in `matches` that were successfully verified using RANSAC.

The remaining four lines append the results to the respective defined matrices.

## BLOG POST IS UNDER CONSTRUCTION
