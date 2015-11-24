---
author: fredguile
comments: true
date: 2011-11-23 15:26:47+00:00
layout: post
slug: create-a-fft-analyzer-part-ii-designing-our-spectral-processor
title: 'Create a FFT Analyzer part II: designing our spectral processor'
wordpress_id: 23
categories:
- VST Plugins / Audio Units
tags:
- Audio
- C++
- Mac OS X
- XCode
---

{% img alignleft /images/blog/libs.jpeg %} We briefly introduced the FFT part of the Accelerated Framework in Part I of this tutorial.




We are now going to focus on the vDSP library and create the C++ class responsible for doing the spectral analysis work of our input samples. We want to keep it simple, with a few public methods, however we would like to perform FFT analysis on different frame sizes. So, one of our challenges is to design a circular buffer as member variable, which is a common pattern in audio programming.


<!-- more -->


### Xcode Setup




In case you don't have your development environment initialized with a clean template for building audio units, I suggest you [download the source code of this tutorial](http://github.com/fredguile/SimpleSpectrumAnalyzer), then open the C++ class called _SimpleSpectrumProcessor.h_ located in the "Sources/SpectrumAU" folder.




First of all, if you look at the "PublicUtility" folder of the CoreAudio SDK, you may notice a quite similar class called "CASpectralProcessor". Actually, you can consider our class as a reduced version of "CASpectralProcessor", more readable to my opinion, with a different buffers organization that allows the use of various FFT sizes upon time (between 1024 and 16384 frames). As common design, we will reuse the classes _CAAutoFree_ and _CAAutoArrayDelete_ so as to implement each data buffer. The former is basically an alternative to the std::auto smart pointer with the same restrictions on pointer ownership, but since it's using _malloc_ to allocate memory, it will guarantee that the allocated memory is aligned on a 16-bits boundary (what vecLib needs). The latter is a similar version using "new" for memory allocation.




So you need to include the header "PublicUtility/CAAutoDisposer.h", either in our class header, either in the project's precompiled headers section. You also need <Accelerate/Accelerate.h> for the vDSP library and the "vecLib.framework" binary added to your build settings.

``` c++ SimpleSpectrumProcessor.h
    class SimpleSpectrumProcessor {
    public:
    enum Window { Rectangular = 1, Hann = 2, Hamming = 3, Blackman = 4 };
    private:
    UInt32 mNumChannels;
    UInt32 mRingBufferCapacity;
    UInt32 mRingBufferPosRead;
    UInt32 mRingBufferPosWrite;
    UInt32 mRingBufferCount;
    UInt32 mFFTSize;
    FFTSetup mFFTSetup;
    bool mFFTSetupCreated;
    struct ChannelBuffers {
    CAAutoFree<Float32> mRingBufferData;
    CAAutoFree<Float32> mInputData;
    CAAutoFree<Float32> mSplitData;
    CAAutoFree<Float32> mOutputData;
    CAAutoFree<DSPSplitComplex> mDSPSplitComplex;
    };
    
    CAAutoArrayDelete<ChannelBuffers> mChannels;
    CAAutoFree<Float32> mWindowData;
    
    protected:
    void InitFFT(UInt32 FFTSize, UInt32 log2FFTSize, UInt32 bins);
    void ExtractRingBufferToFFTInput(UInt32 inNumFrames);
    void ApplyWindow(Window w);
    
    public:
    SimpleSpectrumProcessor();
    virtual ~SimpleSpectrumProcessor();
    void Allocate(UInt32 inNumChannels, UInt32 ringBufferCapacity);
    bool CopyInputToRingBuffer(UInt32 inNumFrames, AudioBufferList* inInput);
    bool TryFFT(UInt32 inFFTSize, Window w = Rectangular);
    CAAutoFree<Float32> GetMagnitudes(Window w, UInt32 channelSelect = 3);
    };
```



