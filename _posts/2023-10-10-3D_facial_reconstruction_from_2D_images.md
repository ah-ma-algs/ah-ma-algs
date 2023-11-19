
 

 
$$\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)$$

polynomial vs spline functions
3D Facial reconstruction can be broadly defined as interpolating facial features from a 2D image/plane to 3D space.  
The complexity arises due to environmental factors, application needs and computational complexity.  

Recent learning-based approaches, in which models are trained by single-view images have shown promising results for monocular 3D face reconstruction, but they suffer from the ill-posed face problem and depth ambiguity issues.  

3D face reconstruction techniques are generally categorized into some classes such as 3DMM, one shot learning, epipolar geomtry and maybe even shape from shading but as you go on, you will find that the most successive models are the hybrid ones and they all do use some methods from different classes in tandem.  

I will use [NextFace](https://github.com/abdallahdib/NextFace) which gave me the following results after many modifications for stability and better results, but I think discussing other models will be very fitting to better understand what we use, and why we chose it for this particular application.  

I also noticed that many parts of NextFace's code were adopted from other sources without further scrutinizing them, not noticing that some of that code was written, constricted by C++'s limitations and when put into python, could have been rewritten to get as low as half the execution time or even more, which I modified and tested myself.

they require controlled capture conditions and extensive calibration. Our aim,
instead, is to robustly extract face attributes from unconstrained
images, where a highly accurate skin reflectance models may not
be applicable due to the low quality of input images

Most face reconstruction approaches rely on a lightweight paramet-
ric skin reflectance model using linear Lambertian models, where it
is assumed that skin does not have specular attributes. This simpli-
fication has shown great success for face reconstruction

The modifications I introduced to the code, got better details in shadows and gets a speed increase of 25-40% depending on the number of bands(spherical harmonics) used, and as the demand for better quality increases, my modifications seem to have a more prominent effect than using the original code which also highlights the cause to their remark of the diminishing returns when increasing the number of bands.

One other problem which I can't discuss here as it needs a book, rotations, the algorithm implemented is very slow as reported by Green and the other more complex algorithm for light rotations implemented by Choi is 10x faster and as Choi writes "Comlex makes life easier!"

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Faces.png){: width="640" height="363" } 
_[Well lit image example.]_

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/optimized.gif){: width="640" height="363" } 
_[All sides view animation.]_

I also tested many parameters on such a challenging image which I will discuss later.
![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/hugo.jpg){: width="640" height="363" } 
_[A very hard image with self shadows and bad lighting.]_

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Hugo.png){: width="640" height="363" } 
_[different outupts due to interplaying parameters.]_

Such results can vary drastially from one subject to another, some models can outperform the others in capturing details and depth on perfect lighting conditions but fail drastically when met with self shadows, lighting shadows and unconditional diffuse.  

We can start with discussing the application as it determines the effectiveness of the chosen method, Normally we either repurpose/train a method/network for our needs or we are lucky enough to have something that already fits out needs.  

I will discuss some of the applications I encountered below, you can directly skip to the next paragraph where I discuss how I used nextface and my remarks on fine-tuning its parameters and the installation process actually being a bit different than what's on their profile.  

## Applications  
3DMM based approaches perform facial biometrics extraction which means they can be useful for applications such as facial recognition as it extracts useful info other than just the facial landmarks.  

