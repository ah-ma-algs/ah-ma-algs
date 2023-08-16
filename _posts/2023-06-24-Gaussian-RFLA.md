---
title: Gaussian RFLA
date: 2023-06-24 12:00:00 +0300
categories: [top_category, sub_category]
tags: [tag1, tag3]     # TAG names should always be lowercase
author: Ahmed
toc: false
---
A normal, very intuitive, method when introducing any unknown subject is to use comparison and contrast to measure a known subject against the unknown one and grow branches betwixt them.

Also, A normal introduction when discussing any CNN's receptive field is discussing a human's visual receptive field first, I believe that discussing the auditory system introduces complementary yet essential concepts to better understand a CNN's receptive field but we will start with the visual receptive field firstly.

We can start with the first comparison, which is the vertebrate eye ( such as humans ) vs cephalopods ( active marine predators )

Vertebrate eyes are formed as outgrowths of the brain while cephalopods's eyes are invaginations of the body surface.

This makes the cornea, a structural part of a human's eyes, and focusing, a result of changing the lens's shape, while the cephalopods focus through movement, mobile phones and cameras generally focus through the sensors movement as well.

in a neural network context, the receptive field is defined as the size of the region in the input that produces the feature. Basically, it is a measure of association of an output feature (of any layer) to the input region (patch).

As with everything in biology, there are naturalists and creationists on opposite sides.
Naturalists stubbornly trying hard to prove that the vertebrate's eye design is an indicative of evolution, mal-adaptation in that case, evolution, the blind, impersonal force ,which stacked one layer of adaptation induced mechanism over another until it was stuck with a basis that has to have its problems fixed instead of starting all over, which it can't do.

For some reason as well, evolution became synonymous with darwinism and darwinian evolution became the sole cause of emergence and evolution, and proponents of such a theory defend it more than darwin even did.

creationists on the other hand, struggle to prove that each and every bit of the eye is 100% where its supposed it to be, serves a purpose and could not have been made any better.

Both ways actually are very devastating to the whole process as they either lead to us only labeling the eye constituing components and treating them as blind tools with limited use only managed by something upper ( just like aristotle once thought a brain was simply a radiator to keep a heart from overheating ) or giving too much significance to some parts and getting lost in the details, missing the bigger picture.

Regardless, Most vertebrate eyes are reverted, meaning, the light sensitive part is facing opposite to the light source, while the cephalopods eyes are placed the same as anyone would work it out, the light sensitive part is placed facing the light source.

