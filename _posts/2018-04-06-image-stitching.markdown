---
layout: post
title: Image Stitching
date: 2018-04-06 13:32:20 +0300
description: For this post, I am writing about another computer vision algorithm my team and I designed for our senior design project last year, which is an image-stitching algorithm.
img: post-3.png # Add image post (optional)
tags: [image-stitching, OpenCV, Python]
---
For this post, I am writing about another computer vision algorithm my team and I designed for our senior design project last year, which is an image-stitching algorithm. The code is written in Python using OpenCV (with `numpy` and `imutils` installed).

## Features
-----
The image stitching algorithm is able to stitch an **unlimited** amount of images together in order from left-to-right orientation to create a panorama.

## Procedure
-----
The key steps in designing the code are:
* **Step 1:** Extract features from input images
* **Step 2:** Match correspondences
* **Step 3:** Calculate homography matrix, H with respect to an anchor image
* **Step 4:** Warp all input images individually
* **Step 5:** Add all warped images together

### **Step 1**
The first step in designing the image stitching algorithm is to extract features (specifically, descriptors and keypoints) from input images.

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

<u>Line 1 sets up the `stitch` body with `images` as its input. The variable `images` (may consists up to an infinite amount) will be the images that we're going to stitch together.

```Shell
        for i in range(0, image_count - 1):
            M_i = self.matchKeypoints(kps[i], kps[i+1],  # calls "matchKeypoints" class located at the bottom of code
                                       features[i], features[i+1], ratio, reprojThresh)
            (matches_i, H_i, status_i) = M_i
            M.append(M_i)
            matches.append(matches_i)
            H.append(H_i)
            status.append(status_i)
```
## BLOG POST IS UNDER CONSTRUCTION
