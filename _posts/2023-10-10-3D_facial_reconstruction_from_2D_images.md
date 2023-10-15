3D Facial reconstruction can be broadly defined as interpolating facial features from a 2D image/plane to 3D space.
The complexity arises due to environmental factors, application needs and computational complexity.

Recent learning-based approaches, in which models are trained by single-view images have shown promising results for monocular 3D face reconstruction, but they suffer from the ill-posed face problem and depth ambiguity issues.

3D face reconstruction techniques are generally categorized into some classes such as 3DMM, one shot learning, epipolar geomtry and maybe even shape from shading but as you go on, you will find that the most successive models are the hybrid ones and they all do use some methods from different classes in tandem.

I will use [NextFace](https://github.com/abdallahdib/NextFace) which gave me the following results, after many modifications for stability and better results, but I think discussing other models will be very fitting to better understand what we use, and why we chose it for this particular application.

We can start with discussing the application as it can drastically affect the chosen method, Normally we either repurpose/train a method/network for our needs or we are lucky enough to have something that already fits out needs.  

3DMM based approaches perform facial biometrics extraction which means they can be useful for applications such as facial recognition as it extracts useful info other than just the facial landmarks.

Assuming we need a 3D caricature drawing system, Han et al.](https://ieeexplore.ieee.org/document/8580421) proposed a sketching system that creates 3D caricature photos by first interpolating from a 2D image to 3D using a vertex wise exaggeration map then modifying the facial feature curves, two images are generated using the 3D generated mesh as a guide for the warping of the 2D image and also for the image enhancement as the output is always blurry, and occlusions introduced failure, and as always with all of the networks, darker skin tones are much more challenging.

[Zhang et al.](https://arxiv.org/abs/2007.12494) on the other hand proposed an automatic landmark detection, occlusion-aware multi-view synthesis method then 3D face restoration for caricatures. 
It aims to solve depth ambiguity and occlusion loss, which it does through creating a relationship between 2D images and 3D images using a covisible map that stores the mask of covisible pixels for
each target-source view pair to solve self-occlusion.
It also introduced novel loss functions for multi-view consistency, 
It uses A ResNet modelfor encoding the input image to a latent space, and a decoder with a fully connected layer to generate 3D landmarks on the caricature. 

[Thies et al.](https://arxiv.org/abs/1912.05566) is an audio-driven facial reenactment driven by a deep neural network that to output a photo-realisitic video of the target in sync with the audio a latent 3D face model space.
It uses DeepSpeech recurrent neural networks using the latent 3D model space and Audio2ExpressionNet was responsible for converting the input audio to a particular facial expression.


Li et al. [45] proposed SymmFCNet, a symmetry consistent convolutional neural network for reconstructing missing pixels on the one-half face using the other half. SymmFCNet consisted of illumination-reweighted warping and generative reconstruction subnet. The dependency on multiple networks is a significant drawback


We model some features of the skin such as reflectance.

Reflectance is a material's ability to reflect incident radiant energy, in our case, that energy is light.
Most of surfaces's reflectance can be divided into specular and diffuse reflectance.
For surfaces such as those of polished metals, water and glass/mirror surfaces, reflection can be assumed to be all specular.
For surfaces such as a human face or matte white paint, reflection can be assumed to be all diffuse.

https://en.wikipedia.org/wiki/Reflectance
https://en.wikipedia.org/wiki/Specular_reflection


![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/ٍSpecular_reflection.png){: width="640" height="363" } 
_[Reflection types for media](https://en.wikipedia.org/wiki/Reflectance)_

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/ٍDiffuse_reflection.png){: width="640" height="363" } 
_[Reflection types for media](https://en.wikipedia.org/wiki/Reflectance)_


https://www.pluralsight.com/blog/film-games/bump-normal-and-displacement-maps

