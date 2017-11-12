<!---
<p align="center">
  <img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/sword_white_bg_360_frames_50_fps.gif"/>
</p>
-->

# Super-Sunshine

A ray-tracer with a simple scene description language for easily generating beautiful images.
<!---
<p align="center">
  <img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/castle.gif"/>
</p>
-->

## Summary

Super-Sunshine is a ray-tracer that can be easily interacted with. The diagram below illustrates the manner in which this interaction is meant to occur:
<!---
<p align="center">
  <img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/summary.jpg"/>
</p>
-->
As an example, let's say you wanted to render an image of three scoops of ice cream sitting in the middle of a desert. Using the scene description language developed for this project, you would start by writing a scene description like the following:

 ```sh
# Setup
size 640 480
output ice_cream.png

# Camera
camera 1.5 3.5 1.5 -0.5 0 -0.5 0 1 0 45

# Point light
attenuation 0 1 0
point 0.4 3.75 -1.6 1 1 1

# Common material
diffuse 0 0.75 0
shininess 100

# Scoops of ice cream (3 spheres)
ambient 0.122 0.541 0.439
sphere -0.6 0.375 -0.6 0.375
ambient 0.745 0.859 0.224
sphere -0.6 1.05 -0.6 0.3
ambient 0.9 0.9 0.102
sphere -0.6 1.575 -0.6 0.225

# Reflective material
specular 0 0.25 0

# Vertices
maxverts 4
vertex 0.7125 0 0.7125
vertex 0.7125 0 -1.9125
vertex -1.9125 0 0.7125
vertex -1.9125 0 -1.9125

# Desert floor (2 triangles)
ambient 0.992 0.455 0
tri 0 1 2
tri 1 3 2
 ```
 
You would then give your scene description to Super-Sunshine, which would read it and turn it into an image like the one below:
<!---
<p align="center">
<img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/ice_cream_noon.png"/>
 <p align="center">
  <em>Dessert on a desert!</em>
 </p>
</p>
-->
As you can see, the scene description language makes it easy to play with the ray-tracer. As an additional benefit, it also enables you to generate animations through scripting; it is hard to believe how such a simple feature can lead to such stunning results:
<!---
<p align="center">
<img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/flower_120_degrees.gif"/>
 <p align="center">
  <em>A very narcissistic flower.</em>
 </p>
</p>
-->
## Technical details

This project started out as a final assignment for Ravi Ramamoorthi's fantastic [course](https://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x-2) on computer graphics. Since then, it has continued to grow because it provides a great environment for experimenting with new computer graphics concepts. In its current form, the project consists of:

- A recursive ray-tracer.
- A file parser designed to read scene descriptions written with a simple scene description language.
- A linear algebra API for performing operations with points, vectors, normals and affine transformation matrices.

