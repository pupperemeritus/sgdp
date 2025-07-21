---
title: GSoC Project Report
slug: gsoc-project-report
description: A brief summary of my GSoC experience
longDescription: This is a brief report of my work at StingraySoftware during Google Summer of Code 2023
cardImage: "https://staging-jubilee.flickr.com/65535/49707414291_871a8a081a_q.jpg"
tags: ["code", "python", "astropy", "numpy", "stingray", "open source"]
readTime: 15
featured: True
timestamp: 2025-07-21T02:39:03+05:30
---

# Astronomy Using Unevenly Sampled Data

# About the Organization

StingraySoftware is a community developed suite of astrophysical spectral timing software

Stingray is a Python library designed to perform times series analysis and related tasks on astronomical light curves. It supports a range of commonly-used Fourier analysis techniques, as well as extensions for analyzing pulsar data, simulating data sets, and statistical modelling. Stingray is designed to be easy to extend, and easy to incorporate into data analysis workflows and pipelines.

# About the Project

In astronomy, it is common for data to be imperfect and unevenly sampled due to various observational factors such as subject obscuration, instrumental limitations, and transmission issues. In X-ray astronomy, objects of interest often include high-energy events like Gamma Ray Bursts, Pulsar/Magnetar flares, Stellar flares, and X-ray binaries. Some of these signals, especially those from pulsar/magnetar jets, exhibit highly periodic behavior, with flashes occurring at intervals of just a few milliseconds.

Analyzing unevenly sampled data presents challenges, as traditional methods like the Fast Fourier Transform are unsuitable, and the Fourier Transform itself is too slow for analyzing lightcurves in practical scenarios. The Lomb-Scargle Fourier transform is a modified version of the Fourier transform designed specifically for unevenly sampled data. It addresses the irregular sampling intervals, making it a valuable tool for the analysis of astronomical data.

This project is highly complementary to Gaurav Joshi's project, ["Quasi-Periodic Oscillation Detection using Gaussian Processes"](https://gaurav17joshi.github.io/contrast/project-report/). The Lomb-Scargle method for detecting oscillations is orders of magnitude faster than the Gaussian Processes method for quasi-periodic oscillation detection. Therefore, my project can serve as a preliminary analysis tool for a wide range of unevenly sampled data, helping identify promising candidates for further in-depth analysis using Gaurav's project.

