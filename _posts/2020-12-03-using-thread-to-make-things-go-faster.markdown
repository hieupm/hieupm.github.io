---
layout: post
title: Using multi-threading or multi-processing to make things go faster
date: 2020-12-03 2:32:20 +0200
description:  # Add post description (optional)
fig-caption: # Add figcaption (optional)
tags: [python, tip, thread, process, parallel]
---

A company has an e-commerce website that includes a bunch of images of their products that are up for sale. There's a rebranding coming up, which means that all of these images will need to be replaced with new ones. This includes both the full-size images and the thumbnails. We have a script that creates the thumbnails based on the full-size images. But there's a lot of files to process, and our script is taking a long time to finish. It looks like it's time to take it up a notch and use something better to do the resizing. 