All the code was written in C++, with a strong focus on making it clear and organized. The only external library used is the [FreeImage](https://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x-2) library (it is used to generate PNG images with the RGB values calculated by the ray-tracer).

## Features

The scene description language developed for this project is very simple. In terms of syntax, you only need to know two things:

- Comments start with a number sign:
 ```sh
 # This is a comment.
 ```

- Commands consist of a keyword followed by a series of parameters, each separated by at least one space:
 ```sh
 keyword parameter1 parameter2 ...
 ```

Below you will find information on all the commands that are currently supported. They are separated into six different categories: *__Setup__*, *__Camera__*, *__Geometry__*, *__Transformations__*, *__Lights__* and *__Materials__*.

Note that once you are done writing a scene description, you can give it to the ray-tracer by specifying its filename as a command-line parameter:
 ```sh
 $ super_sunshine scene.txt
 ```

### 1) Setup
In a scene description, there are three parameters that must be specified before any others. The first two are the *__dimensions__* of the image that will be generated by the ray-tracer, and the third one is the *__filename__* of said image. The commands used to set these parameters are the following:
 ```sh
 size width height
 output filename.png
 ```
Where:
- *__width__* and *__height__* are the desired dimensions in pixels.
- *__filename__* is the filename that will be assigned to the PNG image.

### 2) Camera
A *__camera__* must be specified to define how a scene is framed. This is done with the following command:
 ```sh
 camera fromx fromy fromz atx aty atz upx upy upz fovy
 ```
Where:
- *__from__* is the point at which the camera is located.
- *__at__* is the point that the camera points to.
- *__up__* is the vector that defines which way is up for the camera.
- *__fovy__* is the field of view in the Y direction.

In the animation below, the *__from__* point is rotated along a 45° arc while the *__at__* point remains fixed at the center of the pyramid's base.
<!---
<p align="center">
<img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/pyramid_green_big.gif"/>
 <p align="center">
  <em>A lonely pyramid.</em>
 </p>
</p>
-->
### 3) Geometry
Two geometric primitives are currently supported: *__spheres__* and *__triangles__*. Two doesn't sound like much, but remember you can make any shape with just triangles:
<!---
<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/stanford_dragon.png"/>
 <p align="center">
  <em>This rendering of the Stanford Dragon is made out of 100 thousand triangles (scene description from CSE167x).</em>
 </p>
</p>
-->
A sphere is created using this command:
 ```sh
 sphere centerx centery centerz radius
 ```
Where:
- *__center__* is the point at which the sphere is centered.
- *__radius__* is the radius of the sphere.

In the case of a triangle, it is created as follows:
 ```sh
 maxverts max
 vertex posx posy posz
 vertex posx posy posz
 vertex posx posy posz
 tri index1 index2 index3
 ```
Where:
- *__maxverts__* is the command used to define the maximum number of vertices (*__max__*) that can be created in a scene description. This value is used by Super-Sunshine to allocate the size of a few data structures, so it must be specified before creating any vertices. Since it is an upper limit, it does not have to match the exact number of vertices that are actually created.
- *__vertex__* is the command used to create a single vertex at point *__pos__*.
- *__tri__* is the command used to create a triangle. Its three parameters are the indices of three vertices. The first vertex you create with the *__vertex__* command has index zero, and this value increases by one for each subsequent vertex you create. Note that the indices must be specified in a counterclockwise order so that the normal of the triangle points in the correct direction. Also keep in mind that different triangles can share vertices (e.g. you should be able to make a square by only creating 4 vertices and 2 triangles).

### 4) Transformations
Three basic transformations are currently supported: *__translations__*, *__rotations__* and *__scaling__*. The commands for these transformations are:
 ```sh
 translate x y z
 rotate x y z angle
 scale x y z
 ```
Where:
 - *__translate__* translates a geometric primitive *__x__*, *__y__* and *__z__* units along the X, Y and Z axes, respectively.
 - *__rotate__* rotates a geometric primitive counterclockwise by *__angle__* degrees about the vector defined by *__x__*, *__y__* and *__z__*.
 - *__scale__* scales a geometric primitive by *__x__*, *__y__* and *__z__* units along the X, Y and Z axes, respectively.
 
The image below illustrates a simple use case of these transformations:
<!---
<p align="center">
<img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/tralfamadore.png"/>
 <p align="center">
  <em>The planet Tralfamadore.</em>
 </p>
</p>
-->
You might be surprised to learn that...

- The rings are made out of spheres that were squashed along the Y axis using the scale command.
- The slight tilt of the rings was achieved by rotating them about the X and Z axes using the rotate command.
- The stars are copies of the planet that were moved very far away using the translate command.

Just as in OpenGL, these transformations right multiply the model-view matrix. This means that the last transformation specified is the first one to be applied. For example, if you wanted to:

- Create a sphere of radius 1.5 centered at the origin.
- Scale it by a factor of 2. 
- Rotate it clockwise by 90° about the Y axis.
- Translate it -5 units along the Z axis.

You would write the following:
 ```sh
 translate 0 0 -5
 rotate 0 1 0 -90
 scale 2 2 2
 sphere 0 0 0 1.5
 ```

The order of the commands might seem odd at first, but if you read them from the bottom to the top they match the verbal description of what we wanted to achieve. So if you are ever confused about the order in which transformations apply to a specific geometric primitive, you can always rely on this rule of thumb: read from the command that creates the geometric primitive to the beginning of the scene description, and apply transformations as you run into them. Also keep in mind that the order in which transformations are specified does matter: rotating and then translating is not the same as translating and then rotating.

Additionally, the commands *__pushTransform__* and *__popTransform__* are also supported. These two commands imitate the syntax and functionality of their counterparts in old-school OpenGL, allowing you to apply transformations to specific geometric primitives without affecting others. To better understand their use and the order in which transformations are applied, consider the following example:
 ```sh
 translate 1 0 0
 
 pushTransform
    rotate 0 1 0 45
    sphere 0 0 0 1
 popTransform
 
 sphere 0 1 0 0.5
 ```
 - The sphere that is inside the push/pop block is centered at (0, 0, 0) and has a radius of 1. The first transformation applied to it is the nearest one moving towards the top. In this case it is a 45° counterclockwise rotation about the Y axis. The next transformation is a translation of 1 unit along the X axis.
 - The sphere that is outside the push/pop block is centered at (0, 1, 0) and has a radius of 0.5. Since it is not inside the push/pop block, the 45° counterclockwise rotation about the Y axis does not apply to it. Its first transformation then ends up being the translation of 1 unit along the X axis.
 
Transformations can be intimidating at first, but play around with them for a while and they will start to make sense!

### 5) Lights

<p align="center">
  <img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/luggage4.png"/>
</p>

<p align="center">
  <strong><em>Salesman</strong>: Have you thought much about luggage, Mr. Banks?</em><br />
  <strong><em>Mr. Banks</strong>: No, I never really have.</em><br />
  <strong><em>Salesman</strong>: It is the central preoccupation of my life.</em>
</p>

Replace the word *__luggage__* with the word *__light__*, and you could say that I am the salesman in [that](https://www.youtube.com/watch?v=keuhmY3tQ1A) scene. My obsession with light is so bad, in fact, that I had to rewrite this section about a dozen times, because I kept going off track talking about Maxwell’s equations. Thankfully, we do not have to worry about those equations, because a ray-tracer's approach to modeling light is very simple.

What we do have to worry about are the different types of light sources that we can have in a scene. As an experiment, look around you and take note of all the light sources that you see. Perhaps you are sitting next to a window, and sunlight is pouring in through it. Maybe there is a lamp close by with a warm light bulb. Or a few shiny surfaces that reflect the light emitted by other sources. These are all light sources with different behaviours, which is why each one is modeled differently.

The three subsections below will teach you how to create light sources like the ones I mentioned above, and they will give you some background on how they are modeled.

#### 5.1) Ambient light
Ambient light is the simplest light source available. Once included in a scene, it illuminates all geometric primitives with equal intensity, regardless of their positions or orientations in space. By doing this, it models the uniform illumination produced by rays of light that have been reflected many times.

The command used to set the colour of this light source is:
 ```sh
 ambient r g b
 ```
Where:
- The *__RGB__* values, which can range from 0 to 1, determine the colour of the ambient light.

Once the colour is set, it applies to all the geometric primitives created afterwards. Unless, of course, you change it using the *__ambient__* command again, in which case the new colour begins to apply. As an example of this, consider the following snippet, in which I create four spheres under four different ambient light colours:
 ```sh
# Left sphere (green)
ambient 0.2 0.4 0.1
sphere -0.75 0 -2 0.5

# Right sphere (yellow)
ambient 0.7 0.7 0
sphere 0.75 0 -2 0.5

# Bottom sphere (red)
ambient 0.75 0 0
sphere 0 -0.75 -2 0.5

# Top sphere (blue)
ambient 0 0.262 0.344
sphere 0 0.75 -2 0.5
 ```

The resulting image looks like this:
<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/four_spheres_ambient.png"/>
 <p align="center">
  <em>One ambient light shining four different colours.</em>
 </p>
</p>

This behaviour is very particular, but it is convenient in the context of a ray-tracer. Just remember that when you create a geometric primitive, it stores the current colour of the ambient light, just as it stores the current transformations and material properties. Also note that if you do not specify the colour of the ambient light, it defaults to (0.2, 0.2, 0.2).

#### 5.2) Point lights
A point light is a light source with two defining characteristics:

- It emits light in all directions from a specific point in space.
- The intensity of the light it emits decreases with the distance from its origin.

The second bullet point describes what is known as *__distance falloff__* or *__attenuation__*, which is a phenomenon that you see every day: objects that are close to a light source are illuminated brightly, while those that are far away are not.

In nature, the intensity of a ray of light is inversely proportional to the square of the distance from its origin, which means that its intensity decreases quadratically. In Super-Sunshine, you can choose to have no attenuation at all, or to have it act in a constant, linear or quadratic manner.

The commands used to create a point light with a specific form of attenuation are:
 ```sh
 attenuation constant linear quadratic
 point posx posy posz r g b
 ```
Where:
- *__attenuation__* is the command used to set the attenuation. A point light with no attenuation has a *__constant__* coefficient of 1 and *__linear__*/*__quadratic__* coefficients of 0 (these are the default values, just as in OpenGl).
- *__point__* is the command used to create a point light at point *__pos__*. The colour of the emitted light is determined by the *__RGB__* values.

If you wanted the intensity of the light emitted by a point light to decrease linearly with the distance from its origin, you would set the attenuation coefficients to (0, 1, 0). If you wanted it to decrease quadratically, you would use (0, 0, 1). Note that you can also combine the different forms of attenuation and use coefficients larger than 1 to make the intensity decrease even faster. Also note that you can create multiple point lights with different attenuations by changing the attenuation before creating each one.

The animation below contains two quadratically-attenuated point lights (one at the upper-left corner and the other at the lower-right corner):
<!---
<p align="center">
<img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/rupee_360_frames_50_fps.gif"/>
 <p align="center">
  <em>A green rupee from Ocarina of Time!</em>
 </p>
</p>
-->

#### 5.3) Directional lights
A directional light is commonly viewed as an unattenuated point light that has been placed infinitely far away. Because this point light is so far away, the rays of light it emits arrive parallel to each other at any given scene. This means that a directional light has one defining characteristic: it only emits light in a single direction.

The command used to create a directional light is:
 ```sh
 directional dirx diry dirz r g b
 ```
Where:
- *__dir__* is the vector that defines the direction in which light is emitted, while the *__RGB__* values determine the colour of the light.

---

Before moving on to the material properties, consider this question: what type of light source would you use to model sunlight?

When I was first asked that question, my answer was: "Well ambient light of course! When you are outside, the sun illuminates everything around you uniformly". This seemed natural to me, but let's think about it carefully: 

- The sun is 149.6 million kilometers away from earth. Because this distance is so large, we can think of the sun as a point light that has been placed infinitely far away (at least until humanity figures out how to travel at the speed of light, in which case no distance will be too large).
- The position of the sun affects the way it illuminates objects. Things do not look the same at dawn and at noon, do they?

Now it seems a lot more natural to use a directional light!

### 6) Materials
If you have ever taken a class that covered electromagnetic waves, you are probably familiar with this description of what happens when a ray of light hits a surface:

> A part of the incident ray is absorbed by the surface, while another part is reflected off of it. If the surface is smooth, the angle of incidence is equal to the angle of reflection. If it is rough, the direction of the reflected ray depends on the micro structure of the material that makes up the surface. Some of the incident light is also transmitted into the surface, where it gets refracted and continues propagating.

This is a very simple, yet powerful model. By specifying material properties such as indices of refraction, attenuation coefficients and reflection coefficients, among many others, it enables us to trace the path of a ray of light as it propagates through different media. It is also very flexible in terms of its applications, commonly being used as a tool in optical design. The application we are interested in, however, is image generation. So we need to ask ourselves: how can this model be adapted to produce "realistic" images on a computer? One good way to approach this question is to start with the results we wish to obtain, and work our way backwards. So let's look at what happens in nature:

Hold an apple in front of you and think about how light interacts with it. What details depend on the position of the light sources? Do any of them depend on your position as an observer? Now try this experiment again with something shinier. Do you notice any reflections?

I think there are three visual effects that stand out in these experiments:

1. A surface that faces a light source is brighter than one that is angled with respect to the light source.
2. A shiny surface presents specular highlights.
3. A mirror surface shows the reflections of other objects.

(Note that I did not mention translucency, transparency, refraction or any other effects that involve the transmission of light into different media, because they are not supported by Super-Sunshine yet. Come back in a few weeks and you will hopefully find an expanded discussion!)

Let's break these visual effects down into their basic components (this will help me illustrate how to use Super-Sunshine, and it will give you an understanding of how it works):

The first one is the simplest of the three. I like to approach it by thinking about the following situation:

Imagine you have a scene that contains a single light source and a single surface. Now let's say you choose a point on the surface. How do you determine how brightly the light source illuminates that point?

To answer this question, we need to take two parameters into account:

- The distance between the light source and the point.
- The orientation of the surface with respect to the light source at the point.

The consequences of the first parameter are already taken care of by the attenuation associated with the light source: as the distance between the light source and the point increases, the brightness of the light decreases.

The second parameter is the one that enables us to simulate the visual effect we are examining. Picture a ray of light being emitted by the light source and striking the surface at the point you chose. By measuring the angle between the incident ray of light and the normal of the surface at the point, we can determine how the point is illuminated. If the angle is equal to 0°, the normal and the ray align, which means that the surface faces the light source directly at the point. In this scenario, the point would be illuminated with the maximum brightness that the light source can provide at the given distance. If the angle is equal to 90°, the normal and the ray are perpendicular, which means that the surface is parallel to the light source at the point. In this scenario, the point would not be illuminated at all.

Measuring this angle at every point that is struck by a ray of light, and adding up the contributions of all the light sources in a scene, allows us to determine how an entire surface is illuminated.

Note that so far I have talked about varying the brightness of light sources based on the angle described above. I intentionally used the word "brightness" because it makes everything easier to visualize, but that is not how things are implemented on a computer. In the case of Super-Sunshine, instead of varying the brightness of the light sources, we vary the intensity of a particular colour, which you can define for every object you create. To set this colour, use the following command:
 ```sh
 diffuse r g b
 ```
Note that the name *__diffuse__* comes from the official name of the model behind this effect: *__The Lambertian or Diffuse Reflectance Model__*. Using this model we can achieve images like the ones presented below:

The second effect is a little more complicated than the first one. When I asked you to look at an apple a few minutes ago, you might have noticed that the positions of its specular highlights depended on your position as an observer. If you haven't eaten your apple yet, place it on a table and walk around it. You will see the specular highlights "slide" on its surface as you walk. This means that to simulate this effect we need to take into account the positions of the light sources and of the camera.

## Future Improvements

There is still so much that remains to be done! The more I read about computer graphics, the more I want to continue exploring this field. I really wish my job involved anything related to graphics, or at least a little linear algebra. For now I will continue reading graphics textbooks on my super long commute. If you ever see a guy programming on a bus or metro in Montreal, it is probably me, and I will probably be working on implementing:

- Anti-aliasing (you can blame the saw-like patterns you see in the images above on the lack of this feature).
- Acceleration structures (I reduced the number of operations performed for each pixel as much as I could, but to truly speed things up I must implement acceleration structures).
- Refractive materials (good old Snell's law is not going to implement itself!).
- Surfaces with interpolated normals (this feature will allow users to generate things like [RGB triangles](http://math.hws.edu/graphicsbook/c3/opengl-rgb-triangle.png)).
- Colour bleeding (it would be really cool to generate a [Cornell Box](https://upload.wikimedia.org/wikipedia/commons/2/24/Cornell_box.png)).

## Learning Resources

If you are interested in learning more about computer graphics, I recommend you get started in the following places:

- [CSE167x](https://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x-2): This EDX course is taught by Ravi Ramamoorthi. It covers the basic linear algebra concepts you need to understand computer graphics, as well as the derivations of many fundamental equations. It uses OpenGL to illustrate the concepts discussed in the lectures, and the final project consists of building a big part of what this readme describes!
- [Learn OpenGL](https://learnopengl.com/): This site is maintained by Joey de Vries. It teaches computer graphics with OpenGL, and it is full of excellent examples and diagrams!
- [Scratch a Pixel](https://www.scratchapixel.com/): This site is similar to the previous one, and it has a section that covers the basics of ray-tracing!
- [Real-Time Rendering](http://www.realtimerendering.com/): This book, written by Tomas Akenine-Möller, Eric Haines and Naty Hoffman, is absolutely invaluable. It compiles hundreds of sources and presents them with brilliant clarity. It truly astonishes with its scope.

I think building a ray-tracer is a really fun project because all the effort you put into it yields actual images that you can marvel at. Just the sheer excitement of generating your first image will keep you motivated while you learn new things! I felt elated when Super-Sunshine spat this out:
<!---
<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/first_image.png"/>
 <p align="center">
  <em>The first image generated by Super-Sunshine.</em>
 </p>
</p>
-->
## Dedication

This last image is for Venezuela and all of its citizens.
<!---
<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/bandera.png"/>
</p>
-->
