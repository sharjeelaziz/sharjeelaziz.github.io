---
layout: post
title: "Tetrahedrally Truncated Dodecahedron Clock"
modified:
categories: blog
excerpt:
tags: [raspberrypi,python,sketchup,shapeways]
image:
  feature:
date: 2015-06-21T21:40:58-05:00
---

![Tetrahedrally Truncated Dodecahedron](/images/front-view.jpg)

I am learning to draw in SketchUp these days and made myself a nice clock in the process. The clock displays local/UTC time and a time-based one-time password for a selected 2-Step verification account.

The clock consists of a 3D printed case designed in SketchUp and printed by [ShapeWays](http://shpws.me/Im4Y) in [White Strong & Flexible](https://www.shapeways.com/materials/strong-and-flexible-plastic?li=nav) material, a Raspberry Pi, LED seven segment/dot matrix displays and some chalkboard paint. The python code uses the excellent [MAX7219](https://github.com/rm-hull/max7219) driver by Richard Hull and [onetimepass](https://github.com/tadeck/onetimepass) by Tom Jaskowski.

<iframe width="560" height="315" src="https://www.youtube.com/embed/-RI2aG52GX4" frameborder="0" allowfullscreen></iframe>

The project is hosted at https://github.com/sharjeelaziz/dodecahedron-clock
