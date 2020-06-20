# ADMIRE (Aperture Domain Model Image REconstruction)

## Table of Contents
1. [Overview](#Overview)
2. [CPU Implementation Setup](#CPU-Implementation-Setup)
3. [GPU Implementation Setup](#GPU-Implementation-Setup)
4. [ADMIRE Model Generation User-Defined Parameters](#ADMIRE-Model-Generation-User-Defined-Parameters)
5. [License](#License)
6. [Acknowledgements](#Acknowledgements)

## Overview
ADMIRE (Aperture Domain Model Image REconstruction) is a model-based approach to ultrasound beamforming. The overview of the method is that ultrasound channel data is first collected and time-delayed. Following this, the short-time Fourier transform (STFT) is taken through the depth dimension for each channel, and the aperture domain data for several frequencies within each STFT window are fit using models. The frequencies that are typically fit correspond to the bandwidth of the ultrasound pulse. Each model consists of the modeled wavefronts, localized in time and frequency, that return to the aperture from different scattering locations. Linear regression with elastic-net regularization is utilized to perform the fits, and the objective function is shown below.

![objective function](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Chat%5Cbeta%7D%20%3D%20%5Cunderset%7B%5Cboldsymbol%7B%5Cbeta%7D%7D%7B%5Cmathrm%7Bargmin%7D%7D%5Cfrac%7B1%7D%7B2N%7D%5Csum_%7Bi%3D1%7D%5E%7BN%7D%20%5Cleft%28%5Cboldsymbol%7By%7D_%7Bi%7D%20-%20%5Csum_%7Bj%3D1%7D%5E%7BP%7D%20%5Cboldsymbol%7BX%7D_%7Bij%7D%5Cboldsymbol%7B%5Cbeta%7D_%7Bj%7D%5Cright%29%5E%7B2%7D%20&plus;%20%5Clambda%20%5Cleft%28%20%5Calpha%20%5Cleft%5C%7C%20%5Cboldsymbol%7B%5Cbeta%7D%20%5Cright%5C%7C_%7B1%7D%20&plus;%20%5Cfrac%7B%20%5Cleft%281%20-%20%5Calpha%20%5Cright%29%5Cleft%5C%7C%20%5Cboldsymbol%7B%5Cbeta%7D%20%5Cright%5C%7C_%7B2%7D%5E%7B2%7D%7D%7B2%7D%20%5Cright%29)

In this equation, ![N](https://latex.codecogs.com/svg.latex?N) represents the number of observations, ![P](https://latex.codecogs.com/svg.latex?P) represents the number of predictors, ![X](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7BX%7D) is the model matrix containing ![P](https://latex.codecogs.com/svg.latex?P) predictors with ![N](https://latex.codecogs.com/svg.latex?N) observations each, ![y](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7By%7D) is the vector of ![N](https://latex.codecogs.com/svg.latex?N) observations to which the model matrix is being fit, ![beta](https://latex.codecogs.com/svg.latex?%5Cboldsymbol%7B%5Cbeta%7D) is the vector of ![P](https://latex.codecogs.com/svg.latex?P) model coefficients, ![lambda](https://latex.codecogs.com/svg.latex?%5Clambda) is a scaling factor for the amount of regularization that is applied, and ![alpha](https://latex.codecogs.com/svg.latex?%5Calpha) is a factor in the range [0, 1] that provides a weighting between the L1-regularization and the L2-regularization terms. Essentially, the purpose of the model fits is to estimate how each scattering location contributes to a given set of aperture domain frequency data. Once the models are fit, the decluttered aperture domain data for each frequency is reconstructed by only using the scattering locations that do not contribute to multipath or off-axis scattering. The inverse short-time Fourier transform (ISTFT) is then taken to obtain the decluttered channel data.

This repository provides code for running ADMIRE using a CPU and running ADMIRE using a GPU. It also includes code for performing real-time imaging with the GPU implementation of ADMIRE using a Verasonics Vantage 128 ultrasound research system. Note that if you do not have a CUDA-capable GPU, then you can only use the CPU implementation.

## CPU Implementation Setup
In order to utilize the CPU implementation of ADMIRE, an available release of MATLAB is required. Moreover, a C/C++ compiler that is compatible with the installed release of MATLAB must be installed in order to compile MEX-files containing C/C++ code. The compiler compatibility can be found at https://www.mathworks.com/support/requirements/supported-compilers.html. Note that the code was evaluated on both Windows and Linux OS. For Windows, the free community edition of Microsoft Visual Studio 2017 was used as the C/C++ compiler. To download this older version, go to https://visualstudio.microsoft.com/vs/older-downloads/ and create a free Dev Essentials program account with Microsoft. When installing Microsoft Visual Studio 2017, make sure to also check the box for the VC++ 2015 toolset (the 2015 will most likely be followed by a version number). For Linux, the GNU Compiler Collection (GCC) was used as the C/C++ compiler. Once a compiler is installed, you will need to compile the ```ccd_double_precision.c``` file into a MEX-file. This file is used during the model fitting stage of ADMIRE in order to perform linear regression with elastic-net regularization using the cyclic coordinate descent optimization algorithm. Assuming the code repository is already on your system, go to the MATLAB directory that contains the repository folders and add them to your MATLAB path. Following this, go to the ```CPU_Code``` folder. For both Windows and Linux OS, type the following command into the MATLAB command prompt.

```Matlab
mex ccd_double_precision.c
```

In addition, if desired, the ```-v``` flag can be included at the end of each mex command to display compilation details. If the compilation process is successful, then a success message will be displayed in the command prompt, and a compiled MEX-file will appear in the directory. The compilation process is important, and it is recommended to recompile any time a different release of MATLAB is utilized. Note that the ```glmnet``` software package was originally used to perform the model fits, but it was replaced with a custom implementation of cyclic coordinate descent using the C programming language.

## GPU Implementation Setup 
In order to utilize the GPU implementation of ADMIRE, a CUDA-capable NVIDIA GPU along with an available release of MATLAB is required. The speedup that is obtained using the GPU implementation versus the CPU implementation can vary depending on the GPU that is used. The code was tested using an NVIDIA GeForce GTX 1080 Ti GPU, an NVIDIA GeForce GTX 2080 Ti GPU, and an NVIDIA GeForce GTX 1660 Ti laptop GPU. The MATLAB Parallel Computing Toolbox must also be installed if not already installed in order to allow for the compilation of MEX-files containing CUDA code. Moreover, a C/C++ compiler that is compatible with the installed release of MATLAB must be installed in order to compile MEX-files containing C/C++ code. The compiler compatibility can be found at https://www.mathworks.com/support/requirements/supported-compilers.html. Note that the code was evaluated on both Windows and Linux OS. For Windows, the free community edition of Microsoft Visual Studio 2017 was used as the C/C++ compiler. To download this older version, go to https://visualstudio.microsoft.com/vs/older-downloads/ and create a free Dev Essentials program account with Microsoft. When installing Microsoft Visual Studio 2017, make sure to also check the box for the VC++ 2015 toolset (the 2015 will most likely be followed by a version number). For Linux, the GNU Compiler Collection (GCC) was used as the C/C++ compiler. In addition to a C/C++ compiler, a CUDA toolkit version that is compatible with the installed release of MATLAB must be installed. To determine compatibility, refer to https://www.mathworks.com/help/parallel-computing/gpu-support-by-release.html. Once the compatibility is determined, go to https://developer.nvidia.com/cuda-toolkit-archive and install the particular CUDA toolkit version. Note that the installation process for the toolkit will also allow for the option to install a new graphics driver. If you do not desire to install a new driver, then you must ensure that your current driver supports the toolkit version that is being installed. For driver and toolkit compatability, refer to page 4 of https://docs.nvidia.com/pdf/CUDA_Compatibility.pdf. 

Before compiling the code, you should first check to see that MATLAB recognizes your GPU card. To do so, go to the command prompt and type ```gpuDevice```. If successful, the properties of the GPU will be displayed. If an error is returned, then possible causes will most likely be related to the graphics driver or the toolkit version that is installed. Once the GPU is recognized, the next step is to compile the files that contain the C/CUDA code into MEX-files. Assuming the code repository is already on your system, go to the MATLAB directory that contains the repository folders and add them to your MATLAB path. Following this, go to the ```GPU_Code``` folder. For Windows OS, type the following commands into the MATLAB command prompt.

```Matlab
mexcuda ADMIRE_GPU_curvilinear_probe_reshaped_data_type.cu -lcufft
mexcuda ADMIRE_GPU_curvilinear_probe_verasonics_RF_buffer_data_type.cu -lcufft
mexcuda ADMIRE_GPU_linear_probe_reshaped_data_type.cu -lcufft
mexcuda ADMIRE_GPU_linear_probe_Verasonics_RF_buffer_data_type.cu -lcufft
```

The same commands can be used for Linux OS, but the path to the CUDA toolkit library must also be included. This is illustrated by the following commands. Note that mexcuda might find the CUDA toolkit library even if you do not explicitly type out its path.

```Matlab
mexcuda ADMIRE_GPU_curvilinear_probe_reshaped_data_type.cu -L/usr/local/cuda-10.0/lib64 -lcufft
mexcuda ADMIRE_GPU_curvilinear_probe_verasonics_RF_buffer_data_type.cu -L/usr/local/cuda-10.0/lib64 -lcufft
mexcuda ADMIRE_GPU_linear_probe_reshaped_data_type.cu -L/usr/local/cuda-10.0/lib64 -lcufft
mexcuda ADMIRE_GPU_linear_probe_Verasonics_RF_buffer_data_type.cu -L/usr/local/cuda-10.0/lib64 -lcufft
```
Note that there might be differences in your path compared to the one shown above, such as in regards to the version of the CUDA toolkit that is being used. In addition, if desired, the ```-v``` flag can be included at the end of each mexcuda command to display compilation details. If the compilation process is successful, then it will display a success message for each compilation in the command prompt. In addition, a MEX-file should appear after each mexcuda call. The compilation process is important, and it is recommended to recompile any time a different release of MATLAB is utilized.

## ADMIRE Model Generation User-Defined Parameters
In order to apply ADMIRE to ultrasound channel data, the models that ADMIRE uses must be generated. These models are generated using the ```ADMIRE_models_generation_main.m``` script. The script requires several parameters to be specified, and they are described below.

### Hello

```params.processor_type```: This specifies whether the ADMIRE models that are generated by this script will be applied to channel data using a CPU or a GPU. The two options are either ```params.processor_type = 'CPU'``` or ```params.processor_type = 'GPU'```. Note that the models are always generated using a CPU.

```params.data_type```: This specifies the type of data that is being processed. Set ```params.data_type = 'Reshaped'``` if the channel data is of the form (Depths + t0 - 1) x Elements per Beam x Beams x Frames. In this case, t0 is the sample number that corresponds to a depth of 0. If the first sample corresponds to a depth of 0, then t0 is 1. In addition, Frames corresponds to the fact that the matrix can be 4-D, where the fourth dimension indicates the frame of the channel data. ```params.data_type = 'Reshaped'``` should be selected in cases such as when Field II data is being processed or if a Verasonics RF buffer that has already been reshaped is being processed. Now, the other option for this parameter is ```params.data_type = 'Verasonics_RF_buffer'```, and this should be used when a raw Verasonics RF data buffer is being processed. The data in this buffer is of type int16. Moreover, it is of the form Buffer Rows x Total Elements On Transducer x Frames because on the Verasonics, all of the transducer elements are used to receive for the RF data buffer. 

```params.t0```: This parameter is the axial sample index that corresponds to a depth of 0. For example, when collecting data with a Verasonics system, the first few axial samples for each beam usually need to be cropped out, and ```t0``` is the parameter that is used to do this. The axial samples corresponding to indices less than t0 will be removed. Set ```params.t0 = 1``` when the t0 is already accounted for in the data. Essentially, if ```t0``` is accounted for, then the first axial sample in the data should correspond to a depth of 0.

```params.c```: The speed of sound (m/s).

```params.num_buffer_rows```: The number of rows per frame in the Verasonics RF buffer. This parameter is not used when ```params.data_type = 'Reshaped'```.

```params.num_depths```: The number of depth samples in the channel data after t0 is accounted for.

```params.num_elements```: The number of receive elements used to obtain one beam.

```params.total_elements_on_probe```: The total number of elements that are on the transducer.

```params.num_beams```: The number of beams.

```params.f0```: The transducer center frequency (Hz).

```params.fs```: The sampling frequency (Hz).

```params.BW```: The fractional bandwidth of the transmitted ultrasound pulse. For example, a value for this might be ```params.BW = 0.6```.

```params.probe_type```: The type of transducer array that is used. The two options are either ```params.probe_type = 'Linear'``` or ```params.probe_type = 'Curvilinear'```.

```params.probe_name```: The name of the transducer array that is used (i.e., ```params.probe_name = 'L7_4'```, ```params.probe_name = 'C5_2'```, etc.). Note that this parameter can be set to any string, and the generation of the ADMIRE model matrices does not depend on it. It is only used for purposes of recordkeeping.

```params.probe_radius```: The curved transducer array radius (m). This parameter is not used when ```params.probe_type = 'Linear'```.

```params.dtheta```: The angle increment between beams (rad). This parameter is not used when ```params.probe_type = 'Linear'```.

```params.probe_pitch```: The pitch of the transducer array (m).

```params.start_depth_ADMIRE```: The start depth for applying ADMIRE (m). If the start depth is less than the minimum depth for the scan, then the start depth is set to the minimum depth automatically.

```params.end_depth_ADMIRE```: The end depth for applying ADMIRE (m). If the end depth is greater than the maximum depth for the scan, then the end depth is set to the maximum depth automatically.

```params.alpha```: The alpha value that is used to weight the L1-regularization and L2-regularization terms in elastic-net regularization. The range for this value is [0, 1], and a typical value is ```params.alpha = 0.9```.

```params.lambda_scaling_factor```: The scaling factor that is used in the calculation of lambda, which is used in elastic-net regularization. This is the same as ![c_lambda](https://latex.codecogs.com/svg.latex?c_%7B%5Clambda%7D), and a typical value is ```params.lambda_scaling_factor = 0.0189```.

```params.max_iterations```: The maximum number of iterations of cyclic coordinate descent to perform.

```params.tolerance```: The tolerance convergence criterion for cyclic coordinate descent.

```params.ICA_ADMIRE_flag```: The flag that determines whether or not ICA is applied to the ADMIRE model matrices. Set ```params.ICA_ADMIRE_flag = 0``` to not apply ICA or set ```params.ICA_ADMIRE_flag = 1``` for to apply ICA (must set ```params.ICA_ADMIRE_flag = 1``` if ```params.processor_type = 'GPU'```).

```params.channel_data_output_flag```: The flag that determines whether or not the ADMIRE processed channel data is outputted. Set ```params.channel_data_output_flag = 0``` to only have the envelope data outputted when the ADMIRE models are applied or set to ```params.channel_data_output_flag = 1``` to have both the envelope data and channel data outputted when the ADMIRE models are applied.

```params.aperture_growth_flag```: The flag that determines whether or not aperture growth is applied. Set ```params.aperture_growth_flag = 0``` to not apply aperture growth or set to ```params.aperture_growth_flag = 1``` to apply aperture growth.

```params.min_num_elements```: This parameter does not let aperture growth reduce the number of elements below the parameter's value. A typical value is ```params.min_num_elements = 16```. This parameter is not used when ```params.aperture_growth_flag = 0 ```.

```params.F_number```: The F number for applying aperture growth. This parameter is not used when ```params.aperture_growth_flag = 0 ```.

```params.models_save_path```: The path to the directory in which the generated ADMIRE models are saved.

```params.max_windows_per_set```: The number of windows to group together in one set for the short-time Fourier transform (STFT) calculation on the GPU. It is recommended to set ```params.max_windows_per_set = 17```. This parameter is not used if ```params.processor_type = 'CPU'```.

```params.pulse_multiplier```: The scaling factor that is used in the ```params.stft_window_length``` parameter calculation. It is recommended to set ```params.pulse_multiplier = 1```.

```params.cal_shift```: The calibration offset (m) that is used for the ```tau_n0``` calculation in the ```generate_modeled_signal_for_predictor.m``` function. It is recommended to set ```params.cal_shift = 7.75E-6```.

```params.distance_offset_shift```: The calibration offset (m) that is used for the distance offset in the ```generate_modeled_signal_for_predictor.m``` function. It is recommended to set ```params.distance_offset_shift = 3.85E-5```.

```params.win_tune```: The scaling factor that is multiplied by the ```half_pulse_length_distance``` variable in the ```generate_modeled_signal_for_predictor.m``` function. It is recommended to set ```params.win_tune = 1```.

```params.ellipsoid_constant_1```: The constant that is used in the ROI model ellipsoidal acceptance region calculation in the ```generate_models_for_stft_window.m``` function. It is recommended to set ```params.ellipsoid_constant_1 = 0```.

```params.ellipsoid_constant_2```: The constant that is used in the outer model ellipsoidal acceptance region calculation in the ```generate_models_for_stft_window.m``` function. It is recommended to set ```params.ellipsoid_constant_2 = 0.5E-3 + eps```.

```params.lateral_limit_offset```: The calibration offset (m) that is used in calculating the possible lateral positions for the outer model predictors in the ```generate_models_for_stft_window.m``` function. It is recommended to set ```params.lateral_limit_offset = 1E-3```.

```params.stft_window_overlap```: The fraction of samples that overlap between short-time Fourier transform (STFT) windows. You must set ```params.stft_window_overlap = 0``` in this code release.

```params.gamma_im```: The gamma parameter that is used for intentional modulation. You must set ```params.gamma_im = 0``` in this code release.

```params.stft_windowing_function```: The windowing function to use for the short-time Fourier transform (STFT). You must set ```params.stft_windowing_function = @rectwin``` in this code release.

## License
Copyright 2020 Christopher Khan (christopher.m.khan@vanderbilt.edu)

This is free software made available under the Apache License, Version 2.0. For details, refer to the [LICENSE](LICENSE) file. 

## Acknowledgements
This work was supported by NIH grants R01EB020040 and S10OD016216-01 and NAVSEA grant N0002419C4302.