Notably, this project includes the implementation of two versions of the Lomb-Scargle algorithm: the original slower algorithm introduced in the [original paper](https://adsabs.harvard.edu/full/1982ApJ...263..835S7) and a faster algorithm introduced in [this paper](https://ui.adsabs.harvard.edu/abs/1989ApJ...338..277P/abstract).

For greater understanding please refer [Understanding the Lomb-Scargle Periodogram](https://doi.org/10.3847/1538-4365/aab766).

# Goals of the Project

- Implement the Lomb-Scargle Fourier transform method.
- Develop the Lomb-Scargle cross-spectrum class.
- Create the Lomb-Scargle power spectrum class.
- Conduct thorough testing of the above code.
- Prepare comprehensive documentation.

# Code and Submission

My code is present in this pull request [StingrayPR#737](https://github.com/StingraySoftware/stingray/pull/737)

The usage of my code can be demonstrated in these [notebooks](https://gist.github.com/pupperemeritus/cd9ed30315c36818b75730905a6dc432)

# The Current State

The project is nearly done, only things left are some more reviews by the mentors and the final merge.

# What's Completed

1. Implemented the Lomb-Scargle Fourier transform method.
2. Developed the Lomb-Scargle cross-spectrum class.
3. Created the Lomb-Scargle power spectrum class.
4. Conducted thorough testing of the code.
5. Prepared documentation notebooks for the above code.

# Future Work

1. Further simplifying and optimizing the code (e.g., implementing multithreading, optimizing memory usage, and enabling GPU utilization).

2. Enhancing documentation by adding more content and improving the clarity of existing documentation.

3. Staying updated with the latest research developments and monitoring for potential breaking changes introduced by the libraries used in the project.

# What code got merged

The whole project is set to be merged upstream in a few weeks after some more reviews and changes. Update: The project has been merged on 28th September 2023.

# Maths Behind the Project

The Lomb Scargle method uses the mean of the sampling intervals to account for the uneven sampling intervals. Lomb Scargle method is equivalently interpreted as a Fourier method as well as a least squares method. The generalized periodogram form ensures time shift invariance. Refer [Understanding the Lomb-Scargle Periodogram](https://doi.org/10.3847/1538-4365/aab766) for a clearer understanding.

This project uses the Lomb Scargle implementation based on the Fourier interpretation.

$$ P(f) = \frac{1}{2} [{ \frac{ (\sum_n{g_n\cos(2\pi f[t_n-\tau])})^2} {\sum_n\cos^2(2\pi f[t_n-\tau])} + \frac{(\sum_ng_n\sin(2\pi f[t_n-\tau]))^2} {\sum_n\sin^2(2\pi f[t_n-\tau]) }}] $$ where $$ \ \tau =\frac{1}{4\pi f} \tan^{-1} {\frac{\sum_n\sin(4\pi f_n)}{\sum_n{\cos(4\pi f_n)}}} $$

The above is further optimized by using the [optimizations introduced by William H. Press and George B. Rybicki](https://articles.adsabs.harvard.edu/pdf/1989ApJ...338..277P) which brings down the overall time complexity of the algorithm to $$ \mathcal{O}(N\log{}N) $$

The optimizations basically include the following:

- Simplifications of the following trigonometric sums if we define

  $$S_y = \sum_j y_j \sin(\omega t_j)$$

  $$C_y = \sum_j y_j \cos(\omega t_j)$$

  $$S_2 = \sum_j \sin(2 \omega t_j)$$

  $$C_2 = \sum_j \cos(2 \omega t_j)$$

- Then we can replace

  $$ \sum_j y_j \cos(\omega (t_j - \tau)) = C_y \cos(\omega \tau) + S_y \sin(\omega \tau) $$

  $$ \sum_j y_j \sin(\omega (t_j - \tau)) = S_y \sin(\omega \tau) - C_y \sin(\omega \tau) $$

  $$ \sum_j \cos^2(\omega (t_j - \tau)) = 0.5 (N + C_2 \cos(2 \omega \tau) + S_2 \sin(2 \omega \tau)) $$

  $$ \sum_j \sin^2(\omega (t_j - \tau)) = 0.5 (N - C_2 \cos(2 \omega \tau) - S_2 \sin(2 \omega \tau)) $$

- Where $$ \omega\ is\ 2 \pi f $$ and N is the number of samples.

- If the $$ t_j $$'s were evenly spaced then the above calculations could be performed by FFTs

- Therefore, we "extirpolate" the values at evenly spaced $$ t_j $$'s by using extirpolation weight which is the same as interpolation weight

- In the words of Press and Rybicki, Interpolation, as classically understood, uses several function values on a regular mesh to construct an accurate approximation at an arbitrary point. Extirpolation, just the opposite, replaces a function value at an arbitrary point by several function values on a regular mesh, doing this in such a way that sums over the mesh are an accurate approximation to sums over the original arbitrary point.

  $$ g(t) = \sum_n w_n(t) g(\hat{t}\_n) $$ where $$ w_n(t) $$ are interpolation weights and $$ \hat{t}\_n $$ is the extirpolated evenly spaced points

- Our sum of interest
  $$
  \sum_n y_n g(t_n) \approx \sum_j y_j[\sum_k(w_k(t_j) g(\hat{t}_k))] = \sum_k\sum_j h_j w_k(t_j) \equiv \sum_k \hat{h}\_kg(\hat{t}\_k)
  $$
  where
  $$
  \hat{h}\_k = \sum_j h_j w_k(t_j)
  $$

# Challenges faced and lessons learnt

1. Implementing the Lomb-Scargle Fourier transform was quite challenging to get right. It was easy to obtain SOME output, but obtaining sensible output was very difficult. Almost 1.5 months were spent on this.
2. Pivoting the class from the legacy interface to the modern interface followed by Stingray was a bit challenging. I had initially created the Lomb-Scargle classes using the legacy interface. It took some time to relearn how to structure the class in the modern interface. The legacy interface had a more monolithic structure, while the new interface, as seen in AveragedCrossspectrum and similar classes, is more modular and cleaner.
3. Writing meaningful tests has been a new experience for me since I had not written any tests for my code before. I learned a lot from this experience.
4. I gained insights into CI/CD processes for testing and deploying code. Various testing tools like pytest, codecov, and the black formatter were extensively used.
5. I also learned a lot about good Git practices to follow in complex open-source projects. There were many situations where I had to rebase my branch because critical fixes had been merged into the main branch. Furthermore, I learned about nested repositories, as the notebooks repository is a subrepository of the Stingray repository.

# Acknowledgements

I am immensely grateful to my mentors, [Daniela Huppenkothen](https://github.com/dhuppenkothen) and [Matteo Bachetti](https://github.com/matteobachetti), for their invaluable feedback and guidance. Especially, I would like to thank them for their patience and hands-on help with some parts where I encountered difficulties. Their guidance not only helped me complete my own project but also enabled me to make contributions to the library outside the scope of my project.
