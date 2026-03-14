# Lensless Imaging
Arvin Shahraki

## Abstract
In this paper, I give a brief overview of lensless imaging as well as a comparison to traditional imaging. The goal is to eventually gain insights on the field of view for a large format sensor; but at the moment it is designing the small format sensor lensless camera. Then design of camera is discussed. This includes the algorithms and methods for creating the phase mask, image reconstruction, and reproducing the results from Matlab in Python as well as creating a similar, but new Python function that can create the phase masks. This discussion ends in the reveal that the Python code is functionally identical to that of Matlab except maybe some floating point error differences.



## Introduction
\IEEEPARstart{W}{hile} traditional, lensed, imaging systems are good for a large number of applications they are limited by their bulky size, cost, and weight can be limiting for medical applications, wearables, and microscopy; where lensless cameras could excel due to their small form factor, cost, and light weight. Lensless imaging is a form of computational imaging that uses a model of the entire system to create an image based on measurements from a camera sensor. The model in the case of lensless cameras includes the mask placed over the sensor, light propagation and transmission \cite{unravel}, \cite{design}. The mask acts as a consistent encoding element in the model that takes in the light from a scene, and encodes the scene information on the sensor for measurement. With the sensor measurement, the information is then decoded using the model of the system. 

In the recent past, lensless cameras were exclusively coded masks or phase masks (figure \ref{fig1}). Both coded aperture masks and phase masks rely on the point-squared-function (PSF) to reconstruct the image. The model uses the PSF to recreate how light interacts with the mask and allows the model to reconstruct the image of the scene. Coded masks modulate the light like an array of pinhole cameras: modulating phase and intensity of light. While they served their purpose, coded masks limit too much light passing through them having a negative outcome on the final image output. Phase masks alleviate this problem by relying on the phase modulation rather than intensity modulation. By designing the masks to control the phase of light passing through them, there is a considerable boost in image quality because of the higher light throughput and more granular control over phase. More recently metasurfaces have caused another split in the technology. Metasurfaces provide a newfound control over many properties of light and its interaction with the mask. While, metasurfaces are more powerful than phase masks, and even more so than coded apertures, they are out of reach for the scope of this project \cite{unravel}, \cite{design}, \cite{phlatcam}.

Through the use of phase masks, we hope to eventually gain insights on the field of view that can be attained from lensless cameras. At the moment, and for this semester, I am working on a prototype lensless camera with a small format sensor to gain insight on the phase mask design, construction of the camera, and reconstruction of the image. This paper will present the methods for which I have, and will, go about making said prototype. 



## Methods
### PhlatCam: Learning the Ropes
To learn the design and reconstruction of masks and images in lensless technologies works, my mentor asked me to rewrite the code from \cite{phlatcam} to get a sense of what is being done at each step. There are four main functions in the code database that do the following: design a phase mask, reconstruct a 2-dimensional (2D) image from the sensor capture, reconstruct a 3-dimensional image from the sensor capture, and refocus an image. Since only the functions for phase mask design and 2D reconstruction are relevant to the creation of the prototype, I will just cover those. The functions covered as well as the ones that are not are available in the github repository. 
### Design
The phase mask design starts with using 2D, Perlin noise to generate the pseudorandom basis of the phase mask. The noise then goes through Canny edge detection to create the contour plot, a series of pseudorandom curves that will make the phase mask. To create the final phase mask, a modified Gerchberg-Saxton algorithm is used; where-- instead of using Fourier transforms-- this algorithm uses the Fresnel approximation as well as the angular spectrum derived from Rayleigh-Sommerfeld propagation. Where the Fresnel propagation is:

\[ H(x,y)= e^{j\phi(\xi,\eta)}\exp \left[ j\frac{\pi}{\lambda d} \left( \left(x-\xi \right)^2+ \left(y-\eta\right)^2 \right)\right] \]

and becomes the following in the code:
\begin{equation}
    \label{eq1}
    h(x,y) = bp\exp{\left[ j2\pi \frac{z}{\lambda} * bp -\frac{1}{2}\lambda^2(x^2+y^2)\right]}
