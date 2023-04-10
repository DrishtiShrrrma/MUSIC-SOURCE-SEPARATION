# Acoustic-DL

## Music/Audio/Voice/Speech Source Separation

Source separation for music is the task of isolating contributions, or stems, from different instruments recorded individually and arranged together to form a song. Such components include voice, bass, drums and any other accompaniments.

Source separation models either work on the spectrogram or waveform domain.

???? Unlike audio synthesis tasks that generate waveforms directly, state-of-the-art source separation methods compute masks on the magnitude spectrum. ???? State-of-the-art approaches in music source separation still operate on the spectrograms generated by the short-time Fourier transform (STFT). They produce a mask on the magnitude spectrums for each frame and each source, and the output audio is generated by running an inverse STFT on the masked spectrograms reusing the input mixture phase .


**Waveform Domain Architectures:**
- Demucs
- Hybrid Demucs
- Demucs with Transformers
- Band-split RNN
- Conv-Tasnet

## 1. [DEMUCS](https://arxiv.org/abs/1911.13254): (Deep Extractor for Music Sources)

**Motivation:** Conv-Tasnet, originally designed for monophonic speech separation and audio sampled at 8 kHz, was adapted to the task of stereophonic music source separation for audio sampled at 44.1 kHz. While Conv-Tasnet separates with a high accuracy the different sources, artifacts were observed when listening to the generated audio: a constant broadband noise, hollow instruments attacks or even missing parts. They were especially noticeable on the drums and bass sources.

**DEMUCS Key Features:**
- Waveform-to-Waveform model
- Similar to Conv-Tasnet, Demucs directly operates on the raw input waveform and generates a waveform for each source. In other words, Demucs takes a stereo mixture as input and outputs a stereo estimate for each source (C = 2).
- It is an encoder/decoder architecture composed of a **convolutional encoder, a bidirectional LSTM, and a convolutional decoder (based on wide transposed convolutions with large strides), with the encoder and decoder linked with skip U-Net connections.** 
- The other critical features of the approach are increasing the number of channels exponentially with depth, gated linear units as activation function which also allow for masking, and a new initialization scheme.
- Experiments on the MusDB dataset show that, with proper data augmentation, Demucs surpasses all state-of-the-art architecture in the waveform or spectrogram domain by at least 0.3 dB of SDR. 
- Inspired by models for music synthesis rather than masking approaches.

![image](https://user-images.githubusercontent.com/129742046/230777568-c2ba40fa-d839-4300-9ba3-f3bc29eea57d.png)

**Results:**
- While Conv-Tasnet outperforms several existing spectrogram-domain methods, it does suffer from large audio artifacts. On the other hand, with proper augmentation, the Demucs architecture surpasses all existing spectrogram or waveform domain architectures in terms of SDR, with 6.3 points of SDR without extra training data (against 6.0 for the best existing method D3Net), and up to 6.8 with extra training data.
- In fact, for the bass source, Demucs is the first model to surpass the IRM oracle, with 7.6 SDR. The pitch/tempo shift augmentation was found to be useful, which lead to a gain of 0.4 points of SDR, in particular for a model with a large number of parameters like Demucs, while it can be detrimental to Conv-TasNet.
- There is no clear winner between waveform and spectrogram domain models, as the former seems to dominate for the bass and drums sources, while the latter obtain the best performance on the vocals and other sources, as measured both by objective metrics and human evaluations.
- Spectrogram domain models have an advantage when the content is mostly harmonic and fast changing, while for sources without harmonicity (drums) or with strong and emphasized attack regimes (bass), waveform domain will better preserve the structure of the music source.
- One of the main drawbacks of the Demucs model when compared to other architecture is its large model size, more than 1014MB, against 42MB for Conv-TasNet. The size can be reduced either by reducing the initial number of channels (32 or 48 channels), which will improve both the model size, as well as reduce the computational complexity of the model, or using the DiffQ quantization technique
  - getting to 32 channels will lead to a decrease of 0.2 dB in performance.
  - Quantization reduces the model size down to 120MB without any loss of SDR, which is still more than the 42MB of Conv-Tasnet, but close to 10x improvement over the uncompressed baseline.

![image](https://user-images.githubusercontent.com/129742046/230889177-6a99c439-a213-401f-bd14-f951c9959d07.png)


#### Note:
- Batch Normalization was not used as  it was found to be detrimental to the model performance.
- During training, only small audio extracts are given, so that a quiet part or a loud part would be scaled back to an average volume. However, when using entire songs as input, it will most likely contain both quiet and loud parts. The normalization will not map both to the same volume, leading to a difference between training and evaluation.

## 2. [Hybrid DEMUCS](https://arxiv.org/pdf/2111.03600v3.pdf):

Colab : [Click here](https://colab.research.google.com/drive/1dC9nVxk3V_VPjUADsnFu8EiT-xnU1tGH?usp=sharing)

**Motivation:**


**Key Features:**



 - Performs end-to-end hybrid source separation by letting the model decide which domain is best suited for each source, and even combining both. 
 - This architecture **comes with additional improvements, such as compressed residual branches, local attention or singular value regularization, and chunked biLSTM, and most importantly, a novel hybrid spectrogram/temporal domain U-Net structure, with parallel temporal and spectrogram branches, that merge into a common core.**
 - These changes translated into strong improvements of the overall quality and absence of bleeding between sources as measured by human evaluations.
 - Won the Music Demixing Challenge 2021 organized by Sony. 

![image](https://user-images.githubusercontent.com/129742046/230909664-28aaf4e6-70c9-4bee-9691-727830d71827.png)


![image](https://user-images.githubusercontent.com/129742046/230921899-e1919cf8-977d-4845-81b4-509545b85362.png)


**Results:**
- On the MusDB HQ benchark, overall, a 1.4 dB improvement of the Signal-To-Distortion (SDR) was observed across all sources as measured on the MusDB HQ dataset, with an overall quality rated at 2.83 out of 5 (2.36 for the non hybrid Demucs), and absence of contamination at 3.04 (against 2.37 for the non hybrid Demucs).
- 

**Limitation:**
- For all its gain, one limitation of this approach is the increased complexity of the U-Net encoder/decoder, requiring careful alignmement of the temporal and spectral signals through well shaped convolutions.

## X. Conv-Tasnet:

This model reuses the masking approach of spectrogram methods but learns the masks jointly with a convolutional front-end, operating directly in the waveform domain for both the inputs and outputs. Conv-Tasnet surpasses both the IRM and IBM oracles.