The first member variables are storing our circular buffer's capacity, fill count and two locations for reading/writing. Next, FFTSetup is holding the FFT weights array required by vDSP to perform the FFT transform. Concerning the buffers:






	
  * _mChannels_ is an array containing various auto-pointers for each buffer we need : input/output data, split buffer to hold complex numbers that we'll access using our specific pointer _mDSPSplitComplex _(we'll explain that in a few seconds),

	
  * _mWindowData _holds floating-pont numbers of the window function we'll choose to limit frequency leakage on our frequency spectrum (see [part I](http://sample-hold.github.io/2011/11/23/create-a-fft-analyzer-part-i-prerequisites-concerns-and-setup/)),

	
  * _mFFTSize _holds the FFT size, and we propose 5 values : 1024, 2048, 4096, 8192 or 16384 samples.




By using auto-pointers as member variables (instead of simple pointers),  we use the RAII ("resource acquisition is initialization") technique of C++ to free our buffers when our Audio Unit is released (or, of course, if an exception occur). That is to say, destructors for _CAAutoFree _and _CAAutoArrayDelete_ will be called automatically when our main class is released.




Here is the basic workflow of this class:






	
  * We first call _Allocate()_ to initialize the circular buffer with a capacity of 16384 samples,

	
  *  Each time our plugin render, we copy N frames to our circular buffer using _CopyInputToRingBuffer()_, then we call _TryFFT()_ to compute those data,

	
  * In case the method _TryFFT()_ returns successfully, we call _GetMagnitudes() _to obtain a floating-point array of magnitudes to display on a graph. We can get magnitudes for left/right channel separately (_channelSelect _is 1 or 2), or we can look at the average magnitudes of a stereo channel (_channelSelect _is 3).




### Using a Ring Buffer




{% img alignleft /images/blog/200px-circular_buffer-svg.png?w=150 %} Audio Units generally capture N input samples each time they are rendered, N being set by the host in which they operate. N has a value usually lower than the minimum number of  frames required for computing FFT, so before we can provide at least 1024 samples to FFT, we have those N samples stored into a ring buffer over a few cycles.




 This kind of buffer is circular: thus, it never overflows. We always keep K samples in the ring buffer, K being the buffer's capacity. That's why we maintain two_ int pointers_ :






	
  * _mRingBufferPosRead_ indicates where we must read when we extract samples to compute FFT,

	
  * _mRingBufferPosWrite _ is the same for writing into the ring buffer.




We use an "overlap-add" algorithm to fill our ring buffer, that is, we possibly split the N samples being added, on part being stored at the end of the buffer and the other being stored from the beginning index. You can look at the method _CopyInputToRingBuffer_ to get an example of such algorithm.




We also use this technique to extract N samples from the ring buffer (look at the protected method _ExtractRingBufferToFFTInput() _ method). However, we ensure that enough samples have been stored into the buffer before we call this method.




You may note that our implement isn't thread-safe, hence all calls to the _SimpleSpectrunProcessor_'s methods must be called from the same thread.





### Data packing for vDSP FFT