![Desktop View](/assets/img/2023-06-24-Gaussian_RFLA/1024px-Evolution_eye.png){: width="640" height="363" } 
_[Schematic of a human eye vs cephalopod](https://en.wikipedia.org/wiki/File:Evolution_eye.svg)_

 
1. Retina
1. Nerve fibers
1. Optic nerve
1. Blind spot 

The above schematic shows a simple schematic of both eyes, The cephalopods have the photoreceptors and nerves routed the intiutive way, such that the photoreceptors are facing the light source and the nerves are routed back to the brain through the optic nerve channel, they resemble back illuminated cameras which have very good low light sensitivity.  
  
Humans on the other hand, have their nerves in the path of light, just like front illuminated camera sensors shown in fig below, first light passes through the signal carrying nerves(wires in the camera's case) and then it hits the photoreceptors but contrary to cameras, our photoreceptors are facing opposite to the light source, quite an unintuitive design that would get any designer fired.

![Desktop View](/assets/img/2023-06-24-Gaussian_RFLA/pHHbYWQzvSpgJR8vcKxN3f-970-80.jpg.png){: width="970" height="546" }
_[Front vs back illuminated cameras schematics](https://www.digitalcameraworld.com/features/what-is-a-bsi-sensor-and-are-they-actually-important-bsi-sensors-explained)_

Assume an object is infront of both types of eyes,  
Light hits the object, Some of the light is absorbed, and some reflects off the object,  
The reflected light enters your eye through the transparent outer layer (cornea).    
The cornea acts as a convex lens bending the light rays toward the pupil (the dark center of the eye).  
Muscles in the iris make the pupil larger or smaller. This controls how much light hits the lens.  
The lens focuses light on the retina, the layer of tissue at the back of your eye. The retina contains two types of light-sensitive photoreceptors, called rods and cones.  
Rods and cones are tiny nerve cells that change light rays into electrical impulses. These travel through the optic nerve into the brain’s visual cortex.   
The brain processes the impulses to create an image.  
ِMost of the rest of the eye tissue is about recovery, nourishment and heat dissipation.

Those many layers that the light has to pass through change and reduce a lot of the original image's features, same as we do in preprocessing when working with a Neural network.

You can skip  to the second post if you don't like a bit more biology, gaining insight into the inner workings of your eyes can be helpful though even if not in your career.

![Desktop View](/assets/img/2023-06-24-Gaussian_RFLA/Schematic-of-the-eye-with-healthy-retina-choroid-and-retinal-pigment-epithelium-RPE.jpg){: width="763" height="429" } 
_[Schematic of a healthy retina](https://www.researchgate.net/figure/Schematic-of-the-eye-with-healthy-retina-choroid-and-retinal-pigment-epithelium-RPE_fig1_342538493)_

Continuing the human eye explanation through comparison,  
A mitigation to the light loss issue due to reversed retina, is the nerve cells bundled and patched sideways to make way for an unhindered light path directly to the fovea centralis located on the macula.  
Such a fovea is about 100 times more sensitive than the average photoreceptor on the retina.  
Vision is sharpest at such a center which enables us to read ( try centering your eyes on a point then reading outside such a point without moving your eyes), track, recognize and walk.  
Most the rest of the retina is more focused on peripheral vision and our brain can seamlessly join both into a single coherent image without us noticing any difference in the RGB, brightness, contrast, etc. domains.  

The central fovea contains neither rods nor even synapses, but only 100% cones (responsible for color vision) which have an unobstructed view of the incoming light from the outside world.  

In the fovea, there is a 2 to 1 ratio of red and green specific cone cells, respectively, whereas cones in the peripheral surrounding macula are responsible for blue light detection.

The fovea is employed for accurate vision in the direction where it is pointed. It comprises less than 1% of retinal size but takes up over 50% of the visual cortex in the brain.[12] The fovea sees only the central two degrees of the visual field, (approximately twice the width of your thumbnail at arm's length).[13][14] If an object is large and thus covers a large angle, the eyes must constantly shift their gaze to subsequently bring different portions of the image into the fovea (as in reading).

![Desktop View](/assets/img/2023-06-24-Gaussian_RFLA/AcuityHumanEye.svg.png){: width="763" height="429" } 
_[Approximation of the acuity of the Human eye, horizontal cross section](https://en.wikipedia.org/wiki/File:AcuityHumanEye.svg)_  

Emerges a problem when a cell is subjected to constant light, is photo-bleaching, which means the cell is unable to produce any current in response to light anymore and becomes damaged,  

Cone cells can adapt to light rapidly and do not become saturated under constant light as rods do. 
 
After photobleaching cones can recover their membrane current within 20 milliseconds whereas rods can require upwards of 20 minutes to recover their membrane current.  

That all explains the sole existence of cones in the fovea.  

cephalopods on the other hand record and interpret as stimuli (pattern recognition ) only light and color variations of a moving object.”27 Importantly, the octopus “will respond to certain motions as if they were prey, but will not react to his normal food-objects when they are motionless.  


The octopus eye also contains a complex nerve plexus posterior to the receptors.19 Wells adds that the optic lobes must assume many of the functions of the inverted retina in vertebrates so that the “apparent relative simplicity of the cephalopod system is an illusion. It is a matter of stacking; amacrines, bipolars and ganglion cells are all there, but stuck onto the outer layer of the optic lobe rather than onto the back of the retina.”20 


