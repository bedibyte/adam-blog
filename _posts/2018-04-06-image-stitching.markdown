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
* **Step 4:** Warp input images individually
* **Step 5:** Add warped images together

### **Step 1**
The first step in designing the image stitching algorithm is to extract features (specifically, keypoints and descriptors) from all input images.

```Shell
    def stitch(self, images, ratio=0.75, reprojThresh=4.0, showMatches=False):  
        image_count = len(images)

        for i in range(0, image_count):
            (kps_i, des_i) = self.detectAndDescribe(images[i]) 
            kps.append(kps_i) 
            des.append(des_i)
```

<u>Line 1</u> sets up the `stitch` function with `images` as its input. The variable `images` (may consists up to an infinite amount) will be the images that we're going to stitch together. It is important to note that the `images` list must be taken in according to order as the code will stitch these images from left to right, starting from the first image on the list to the next.

<u>Line 2</u> calculates the number of input images. Once we have done that, we create a `for` loop on <u>Line 4</u> to account for all of the images and call the `detectAndDescribe` function in the loop on <u>Line 5</u>. This function detects keypoints and extracts local invariant descriptors (i.e. SIFT) from the images.

Finally, <u>Lines 6 and 7</u> append the resulting keypoints and descriptors into our `kps` and `des` lists, respectively.

### **Step 2**

Our next step is to match correspondences between images. 

```Shell
        for i in range(0, image_count - 1):
            M_i = self.matchKeypoints(kps[i], kps[i+1],
                                       des[i], des[i+1], ratio, reprojThresh)
            (matches_i, H_i, status_i) = M_i
            M.append(M_i)
            matches.append(matches_i)
            H.append(H_i)
            status.append(status_i)
```

Using the `kps` and `des` lists found in the previous step, <u>Lines 2 and 3</u> above call the `matchKeypoints` function to match the features from all adjacent images (i.e. image_0 with image_1, image_1 with image_2, and so on). This method will be explained in more detail later at the bottom part of this post. 

<u>Line 4</u> extracts the outputs of the function. The most important output is the homography matrix, `H_i` which is derived from the RANSAC algorithm. A homography matrix is typically an image transformation matrix that tells you how one image relates to the other. In mathematics, this can be equated by *image_0 = H_01 X image_1*, where H_01 relates image_1 to image_0. Hence, the homography matrices found in this step are H_01, H_12, and so on.

<u>Lines 5 - 8</u> append the results to their respective lists. 

### **Step 3**

To create a panorama, we need homography matrices of all images to be with respect to only one chosen anchor image. For example, in our case where image_0 is chosen as the anchor image, the homography matrices needed to be found are H_01, H_02, and so on. Hence, we need to tweak all of the homography matrices found from the previous step.

```Shell
        Href = []

        for i in range(0, image_count):
            if i == 0:
                Href_i = np.identity(3)
                Href.append(Href_i)
            else:
                Href_i = np.dot(Href[i-1], np.linalg.inv(H[i-1]))
                Href.append(Href_i)
```

<u>Line 1</u> above initializes the empty `Href` list. 

<u>Lines 4 - 6</u> under the `for` loop sets image_0 as our anchor image. This can be done by setting its homography matrix, H_0 as an identity matrix. 

Using basic matrix multiplication rules, <u>Lines 7 - 9</u> compute new homography matrices of all images with respect to image_0. For example, *H_01 = H_0 X inv(H_01)* and *H_02 = H_01 X inv(H_12)*. Once we have done that, the matrices are appended to our `Href` list.

### **Step 4**

Next, we warp all input images with respect to the chosen anchor image using the computed homography matrices found in the previous step.

```Shell
        warped = []

        for i in range(0, image_count):
            warped_i = cv2.warpPerspective(images[i],Href[i],
                                           (images[0].shape[1]*image_count,
                                            images[0].shape[0]))   
            warped.append(warped_i)
```

<u>Line 1</u> initializes the empty `warped` list. 

<u>Lines 3 - 6</u> handle the warping of the images. More specifically, <u>Line 4</u> ensures that `images[i]` is warped using its corresponding calculated homography `Href[i]`, while <u>Line 5</u> makes sure that the length of the warped image is equal to the total length of all the images and <u>Line 6</u> sets the height of the warped image to be the height of image_0 (Note: since we are stitching from left to right, the height of warped image stays the same). 

Again, <u>Line 7</u> appends the warped images to our `warped` list.

### **Step 5**
Once we get all of the warped images, we simply "add" all of them together to create a panorama. To do that, we first need to obtain the pixel size (specifically, height and length) of the warped images.

```Shell
        x = []
        y = []

        for i in range (0, image_count):
            (y_i, x_i) = warped[i].shape[:2]
            y.append(y_i)
            x.append(x_i)
```
<u>Lines 1 and 2</u> initialize the length `x` and height `y` variables.

<u>Lines 4 - 7</u> obtain the height and length of each warped image and save the values in the corresponding `x` and `y` lists.

Using this information, we can now create the panorama. This is done by comparing one warped image to the next (from left to right), pixel by pixel. 

```Shell
        for i in range(0, image_count-1):
            for m in range(0, x[i]):
                for n in range(0, y[i]):
                    try:
                        if(np.array_equal(warped[i][n,m],np.array([0,0,0]))  # where [n,m] are pixel coordinates
                           and np.array_equal(warped[i+1][n,m],np.array([0,0,0]))):
                            warped[i+1][n,m] = [0,0,0]
                        else:
                            if(np.array_equal(warped[i+1][n,m],[0,0,0])):
                                warped[i+1][n,m] = warped[i][n,m]
                            else:
                                if not np.array_equal(warped[i][n,m],[0,0,0]):
                                    warped[i+1][n,m] = warped[i][n,m]
                    except:
                        pass

        for i in range(image_count):
            result = warped[i]
```
<u>Lines 1 - 15</u> handle the stitching of all warped images. To illustrate, if at a particular pixel both the left and right images are black, <u>Lines 4 - 7</u> output the color black. <u>Lines 8 - 13</u> on the other hand ensure to output only the left image if only the right image is black at that pixel, or if both images are not black.

Finally, <Lines 17 - 18</u> combine the `warped` list to our `result` variable to create a panorama.

## BLOG POST IS UNDER CONSTRUCTION
