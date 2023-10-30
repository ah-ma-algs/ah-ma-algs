3D Facial reconstruction can be broadly defined as interpolating facial features from a 2D image/plane to 3D space.  
The complexity arises due to environmental factors, application needs and computational complexity.  

Recent learning-based approaches, in which models are trained by single-view images have shown promising results for monocular 3D face reconstruction, but they suffer from the ill-posed face problem and depth ambiguity issues.  

3D face reconstruction techniques are generally categorized into some classes such as 3DMM, one shot learning, epipolar geomtry and maybe even shape from shading but as you go on, you will find that the most successive models are the hybrid ones and they all do use some methods from different classes in tandem.  

I will use [NextFace](https://github.com/abdallahdib/NextFace) which gave me the following results after many modifications for stability and better results, but I think discussing other models will be very fitting to better understand what we use, and why we chose it for this particular application.  

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images.md/render_0.png){: width="640" height="363" } 
_[Well lit image example.]_

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/Hugo.png){: width="640" height="363" } 
_[A very hard image with self shadows and bad lighting.]_

Such results can vary drastially from one subject to another, some models can outperform the others in capturing details and depth on perfect lighting conditions but fail drastically when met with self shadows, lighting shadows and unconditional diffuse.  

We can start with discussing the application as it can drastically affect the chosen method, Normally we either repurpose/train a method/network for our needs or we are lucky enough to have something that already fits out needs.  

I will discuss some of the applications I encountered below, you can directly skip to the next paragraph where I discuss how I used nextface and my remarks on fine-tuning its parameters and the installation process actually being different that what's on their profile.  

## Applications  
3DMM based approaches perform facial biometrics extraction which means they can be useful for applications such as facial recognition as it extracts useful info other than just the facial landmarks.  