\end{equation}
That has h(x,y) as the PSF \cite{imaging}. 
As for the angular spectrum, it has two parts but the one that matters is \cite{as}, \cite{phlatcam}:
\begin{equation}
    \label{eq2}
    h(x,y) = \exp\left[j2\pi\frac{z}{\lambda}(bp)(1-\lambda^2(x^2-y^2) \right]
\end{equation}
In the original, only the angular spectrum is used. Fresnel works too on the condition that the details in the contour map are not too fine. The contour map in \cite{phlatcam} is passed back and forth through the angular spectrum refining the phase map until it has the desired PSF. 

Finally, the contour map is quantized to allow for fabrication in the future. This is done with:
\begin{equation}
    \label{eq3}
    h(x,y) = dh floor\left(\frac{\phi(x,y) \frac{\lambda}{2\pi(n-n_0)}}{dh}\right)
\end{equation}
Eq.\ref{eq3} uses the wavelength of the light from the scene, the phase map, the height to quantize to, and the difference in refractive indices to get the height map of the phase map \cite{phlatcam}, \cite{design}. This step is done to ensure that the two-photon polymerization is able to produce the final product. 
### 2-Dimensional Reconstruction
Once the phase map is designed, produced, and the camera is assembled and image can be captured and reconstructed. Because the scene depth is much larger than the thickness of the device, the reconstruction is done through Tinkhonov regularization with Wiener deconvolution \cite{phlatcam}. 
\begin{equation}
    \label{eq4}
    \hat{x}=\mathscr{F}^{-1}\left( \frac{(\mathscr{F}(p))^*\odot\mathscr{F}(b)}{\left| \mathscr{F}(p) \right|^2 + \gamma} \right)
\end{equation}
In \cite{phlatcam} there are two methods for 2D reconstruction. Using Tinkhonov regularization with Wiener deconvolution is preferably due to the considerable reduction in computational intensity.
\begin{figure}[t]
\centering
\includegraphics[width=2.5in]{LenslessTimeline.png}
\caption{History of lensless camera masks \cite{unravel}.}
\label{fig1}
\end{figure}


### Reproducing the Phase Mask Design
\noindent To reproduce the phase mask design, I started by writing a function for Perlin noise. Because Perlin noise is a widely recognized algorithm, it operates nearly identically to the function in \cite{phlatcam}. Using \cite{perlin}, I created a very similar Perlin noise algorithm in Python. There is one minor difference where \cite{phlatcam} has only 16 gradient vectors, I have integrated a customizable number of gradient vectors. The idea was to allow for smoother noise, but it appears the number of gradients has vastly diminishing returns after reaching a large enough value. 

After regularizing and resizing the noise, to fit to the specifications of the system it again passes to a nearly identical function to the one in \cite{phlatcam} for generating the phase map. The difference comes from the equation used for the Fresnel and angular spectrum calculations. Compared to eq. \ref{eq2} for the Fresnel calculation, this is the formula I've used:
\begin{equation}
    \label{eq5}
    h = (bp)\exp{\left[ j\frac{2\pi}{\lambda}z \right]} \exp{\left[ -j\pi\lambda z(x^2 +y^2) \right]}
\end{equation}
Angular spectrum equation which, compared to eq. \ref{eq2}, is:
\begin{equation}
    \label{eq6}
    h=\exp\left[ j\frac{2\pi}{\lambda}z\sqrt{1-(\lambda x)^2 -(\lambda y)^2}\right]
\end{equation}

Finally, the equation for quantizing the phase mask, eq. \ref{eq3}, and most of the other code is essentially the same as that in \cite{phlatcam}. Most of the code is very similar to what is in \cite{phlatcam} as that is one of the most important references my mentor gave me and in \cite{design}, the paper by my mentor, it is essentially following the same steps too. There are not many other ways I could go about this. It mostly took a long time because I went through the original code, line-by-line, writing down what each step was doing to the best of my ability.

## Results
### PhlatCam: Learning the Ropes
\subsubsection{Design}
\begin{figure}[h]
\centering
    \includegraphics[width=2.5in]{matlabPhaseMask.png}
\caption{Matlab phase mask from \cite{phlatcam}.}
\label{fig2}
\end{figure}

\begin{figure}[h]
\centering
    \includegraphics[width=2.5in]{matlabContour.png}
\caption{Matlab contour from \cite{phlatcam}.}
\label{fig3}
\end{figure}

\begin{figure}[h]
\centering
    \includegraphics[width=2.5in]{phaseMaskPython.png}
\caption{Python Mask Design.}
\label{fig4}
\end{figure}

Comparing fig. \ref{fig2} with the left hand side of fig. \ref{fig4}, the results are quite similar with one another. They are both quite noise in between the contours defined by the contour plots and have very similar phase profiles compared to each other based on appearance. The comparing fig. \ref{fig3} with the right hand side of fig. \ref{fig4} they are, again, both very similar to one another representing the contours defined by the Perlin noise they are generated from. Both figures also appear to have very similar fill percentages, the one I made (fig. \ref{fig4}) may have a slightly higher percentage, but nothing major. 

### 2-Dimensional Reconstruction
\begin{figure}[h]
\centering
    \includegraphics[width=2.5in]{matlabTiger.png}
\caption{Matlab simple reconstruction \cite{phlatcam}.}
\label{fig5}
\end{figure}
\begin{figure}[h]
\centering
    \includegraphics[width=2.5in]{reconPython.png}
\caption{Python simple reconstruction.}
\label{fig6}
\end{figure}

The reconstruction from Matlab (fig. \ref{fig5}) versus the Python reconstruction (fig. \ref{fig6}) are also quite similar to one another. They both have some reddish noise around the outside of the image, which does differ, but that is probably due to different floating point error differences between the Matlab and Python implementations. Besides the noise being somewhat different, the image reconstruction in the center of both images is functionally identical. 

### Reproducing the Phase Mask Design
\begin{figure}[h]
\centering
    \includegraphics[width=2.5in]{newPythonPhaseProfile.png}
\caption{Python phase mask created partially by me.}
\label{fig7}
\end{figure}
\noindent The final comparison is between the phase and contour profiles from \cite{phlatcam} (figs. \ref{fig2} and \ref{fig3}), the Python implementation (fig. \ref{fig4}), and my own Python implementation (fig. \ref{fig7}). Again, they are all very similar and functionally the same. They all have the same settings, but appear as though they would be functionally identical. The only way would be to simulate testing each one of them, but I have not yet finished writing the code to be able to do that and pass an image through each of the phase masks then reconstruct it. Appearance wise, though, they are pretty much the same.

## Discussion
\noindent Overall, the differences between the different versions of Python code I translated and wrote compared to the original Matlab code appear to be functionally identical. This means that the floating point error, specifically in reconstruction appear to be negligible and don't have a large effect on the outcome of the image. The largest issue would be for running the more complicated functions, like the 3D reconstruction, where Python begins to fall behind. When running the Python code compared to the Matlab code, it takes approximately 30\% longer. If this were to be run on a contained device it would likely be a much larger gap in performance, which could become problematic depending on the application. Since I am not running that, it will not be an issue, but if someone were to want to run such a function on a computationally limited single board computer, I would likely recommend using a language such as C++ or, if available, VDHL. It is quite a computationally demanding task to do such a thing which makes it somewhat unreasonable unless the images are sent to a separate computer for processing. Since that is not my application, it is not something I have to specifically worry about. 

My next step in continuing this work will to be simulating sending a scene through different phase masks, then decoding them to compare their performance. After that, I will begin designing the camera itself and aim to get a functional prototype by the end of this semester. If there is time, I would like to set up a small reinforcement learning unsupervised learning to see if it can produce a phase mask that has better performance than that of the contour mask; however, it is unlikely to exceed the performance of the metasurface masks mentioned in \cite{unravel}. 


## Acknowledgments
Young Wang: my mentor for this and giving me the information to work on this device.
Dr. Vivek Boominathan: for giving my mentor much of the information he learned which allowed me to work on this device and whom I cited 16 times in this paper. 
## CRediT Author Statement
\textbf{Vivek Boominathan}: Conceptualization, Methodology, Software, Visualization, Resources \textbf{Young Wang}: Methodology, Resources, Supervision \textbf{Arvin Shahraki} Software, Visualization, Validation, Writing
## Sources
% Basic information on lensless cameras
\bibitem{unravel} 
S. Li et al., {\it{Lensless camera: Unraveling the breakthroughs and prospects}}, Fundamental Research, 2024.

\bibitem{design}
J. Wang, {\it{Design, Fabrication, and Characterization of a Miniaturized Lensless Fluorescence Imaging Device with Contour Phase mask}}, Rice University, 2025. 

\bibitem{phlatcam}
V. Boominathan et al., {\it{PhlatCam: Designed Phase-Mask Based Thin Lensless Camera}}, IEEE Transactions on Pattern Analysis and Machine Intelligence, 2020.

% Different sources for information
% Angular Spectrum
\bibitem{as}
P. Christopher et al., {\it{New approaches to nonlinear diffractive field propagation}}, Acoustical Society of America, 1991.
\bibitem{fourier}
J. Goodman et al., {\it{Introduction to Fourier Optics}}, McGraw-Hill, 1998.
% Fourier Approximation and Angular Spectrum/Rayleigh-Sommerfeld approximation
\bibitem{imaging}
J. Kalkman, {\it{Advanced optical imaging}}, Delft University of Technology, 2022

\bibitem{perlin}
K. Perlin, {\it{Improved Noise reference implementation}} Computer Science NYU, https://cs.nyu.edu/$\sim$perlin/noise/, February, 2026.
\end{document}