Assuming we need a 3D caricature drawing system, [Han et al.] (https://ieeexplore.ieee.org/document/8580421) proposed a sketching system that creates 3D caricature photos by first interpolating from a 2D image to 3D using a vertex wise exaggeration map then modifying the facial feature curves.  
Two images are generated using the 3D generated mesh as a guide for the warping of the 2D image and also for image enhancement as the output is always blurry, and occlusions introduced failure, and as always with all of the networks, darker skin tones are much more challenging.  
Below is an example that shows how good it can modify facial features based on a simple drawing, It can be used to modify facial features for face editing apps such as faceapp (https://www.faceapp.com/) reducing or englarging certain features such as noses, or eyes.  
  
![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Caricature-shop.png){: width="640" height="363" } 
_[An example of exaggerated facial features. but they do expose its true potential.]_  
  
[Zhang et al.](https://arxiv.org/abs/2007.12494) on the other hand proposed an automatic landmark detection, occlusion-aware multi-view synthesis method then 3D face restoration for caricatures.   
It aims to solve depth ambiguity and occlusion loss, which it does through creating a relationship between 2D images and 3D images using a covisiblity map that stores the mask of covisible pixels for
each target-source view pair to solve self-occlusion which is somewhat similar to NextFace's weightDiffuseSymmetryReg.  
It also introduced novel loss functions for multi-view consistency,   
It uses A ResNet model for encoding the input image to a latent space, and a decoder with a fully connected layer to generate 3D landmarks on the caricature.  
Although it looks inferior to many other methods with regards to quality, the covisibility map is where its true potential lies.  

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Multi-view.png){: width="640" height="363" } 
_[Very nice.]_  
    
[Thies et al.](/arxiv.org/abs/1912.05566) is an audio-driven facial reenactment framework driven by a deep neural network that outputs a photo-realisitic video of the target in sync with the audio a latent 3D face model space.  
It uses DeepSpeech recurrent neural networks using the latent 3D model space and uses Audio2ExpressionNet for converting the input audio to a particular facial expression.  
  
![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/TTS.gif){: width="640" height="363" } 
_[Example output]_
  
[Li et al.](https://arxiv.org/abs/1812.07741) proposed SymmFCNet, a symmetry consistent convolutional neural network for reconstructing missing pixels on one half of the face using the other half.  
SymmFCNet consisted of illumination-reweighted warping and generative reconstruction subnet.  
The dependency on multiple networks is a significant drawback in speed and memory usage.

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Symfccnet.png){: width="640" height="363" } 
_[Example output]_(https://github.com/csxmli2016/SymmFCNet)

## NextFace
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
Such attributes can be baked into each other, which is why we optimize them separately.  
In the first stage, we optimize the pose, illumination, geometry, diffuse and specular albedos, statistically regularized by the 3DMM,  
while specular roughness remains fixed.  
Using ray tracing also helps extract self shadows from interplay with face geometry.  
In the second stage, we extract unconstrained diffuse reflectance, specular reflectance and roughness that capture specific facial attributes not modeled via statistical diffuse or specular albedos.  
This staged optimization strategy adds structure and makes the under-constrained optimization problem tractable, leading to superior reconstruction vs. the naive approach.

To capture a skin's reflectance we need a light stage to model the input image's lighting conditions, That's why we construct a light stage.

### Light stage
A novel virtual light stage is formulated using differentiable ray tracing, obtaining more accurate scene illumination and reflectance, implicitly modeling self-shadows. 
The virtual light stage models the switch from point to directional area lights and vice-versa.  

![Desktop View](/assets/img/2023-10-10-3D_facial_reconstruction_from_2D_images/Light_stage.png){: width="640" height="363" } 
_[icosahedron light stage.]_(https://arxiv.org/abs/2101.05356)  
  
Such a light stage enables modeling face reflectance (diffuse, specular) and roughness reconstruction that is basically scene illumination and self-shadows aware.

By using area lights that can be turned on or off, and by controlling their intensity, position and surface area, we are capable of modeling several illumination and self-shadow scenarios.


To model incoming light on face geometry, various geometric configurations were explored such as a tetrahedron, octahedron, icosahedron and spherical – convex 3D manifolds.  

Such configurations’s triangles can be thought of as area lights, directed towards the manifold’s origin, NextFace seems to follow Green's implementation in everything.  

Triangles were used as the simplest method to model light.  

The light stage was implemented in accordance with a design philosophy that a face attribute can be acquired independently from other attributes, all from a monocular image and the face attributes can be optimized on multiple stages using the light attributes.

Icosahedron provides optimal complexity for illumination modeling.

Light is now represented as area lights derived from the face triangles of the icosahedron, such that each area light has the following independent parameters:  
- distance d<sub>j</sub> ∈ R from the face geometry (at the origin).
- relative surface area a<sub>j</sub> ∈ R.
- local position p<sub>j</sub> ∈ R<sup>2</sup> of the light center in barycentric coordinates within the face triangle.
- perceived intensity i<sub>j</sub> ∈ R<sup>3</sup>.
   
d<sub>j</sub> is the distance from the area light's origin to the face area's origin.  
a<sub>j</sub> is the surface area of the light relative to the face area's triangle, 0 being a point light, and being the maximum surface area of the triangle.  
p<sub>j</sub> is the light's center within the simplex of the face.  
i<sub>j</sub> can be set to 0 to disable a light area or larger to modify overall illumination intensity.  

I<sub>j</sub> $= {{d_j^2} \over {a_i}} i_j$  

Such a representation of light intensity decouples light parameters making $i_j$ an easily controllable parameter for changing light intensity as all the parameters are orthogonal.  

Each light $γ_j$ = { $d_j$, $a_j$, $p_j$, $i_j$ } can move according to its distance $d_j$ from the geometry's center, its size remaining proportional to $d_j, a_j$ and $p_j$ being used to control position and size of each light.  
$γ_j$ resides within the surface defined by the homothetic face, the icosahedron face scaled by $d_j$.  
Thus, the area light remains parallel to the original icosahedron’s face.  
A robust optimization strategy that extracts semantically meaningful personalized face attributes, from unconstrained images, Consequently, it reconstructs geometric patch’s reflectance separating incurred shadows   

Spherical harmonics are a set of eigenfunctions used to represent functions on the surface of a sphere, they form a complete set of orthonormal functions meaning they are completely independent.  
They are the higher dimensional analogy of the fourier series solving for periodic functions in the cartesian coordinates system.  
They solve for 3D eigenfunctions of the angular portion of laplace's equations in spherical coordinates with an assymetrical azimuth, basically solving for partial differential functions.   
They are described as harmonic because they can be expressed as a sum of circular functions as angular frequency.  
NextFace uses them here to model light projection on a face.  
They use an adaptation of such functions from the very famous paper by [Robin Green](https://3dvar.com/Green2003Spherical.pdf).  
  
The functions we use are orthonormal.  
Orthogonal polynomials are sets of polynomials that return a constant when you integrate the product of two of them.  
Orthonormal polynomials are special subset of those where it either returns a 0 or a 1.  
ِLegendre polynomials are a subset of those studied by Legendre.  
associated Legendre polynomials are a subset of those returning real numbers, they have two arguments, l and m, such that l takes any positive integer value starting 0 and m ∈ \[0,l]   

The brilliance of Green's implementation comes from exploiting the orthogonality and orthonormality attributes.

To model a surface's reflectance is normally the integration of incoming light multiplied by a transfer function depicting the surface's properties over the whole sphere.  
$_s$∫ L(s) t(s) ds
If we project both the illumination and transfer functions into SH coefficients then orthogonality guarantees that the integral of the function’s products is the same as the dot product of their coefficients.  
$$ _s∫ L(s) t(s) ds = \sum _{i=0} ^{n^2} L_i t_i $$

SH coefficients are merely the constant part mutliplied by the spherical functions to approximate the points to be constructed by the true functions   
Spherical functions project (associated) Legendre polynomials unto points across a sphere

Spherical functions or from a functional point of view, the basis funtions return the point sample of a Spherical Harmonic basis function.  
Basis functions are a set of functions that can be "stacked" to approximate a function, such functions can be used to approximate a graph or in our case, get approximate results at less computational time.  
  
The idea is that to get an approximation of reflection functions, we take the sum of basis coefficients multiplied by basis functions over many samples.  
Basis functions serve as an approximation to the real reflection function.  
  
Points are sampled across bands, now that means that the true function will be reconstructed through a band limited approximation where band–limiting is just the process of breaking a signal into it’s component frequencies and removing frequencies higher than some threshold.  

We can mix and match SH lighting and normal lighting because light sums linearly, so we can use SH lighting for the ambient and diffuse part of a shader and add a specular term over the top.  
This is exactly what they do for any image, first rendering the image diffusely with SH lighting and alpha blending a pre-filtered specular environment map over the top.  

Polynomials, though simple and familiar, are actually not that flexible, and consequently they have been more or less replaced by a basis system called the spline basis system.    

When we discuss the code 

we introduce our optimization formulation that relies on differentiable ray tracing for image synthesis. By varying the number of ray-bounces against scene geometries and subsequent indirect illumination, self-shadows can be
modeled.

In-order to use unconstrained monocular images, statistical priors have been introduced. Such priors add structure to the reconstruction formulation.


While such approaches lead to highly accurate skin (diffuse and specular) reflectance modeling, 

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
