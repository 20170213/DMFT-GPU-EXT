# DMFT-GPU-EXT

By Pak Ki Henry Tsang, email: tsang@magnet.fsu.edu

Introduction

  This is a simple piece of code that extends the original IPT DMFT code created by Jaksa Vucicevic

  https://github.com/JaksaVucicevic/DMFT 

  such that the demanding part of the code is sent to GPU(s) for dramatic increase in computation efficiency. 

  So far this code only works on nvidia cards with CUDA support.


How to use

  1. Make sure the latest cuda toolkit(& driver) is installed on your system, you can download it from

      https://developer.nvidia.com/cuda-downloads
      
  2. Download libgsl-dev and install it to use the GSL integration routines, in ubuntu use the following lines
  
      sudo apt update
      sudo apt-get install libgsl-dev
      
  3. Download the original code by Jaksa, you can find it on
  
      https://github.com/JaksaVucicevic/DMFT
      
      make sure the system is correctly set-up to run the above downloaded code
      
  4. Download siam.cu from this repository, copy and paste it in the code downloaded in step 3
      under the directory DMFT/source (or DMFT-master/source), where you should be able to find
      the files siam.cpp, siam.h and so on
      
  5. In the file siam.cpp found in step 3, comment out the functions SIAM::get_Ps() and SIAM::get_SOCSigma(), the codes in
      siam.cu will replace those two functions
      
  6. Find the architecture of your graphics card, at the time of writing the following link works
  
      https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/
      
      take note of the SM_XX where xx should match the architecture.
      
  7. The simplest way to test is to modify the makefile found in DMFT/mains
      
      ...
      
      FLAGS =   -qopenmp -D_OMP $(INC) 
      
      ...
      
      LIBS = -L/usr/local/cuda/lib64 -lcuda -lcudart -lgsl -lgslcblas

      INC = -I/usr/local/cuda/include
      
      all : ...  $(OP)/SIAM_GPU.o
      $(mpiCC) $(FLAGS) ... $(OP)/SIAM_GPU.o
      
      ...
  
      # cuSIAM (GPU)
      $(OP)/SIAM_GPU.o : $(SP)/SIAM.cu $(SP)/SIAM.h
        nvcc -arch=sm_xx -ccbin icpc -Xcompiler -qopenmp -std=c++14 -c -o $@ $(SP)/SIAM.cu
        
      ...
      
      where you should substitute the xx in sm_xx to the number you found in step 6
      
  8. Use the code as if it were identical to the original code, with easily noticed speed difference.
  
  
  Note 1: If one is using consumer GPUs with slow FP64 (double precision) performance, 
          consider commenting out the FP64 functions in the downloaded siam.cu and uncomment the FP32 (single precision) ones.
          This trades some precision for a massive increase in speed. (I/O might still bottleneck application)
  
  Note 2: You can replace in siam.cu the Cauchy integral part (which is done on the CPU) by the "KramersKronig" 
          integration routine of the un-patched code. This should give a speed boost compared to the slower GSL alternative.
            
  Note 3: The interpolation function used in siam.cu is linear interpolation with binary search. In theory the
          "patched" code supports omega-grid of arbitrary spacing, and one would not be anymore restricted to the 
          "default" log-lin grid. If faster speed is desired issue and memory is plenty, one can consider "upgrading" the 
          binary search to interpolation search for further increase in speed.
          
  Note 4: This code is multi-GPU friendly. Currently the hard-cap is 8 GPUs but one can easily increase it by changing 
          the constant MAX_GPU_COUNT=8 to MAX_GPU_COUNT=16 for example.
          
  Note 5: The constant CUDA_threadcount = 250 is the default setting. Other values such as 128 or 512 can work well and only
          a benchmark could tell if it is the right choice.
          
  Note 6: One can take grid sizes far larger than default such as 100000 if hardware is powerful enough.
  
   
   