The library vDSP.h provides two structures for packing the N samples you pass to the FFT: _DSPComplex _and _DSPSplitComplex_. You should first read the [neat article made by Apple on data packing](http://developer.apple.com/library/ios/#documentation/Performance/Conceptual/vDSP_Programming_Guide/UsingFourierTransforms/UsingFourierTransforms.html). Here is a summary:






	
  1. Your N samples are first stored into a 16-bits aligned float-point array, called our _input buffer_,

	
  2. Whereas a real FFT would produce 2N complex numbers, the vDSP FFT truncates the result to store N/2 complex numbers in our _output buffer:_ hence the input/output buffers have the same N size,

	
  3. Prior to the FFT function, you need to reorganize your _input buffer_ by copying your N samples into a _split buffer_.




This _split buffer_ is first initialized as a 16-bits aligned floating-point array, as you may read in the protected method _InitFFT()_. It is next accessed using a _DSPSplitComplex _structure that "groups" floating-point together, for this buffer to behave as an array of N/2 complex numbers (with both real and imaginary parts).




__This is important to remember that the last mandatory step before we can compute FFT is to reorganize you _split buffer_ by calling **vDSP_ctoz**: this will "pack" floating-point numbers for the vDSP FFT by a stride of 2.




For instance,  if your input buffer is [x1, x2, x3, x4, x5, x6, x7, x8], then your split buffer will be [x1, x5, x2, x6, x3, x7, x4, x8]. After FFT, you'll get [c1.real, c1.imag, c2.real, c2.imag, c3.real, c3.imag, c4.real, c4.imag]. But, if the _DSPSplitComplex_ structure can see interleaved complex numbers like previously, our split buffer remains an aligned buffer with [c1.real, c2.real, c3.real, c1.imag, c2.imag, c3.imag, c4.imag].





### The power of SIMD




{% img alignleft /images/blog/images-1.jpeg?w=150 %} What is the plot of having all those buffers ? We could have reworked things to diminish memory footprint.




Instead, we'll use the great SIMD features of the vDSP library with no proprietary code at all!  The benefit here is to drastically reduce the computing time of a large number of samples. Furthermore, vDSP provide numerous mathematical functions that will help us achieve our sound analysis.




At first sight, there are a lot of functions. You'll soon get use to the naming conventions used by Apple to find the good one: for instance, if you are looking to a vector-scalar operation, you may search for a function named vDSP_vs[something] or vDSP_sv[something]. It you are working with 64-bits IEEE floating point numbers, you will look at the functions named vDSP_[something]D (D for double).


Here is what our SimpleSpectrumAnalyzer will do:



	
  1. First, we determine our current windowing function by simply calling one of the ready-to-use functions: **vDSP_hann_window **(Hann), **vDSP_hamm_window** (Hamming) or **vDSP_blkman_window** (Blackman)

	
  2. We multiply our N samples with the window function using **vDSP_vmul**

	
  3. As seen above, our DSPComplexSplit structure is organized by **vDSP_ctoz**

	
  4. We compute FFT with **vDSP_fft_zip **(as a naming convention, "z" stands for complex numbers),

	
  5. Magnitude of a complex number can be obtained with **vDSP_zvabs **(we could have possibly used **vDSP_zvmags**, see below)

	
  6. We next normalize our magnitudes, by dividing then by two, using **vDSP_vsdiv **(since we got N/2 complex numbers which is half of the N input samples)

	
  7. We convert magnitudes to a decibel value using **vDSP_vdbconv**

	
  8. We correct each decibel values by applying a Db correction with **vDSP_vadd**

	
  9. We could possible obtain an average value for left and right channels using **vDSP_vadd** and **vDSP_vsdiv**.




We'll leave these steps unchanged to keep this tutorial simple, though we could have tweaked things a bit. As you know, the decibel formula is given by:




{% img centered /images/blog/062fdd96385ff2ddfdb4426194c49b29.png %}




As an optimization, rather than multiplying/dividing U1 or U2, we could have left U1 unchanged, called **vDSP_vdbconv**, then take this into account when applying our dB correction. Indeed, every multiplication or division on U1 can translate into an addition or deletion on log10(U1). In the same manner, we could have used **vDSP_zvmags** instead of **vDSP_zvabs** and saved one _sqrt _operation.





### Using SimpleSpectrumProcessor in our Audio Unit




We'll wrap up this part by showing you how our SimpleSpectrumProcessor is used in the main code. Basically:






	
  * It's a member variable of the main class _SimpleSpectrum.h_ (hence all of our resources are released using the RAII technique),

	
  * We override the _AUEffectBase::Render()_ method, responsible for grabbing N samples from the audio inputs and computing FFT,

	
  * You may notice that we aren't using the class _SimpleSpectrumKernel_ at all, even if we have overridden the method _AUKernelBase::Process()_, which is a required step for our Audio Unit to be validated by the **auval** tool.


Here the snippet of the main work:

``` c++
    ...
    AudioBufferList& inputBufList = GetInput(0)->GetBufferList();
    mProcessor.CopyInputToRingBuffer(inFramesToProcess, &inputBufList);
    ...
    if(mProcessor.TryFFT(currentBlockSize, currentWindow)) {
    ...
    CAAutoFree<Float32> magnitudes = mProcessor.GetMagnitudes(currentWindow, channelSelect);
    ...
    }
```



### Conclusion




I let you examine the Apple documentation to get acquainted of the different methods you can override from the base classes _AUEffectBase _and _AUBase_. In an upcoming tutorial, we will make a better use of the _AUKernelBase_ class, but in the meantime, we shall look how we'll build a GUI to draw our spectrum data.
