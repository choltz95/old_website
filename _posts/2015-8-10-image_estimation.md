---
title: Image Estimation with Finite Polygons
author: Chester Holtz
layout: post
permalink: /blog/post/image_estimation
categories:
  - Uncategorized
---

This project was inspired by [Richard Alsing's][1] blog post on developing a simple evolutionary algorithm to generate an image estimation of the Mona Lisa with a finite number of polygons. Provided even a naive algorithm which consisted of a population size of two - parent and child - and unformly random mutation, the generated images proved surprisingly accurate (after a number of generations had been processed). Here is the simplified algorithm implimented by Richard Alsing in C#. 

1. Generate an initial population of programs
2. Take the n best programs of the current population
3. Create Children from the best programs by mating and mutating them
4. Replace the current population with the n-best and the children programs
5. Repeat from 2 until satisfied

Recently, I have been interested in rendering graphics in C and had only completed a few tutorials for OpenGL. This project is my first independent project utilizing OpenGL and the freeGlut toolkit  to render the polygons to a window.

In terms of linear algebra, we can say an image is defined as a matrix of pixels. For the sake of simplicity, assume that the ith, jth pixel in an image can be either black or white - with the correspnding ith, jth element in the matrix being a 0 or 1. We can say there are both complex images and simple images. complex images - such as the mona lisa - are as you would expect. Simple images are matricies which represent a single polygon. Polygons are closed, have a variable number of verticies, and are shaded - ie the pixels within the bounds created by segmenting lines between verticies are black. Our goal is as follows: given a target complex image and positive numbers n and m, we would like to manufacture <= n simple images with <= m verticies such that the complex image defined as the matrix-sum of all the simple images is as close as possible to the target image. There are a number of metrics we can use to define "closeness", but the one I use is simply the standard euclidean distance formula. Instead of implimenting a polygon as a matrix, however, we consider each polygon as a k-tuple "DNA" structure containing various identifying features and let opengl handle their rendering.

DNA of each polygon is represented as a tuple containing data representing vertex coordinates, color, and transparency. Initially, for each polygon, random coordinates are chosen for a single vertex, and the coordiantes for the remaining vertecies are chosen also randomly, but bounded above and below to prevent drastically large or small polygons. Mutation then takes place and for each mutation, opengl grabs pixel data and compares it to a matrix of pixels in memory representing the target image. The algorithm itterates through each pixel applying the square of the distance function to each color value (red, blue, green) before summing the result. This value represents the pixel fitness. We accumulate a sum of these pixel fitness values for every pixel indecie of the target image to aquire the child generation fitness. We then compare the child generation fitness of the child to the parent fitness. If the fitness of the child is less than or equal to the fitness of the parent, the parent dies and the child becomes the parent before "bearing" a mutated child. Through this naive process, the estimation image evolves over time as the parent becomes more and more similar to the target. 

There is really not too much else to comment on. The code is on my [github][2]. The biggest problems I have encountered - disregarding difficulties learning to use opengl and the libpng library - are with the speed of the program. By far, the least efficient component of the program is the naive image comparison algorithm given below. Initially, I had difficulties reading pixel data from the opengl render and instead system called scrot and read the image into memory with libpng. When I switched to using OpenGl's read pixels, the reduction in the number of references to libpng functions and system calls resulted in a 3x decrease in rendering and comparing generations.

<pre class="prettyprint linenums">
//image comparison
for (y=0; y < target_image_height; y++) {
    png_byte* row1 = row_pointers_1[y];
    png_byte* row2 = row_pointers_2[y];
    for (x=0; x < target_image_width; x++) {
        png_byte* ptr1 = &(row1[x*3]);
        png_byte* ptr2 = &(row2[x*3]);
        dr = (ptr1[0] - ptr2[0]);
        dg = (ptr1[1] - ptr2[1]);
        db = (ptr1[2] - ptr2[2]);
        dr2 = dr * dr;
        dg2 = dg * dg;
        db2 = db * db;
        pixel_fitness = dr2 + dg2 + db2;
        current_fitness += pixel_fitness;
    }
}
</pre>

I think I am done for now with this project, but it would be interesting to experiment with other metrics of image similarity. I noticed github's own document similarity metric includes images. It might be fun to look up what algorithm they use. I included some useful links on the github page as starting points for reading and will add fun screenshots to this post later.

[1]: http://rogeralsing.com/2008/12/07/genetic-programming-evolution-of-mona-lisa/
[2]: https://github.com/Choltz95/image_estimation