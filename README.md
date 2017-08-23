# Ray-Tracer

A recursive ray-tracer with a built-in scene parser for easily generating beautiful images.

## About

This ray-tracer was written as a final project for Ravi Ramamoorthi's fantastic [course](https://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x-2) on computer graphics. It was built from scratch with a strong focus on making the C++ code clear and organized. In its current form, the project consists of:

- A recursive ray-tracer.
- A scene parser that can be used to generate images from text files.
- A linear algebra API for performing operations with points, vectors, normals and affine transformation matrices.

The only external library used is the [FreeImage](https://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x-2) library (it is used to generate PNG images with the RGB values calculated by the ray-tracer).

## Description

When I started working on this project, I thought the most challenging part of it would be to translate the theory behind a ray-tracer into well-written code. Once I finished it, however, I found that using a ray-tracer is a lot more challenging that actually implementing one. Just consider some of the things that need to be specified to generate an image:

- A camera (including its position, orientation and field of view).
- Geometric shapes (including their positions, dimensions and the transformations that act on them).
- Light sources (including their positions, directions, colours and attenuations).
- Material properties.

Keeping track of all these values can become quite complex, specially as the number of geometric shapes and light sources increases. On top of that, the process of generating an image is slowed down significantly if you have to recompile your ray-tracer whenever you make changes to a scene.

It is because of these obstacles that this project includes a scene parser. With this tool, a ray-tracer can take a scene description written with simple commands, such as this one:

And turn it into this:

A scene parser makes it a lot easier to play with a ray-tracer, and it also allows users to generate animations by writing scripts. It is hard to believe how such a simple feature can enable users to produce such stunning results:

## Features

Using the ray-tracer described in this readme involves two steps. First you must write a scene description, and then you must feed it to the ray-tracer by specifying its filename in the command-line:
 ```sh
 ray-tracer.exe scene.txt
 ```

The language used to write a scene description is very simple. In terms of syntax, all you need to know is that each line of a scene description can contain a single command, and that they all follow this format:
 ```sh
 command parameter1 parameter2 ...
 ```
Where each parameter is separated by at least one space.

Below you will find descriptions for all the commands supported by the scene parser. They are separated into six different categories: Setup, Camera, Geometry, Transformations, Lights and Materials.

### 1) Setup
In a scene description, there are three things that need to be specified before anything else. The first two are the dimensions of the image that will be generated by the ray-tracer, and the third one is the filename of said image. These settings are specified with the following commands:
 ```sh
 size width height
 output filename.png
 ```
Where:
- *__width__* and *__height__* are the desired dimensions in pixels.
- *__filename__* is the filename that will be assigned to the PNG image.

### 2) Camera
A camera must be specified to define how a scene is framed. This is done with the following command:
 ```sh
 camera fromx fromy fromz atx aty atz upx upy upz fovy
 ```
Where:
- *__from__* is the point at which the camera is located.
- *__at__* is the point that the camera points to.
- *__up__* is the vector that defines which way is up.
- *__fovy__* is the field of view in the Y direction.

In the animation below, the *__from__* point is rotated along a 45° arc while the *__at__* point remains fixed.

<p align="center">
<img src="https://github.com/diegomacario/Ray-Tracer/blob/master/readme_images/pyramid_on_mars.gif"/>
 <p align="center">
  <em>A pyramid on Mars.</em>
 </p>
</p>

### 3) Geometry
Two geometric primitives are currently supported: *__spheres__* and *__triangles__*. Two doesn't sound like much, but remember you can make any shape with just triangles:

<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/stanford_dragon.png"/>
 <p align="center">
  <em>This rendering of the Stanford Dragon is made out of 100 thousand triangles (scene description from CSE167x).</em>
 </p>
</p>

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
- *__maxverts__* is the command used to define the maximum number of vertices (*__max__*) that can be used in a scene description. This value is used to allocate the size of a few data structures, so it must be specified before creating any vertices. Since it is an upper limit, it does not have to match the exact number of vertices that are actually created.
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

Additionally, the commands *__pushTransform__* and *__popTransform__* are also supported to imitate the syntax of old-school OpenGL. To better understand their use and the order in which transformations are applied, consider the following example:
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
Three types of light sources are currently supported. One of those types is singular (*__ambient light__*) while the other two (*__point lights__* and *__directional lights__*) can be created as many times as desired.

The command used to set the colour of the ambient light is:
 ```sh
 ambient r g b
 ```
Where:
- The *__RGB__* values, which can range from 0 to 1, determine the colour of the ambient light. If this command is not specified in a scene description, the colour of the ambient light defaults to (0.2, 0.2, 0.2).

In theory, ambient light exists at all points in space and is propagated with equal intensity in all directions. Based on this definition, there should only exist one ambient light in a scene. While this is true for this ray-tracer, it also allows something rather unusual: the colour of the ambient light can be modified in between the creation of geometric primitives. In other words, users can use the ambient light to give primitives a base colour. Consider the following example, in which I create four spheres and modify the colour of the ambient light before creating each one:
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

The resulting image is:
<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/four_spheres_ambient.png"/>
 <p align="center">
  <em>One ambient light shining four different colours.</em>
 </p>
</p>

This behaviour is very particular, but it is convenient in the context of a ray-tracer. Just remember that settings like the ambient light, the attenuation and the material properties are all maintained by a state machine. Once they are set, they affect all the lights and geometric primitives created afterwards. If you want different lights or primitives to have different appeareances, you need to modify these settings before creating them.

In the case of point lights, these emit light in all directions from specific points in space. Because their positions are defined, point lights can be affected by three different types of attenuation: *__constant__*, *__linear__* and *__quadratic__* attenuation. The commands used to create this type of light are:
 ```sh
 attenuation constant linear quadratic
 point posx posy posz r g b
 ```
Where:
- *__attenuation__* is the command used to set the attenuation. A point light with no attenuation has a *__constant__* coefficient of 1 and *__linear__*/*__quadratic__* coefficients of 0 (these are the default values, just as in OpenGl).
- *__point__* is the command used to create a point light at point *__pos__*. The colour of the emitted light is determined by the *__RGB__* values.

If you wanted the intensity of a point light to decrease linearly with the distance from its origin, you would set the attenuation coefficients to (0, 1, 0). If you wanted it to decrease quadratically, you would use (0, 0, 1). Note that you can also combine the different forms of attenuation and use coefficients larger than 1 to make the intensity of a point light decrease even faster.  In the animation below, a point light with quadratic attenuation is rotated along a 180° arc from right to left:

As for directional lights, these are considered to be placed infinitely far away, which is why they only emit light in a single direction and are not affected by attenuation. The command used to create this type of light is:
 ```sh
 directional dirx diry dirz r g b
 ```
Where:
- *__dir__* is the vector that defines the direction in which light is emitted, while the *__RGB__* values determine the colour of the light.

Before moving on to the material properties, consider this question: what type of light source would you use to model sunlight?

When I was first asked that question, my answer was: "Well ambient light of course! When you are outside, the sun illuminates everything around you uniformly". This seemed natural to me, but let's think about it scientifically: 

- The sun is 149.6 million kilometers away from earth. Because this distance is so large, we can think of the sun as a light source that is placed infinitely far away (at least until humanity figures out how to travel at the speed of light, in which case no distance will be too large).
- The position of the sun affects the way it illuminates objects. Things don't look the same at dawn and at noon, do they?

Now it seems a lot more natural to use a directional light! It's fun to think about things scientifically. Another fun question: why is the sky blue?

### 6) Materials
If you have ever taken a class that covered electromagnetic waves, you are probably familiar with this description of what happens when a ray of light hits a surface:

> _A part of the incident ray is absorbed by the surface, while another part is reflected off of it. If the surface is smooth, the angle of incidence is equal to the angle of reflection. If it is rough, the direction of the reflected ray depends on the micro structure of the material that makes up the surface. Some of the incident light is also transmitted into the surface, where it gets refracted and continues propagating._  

This is a very simple, yet powerful model. By specifying material properties such as indices of refraction, attenuation coefficients and reflection coefficients, among many others, it enables us to trace the path of a ray of light as it propagates through different media. It is also very flexible in terms of its applications, commonly being used as a tool in optical design. The application we are interested in, however, is image generation. So we need to ask ourselves: how can this model be adapted to produce "realistic" images on a computer? One good way to approach this question is to start with the results we wish to obtain, and work our way backwards. So let's look at what happens in nature:

> Hold an apple in front of you and think about how light interacts with it. What details depend on the position of the light sources? Do any of them depend on your position as an observer? Now try this experiment again with something shinier. Do you notice any reflections?

I think there are three visual effects that stand out in these experiments:

1. A surface that faces a light source is brighter than one that is angled with respect to the light source.
2. A shiny surface presents specular highlights.
3. A mirror surface shows the reflections of other objects.

(Note that I did not mention translucency, transparency, refraction or any other effects that involve the transmission of light into different media, because they are not supported by the ray-tracer described in this readme. Come back in a few weeks and you will hopefully find an expanded discussion!)

Let's break these visual effects down into their basic components (this will help me illustrate how to use the ray-tracer described in this readme, and it will give you an understanding of how it works):

The first one is the simplest of the three. I like to approach it by thinking about the following situation:

> Imagine you have a scene that contains a single light source and a single surface. Now let's say you choose a point on the surface. How do you determine how brightly the light source illuminates that point?

To answer this question, we need to take two parameters into account:

- The distance between the light source and the point.
- The orientation of the surface with respect to the light source at the point.

The consequences of the first parameter are already taken care of by the attenuation associated with the light source: as the distance between the light source and the point increases, the brightness of the light decreases.

The second parameter is the one that enables us to simulate the visual effect we are examining. Picture a ray of light being emitted by the light source and striking the surface at the point you chose. *__By measuring the angle between the incident ray of light and the normal of the surface at the point, we can determine how the point is illuminated__*. If the angle is equal to 0°, the normal and the ray align, which means that the surface faces the light source directly at the point. In this scenario, the point would be illuminated with the maximum brightness that the light source can provide at the given distance. If the angle is equal to 90°, the normal and the ray are perpendicular, which means that the surface is parallel to the light source at the point. In this scenario, the point would not be illuminated at all.

Measuring this angle at every point that is struck by a ray of light, and adding up the contributions of all the light sources in a scene, allows us to determine how an entire surface is illuminated.

Note that so far I have talked about varying the brightness of light sources based on the angle described above. I intentionally used the word "brightness" because it makes everything easier to visualize, but that is not how things are implemented on a computer. In the case of the ray-tracer described in this readme, instead of varying the brightness of the light sources, we vary the intensity of a particular colour, which you can define for every object you create. To set this colour, use the following command:
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

- [CSE167x](https://www.edx.org/course/computer-graphics-uc-san-diegox-cse167x-2): This EDX course is taught by Ravi Ramamoorthi. It covers the basic linear algebra concepts you need to understand computer graphics, as well as the derivations of many fundamental equations. It uses OpenGL to illustrate the concepts discussed in the lectures, and the final project consists of building what this readme describes!
- [Learn OpenGL](https://learnopengl.com/): This site is maintained by Joey de Vries. It teaches computer graphics with OpenGL, and it is full of excellent examples and diagrams!
- [Scratch a Pixel](https://www.scratchapixel.com/): This site is similar to the previous one, and it has a section that covers the basics of ray-tracing!
- [Real-Time Rendering](http://www.realtimerendering.com/): This book, written by Tomas Akenine-Möller, Eric Haines and Naty Hoffman, is absolutely invaluable. It compiles hundreds of sources and presents them with brilliant clarity. It truly astonishes with its scope.

I think building a ray-tracer is a really fun project because all the effort you put into it yields actual images that you can marvel at. Just the sheer excitement of generating your first image will keep you motivated while you learn new things! I felt elated when the ray-tracer described in this readme spat this out:

<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/first_image.png"/>
 <p align="center">
  <em>The first image generated by the ray-tracer described in this readme.</em>
 </p>
</p>

## Dedication

This last image is for all the citizens of Venezuela, who are currently fighting for their freedom against an oppressive dictatorship.

<p align="center">
<img src="https://github.com/diegomacario/Manta-Ray-Tracer/blob/master/readme_images/bandera.png"/>
</p>