Assuming we need a 3D caricature drawing system, Han et al.](https://ieeexplore.ieee.org/document/8580421) proposed a sketching system that creates 3D caricature photos by first interpolating from a 2D image to 3D using a vertex wise exaggeration map then modifying the facial feature curves.  
two images are generated using the 3D generated mesh as a guide for the warping of the 2D image and also for the image enhancement as the output is always blurry, and occlusions introduced failure, and as always with all of the networks, darker skin tones are much more challenging.  
Below is an example that shows how good it can modify facial features based on a simple drawing, It can be used to modift facial features for face editing apps such as faceapp(https://www.faceapp.com/) reducing or englarging certain features such as noses, or eyes.  

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/Caricature-shop.png){: width="640" height="363" } 
_[An example of exaggerated facial features. but they do expose its true potential.]_

[Zhang et al.](https://arxiv.org/abs/2007.12494) on the other hand proposed an automatic landmark detection, occlusion-aware multi-view synthesis method then 3D face restoration for caricatures.   
It aims to solve depth ambiguity and occlusion loss, which it does through creating a relationship between 2D images and 3D images using a covisiblity map that stores the mask of covisible pixels for
each target-source view pair to solve self-occlusion which is somewhat similar to NextFace's weightDiffuseSymmetryReg.  
It also introduced novel loss functions for multi-view consistency,   
It uses A ResNet model for encoding the input image to a latent space, and a decoder with a fully connected layer to generate 3D landmarks on the caricature.  

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/Multi-view.png){: width="640" height="363" } 
_[An example of exaggerated facial features. but they do expose its true potential.]_
  
[Thies et al.](https://arxiv.org/abs/1912.05566) is an audio-driven facial reenactment framework driven by a deep neural network that to output a photo-realisitic video of the target in sync with the audio a latent 3D face model space.  
It uses DeepSpeech recurrent neural networks using the latent 3D model space and Audio2ExpressionNet was responsible for converting the input audio to a particular facial expression.

Li et al. [45] proposed SymmFCNet, a symmetry consistent convolutional neural network for reconstructing missing pixels on the one-half face using the other half. SymmFCNet consisted of illumination-reweighted warping and generative reconstruction subnet. The dependency on multiple networks is a significant drawback

## nextFace
We discuss some skin features that can be modeled to give a better representation of the face to appear more realistic.  

### Reflectance
Reflectance is a material's ability to reflect incident radiant energy, in our case, that energy is light.  
Most of the surfaces's reflectance can be divided into specular and diffuse reflectance.  
For surfaces such as those of polished metals, water and glass/mirror surfaces, reflection can be assumed to be all specular.
For surfaces such as a human face or matte white paint, reflection can be assumed to be all diffuse.

https://en.wikipedia.org/wiki/Reflectance
https://en.wikipedia.org/wiki/Specular_reflection


:-------------------------:|:-------------------------:
![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Specular_reflection.png)  |  ![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Diffuse_reflection.png)
[Specular reflection](https://en.wikipedia.org/wiki/Reflectance)            |  [Diffuse reflection](https://en.wikipedia.org/wiki/Reflectance)


A skin's reflectance is mainly diffuse which Nextface models as a lambertian reflection with a distant light illumination.  
A good thing about NextFace is that it does 3D face reconstruction with explicit separation of face attributes, which is very useful as it later optimizes those attributes separately on later stages.  
We can also use those attributes for facial recognition.  
Such attributes are baked into each other, which is why we optimize them separately.  
In the first stage, we optimize the pose, illumination, geometry, diffuse and specular albedos, statistically regularized by the 3DMM,  
while specular roughness remains fixed, using ray tracing also helps extract self shadows from interplay with face geometry.  
In the second stage, we extract unconstrained diffuse reflectance, specular reflectance and roughness that capture specific facial attributes not modeled via statistical diffuse or specular albedos. 
This staged optimization strategy adds structure and makes the under-constrained optimization problem tractable, leading to superior reconstruction vs. the naive approach.

To capture a skin's reflectance we need a light stage to model the input image's lighting conditions, That's why we construct a light stage.

### Light stage
A novel virtual light stage formulation, which in-conjunction
with differentiable ray tracing, obtains more accurate scene illu-
mination and reflectance, implicitly modeling self-shadows. The
virtual light stage models, the switch from point to directional
area lights and vice-versa, Sec. 3.
• Face reflectance – diffuse, specular and roughness reconstruction
that is scene illumination and self-shadows aware.
• A robust optimization strategy that extracts semantically mean-
ingful personalized face attributes, from unconstrained images,

In-order to use unconstrained monocular images, statistical priors
have been introduced [ZTB ∗ 18]. Such priors add structure to the re-
construction formulation.


While such approaches lead to highly accu-
rate skin (diffuse and specular) reflectance modeling, they require
controlled capture conditions and extensive calibration. Our aim,
instead, is to robustly extract face attributes from unconstrained
images, where a highly accurate skin reflectance models may not
be applicable due to the low quality of input images

Most face reconstruction approaches rely on a lightweight paramet-
ric skin reflectance model using linear Lambertian models, where it
is assumed that skin does not have specular attributes. This simpli-
fication has shown great success for face reconstruction

illumination model
relying on Spherical Harmonics [RH01] that assume Lambertian re-
flectance.

Most approaches assume that illumination is mostly
uniform resulting in self-shadow being baked into albedo attribute.

a novel parameterized virtual area light
stage is introduced that simulates real world illumination conditions.
This illumination model is used together with ray tracing, that im-
plicitly models self-shadow attributes. Consequently, it reconstructs
geometric patch’s reflectance separating incurred shadows (Sec 3.3).
To the best of our knowledge, the proposed method is the first to
estimate reflectance (diffuse, specular, roughness), illumination, and
self-shadows robustly from monocular images

While quality
face tracking has several advantages, such as reenactment, realis-
tic virtual avatars [SSKS17, KGT ∗ 18], attributes separation opens
up new possibilities. Photoshop-like applications for face portrait
touch-up have been proposed. For example, [SPB ∗ 14] shows how
style from one image can be transferred to another employing image-
based methods for style transfer. [SHS ∗ 17] proposes a method for
illumination transfer from source to target images, while [SBT ∗ 19]
describes a method for portrait relighting. More recently, [ZBT ∗ 20]
proposes a method for foreign shadow removal from images. Since
our method can separates several face attributes, it makes many such
applications feasible, as discussed in the paper.

we introduce our optimiza-
tion formulation that relies on differentiable ray tracing for image
synthesis. By varying the number of ray-bounces against scene ge-
ometries and subsequent indirect illumination, self-shadows can be
modeled.

By using area lights that can be turned on or off, and by
controlling their intensity, position and surface area, we are capable
of modeling several illumination and self-shadow scenarios.


symmetric regularizer that prevents
get baked into C.
ˆ penalizing for a image-based
baking of the residual shadow into C,
ˆ C) the consis-
imbalance between the two sides of the face



One solution is to use high number of sample points for sampling
along edges. However, this is computationally infeasible. Several
techniques [LHJ19, LADL18] have been proposed to overcome this
limitation. In our work, we rely on [LADL18]’s technique to explic-
itly sample the geometry edges – a costly yet mandatory operation
needed for correct geometric shape estimation

solving for the rendering equation [Kaj86] via Monte Carlo
ray tracing, very few points on the edge of the geometric shape are
sampled, causing a discontinuity along the edges. As a result, back-
propagation based gradients calculation fails to take into account
sensitive information along the geometric edges. Consequently, the
gradients on the edges remain noisy



We note that [YS ∗ 18] and [LMG ∗ 20] estimates displace-
ment/normal maps while our method does not. This requires high-
quality and well lit input images (as reported by authors) for optimal
results. Additionally, [LMG ∗ 20] estimates reflectance maps for full
face head in the UV space, whereas our method restricts recon-
struction to frontal face only. [SSD ∗ 20] estimates light (three bands
spherical harmonics) but, may not correctly estimate personalized
reflectance outside the statistical albedo space. A complete catalog
of comparisons against these methods is available in the supplemen-
tary material (section C).

Another limitation of our method is reliance on statistical albedo
priors (Optimization, Stage I) that do not model certain skin tones

We note that our albedos (esp. roughness) attributes are view and
input image illumination condition dependent, however, when avail-
able, statistical priors help give meaningful estimates. Here, our
method relies on symmetry, consistency and smoothness regulariz-
ers (Eq 6) to avoid overfittingHere, our
method relies on symmetry, consistency and smoothness regulariz-
ers (Eq 6) to avoid overfitting. In some cases, due to these regulariz-
ers, person specific attributes are not captured. Additionally, while
the consistency and symmetry regularizers (Stage-II) help avoid bak-
ing shadows in the final albedo, in some cases, when the optimized
light and consequent shadows are inaccurate, some light/shadow
patches may appear in the estimated albedos



Currently, we use single bounce rays for illumi-
nation modeling due to lack of external scene geometries, a natural
extension is to model multi-ray bounces for softer shadows.


read from page 9



https://www.pluralsight.com/blog/film-games/bump-normal-and-displacement-maps

https://en.wikipedia.org/wiki/Albedo
