#/******************************************************************************
# * Copyright (c) 2011, Duane Merrill.  All rights reserved.
# * Copyright (c) 2011-2013, NVIDIA CORPORATION.  All rights reserved.
# * 
# * Redistribution and use in source and binary forms, with or without
# * modification, are permitted provided that the following conditions are met:
# *	 * Redistributions of source code must retain the above copyright
# *	   notice, this list of conditions and the following disclaimer.
# *	 * Redistributions in binary form must reproduce the above copyright
# *	   notice, this list of conditions and the following disclaimer in the
# *	   documentation and/or other materials provided with the distribution.
# *	 * Neither the name of the NVIDIA CORPORATION nor the
# *	   names of its contributors may be used to endorse or promote products
# *	   derived from this software without specific prior written permission.
# * 
# * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# * ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# * WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# * DISCLAIMED. IN NO EVENT SHALL NVIDIA CORPORATION BE LIABLE FOR ANY
# * DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# * (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# * LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
# * ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# *
#******************************************************************************/

#-------------------------------------------------------------------------------
#
# Makefile usage
#
# make <target> [sm=<XXX,...>] [cnp=<0|1>] [force64=<0|1>] [abi=<0|1>] [open64=<0|1>] [verbose=<0|1>] [keep=<0|1>]
#
#-------------------------------------------------------------------------------
 
#-------------------------------------------------------------------------------
# Compiler and compilation platform
#-------------------------------------------------------------------------------

NVCC = "$(shell which nvcc)"
NVCC_VERSION = $(strip $(shell nvcc --version | grep release | sed 's/.*release //' |  sed 's/,.*//'))

# detect OS
OSUPPER = $(shell uname -s 2>/dev/null | tr [:lower:] [:upper:])


#-------------------------------------------------------------------------------
# Defines
#-------------------------------------------------------------------------------

# Set TEST_CNP macro for programs (default: no)
ifeq ($(cnp), 1)
	DEFINES += -DTEST_CNP=1
	CNP_SUFFIX = cnp
    NVCCFLAGS += -rdc=true -lcudadevrt
else
	DEFINES += -DTEST_CNP=0
	CNP_SUFFIX = nocnp
endif

#-------------------------------------------------------------------------------
# SM Arch
#-------------------------------------------------------------------------------

# Architecture(s) to compile for, e.g., "sm=200,300,350" (SM20 by default).  
COMMA = ,
ifdef sm
	SM_ARCH = $(subst $(COMMA),-,$(sm))
else 
    SM_ARCH = 200
endif

ifeq (350, $(findstring 350, $(SM_ARCH)))
    SM_TARGETS += -gencode=arch=compute_35,code=\"sm_35,compute_35\" 
endif
ifeq (300, $(findstring 300, $(SM_ARCH)))
    SM_TARGETS += -gencode=arch=compute_30,code=\"sm_30,compute_30\"
endif
ifeq (200, $(findstring 200, $(SM_ARCH)))
    SM_TARGETS += -gencode=arch=compute_20,code=\"sm_20,compute_20\"
endif
ifeq (130, $(findstring 130, $(SM_ARCH)))
    SM_TARGETS += -gencode=arch=compute_13,code=\"sm_13,compute_13\" 
endif
ifeq (110, $(findstring 110, $(SM_ARCH)))
    SM_TARGETS += -gencode=arch=compute_11,code=\"sm_11,compute_11\" 
endif
ifeq (100, $(findstring 100, $(SM_ARCH)))
    SM_TARGETS += -gencode=arch=compute_10,code=\"sm_10,compute_10\" 
endif


#-------------------------------------------------------------------------------
# Compiler Flags
#-------------------------------------------------------------------------------

# Default flags: verbose kernel properties (regs, smem, cmem, etc.); runtimes for compilation phases 
NVCCFLAGS += -Xptxas -v -Xcudafe -\#

ifeq (WIN_NT, $(findstring WIN_NT, $(OSUPPER)))
    # For MSVC
    # Disable excess x86 floating point precision that can lead to results being labeled incorrectly
    NVCCFLAGS += -Xcompiler /fp:strict
    # Help the compiler/linker work with huge numbers of kernels on Windows
	NVCCFLAGS += -Xcompiler /bigobj -Xcompiler /Zm500
else
    # For g++
    # Disable excess x86 floating point precision that can lead to results being labeled incorrectly
    NVCCFLAGS += -Xcompiler -ffloat-store
endif

# 32/64-bit (32-bit device pointers by default) 
ifneq ($(force64), 1)
	# 
	CPU_ARCH = -m32
	CPU_ARCH_SUFFIX = i386
else
	CPU_ARCH = -m64
	CPU_ARCH_SUFFIX = x86_64
endif

# CUDA ABI enable/disable (enabled by default) 
ifneq ($(abi), 0)
	ABI_SUFFIX = abi
else 
	NVCCFLAGS += -Xptxas -abi=no
	ABI_SUFFIX = noabi
endif

# NVVM/Open64 middle-end compiler (nvvm by default)
ifeq ($(open64), 1)
	NVCCFLAGS += -open64
	PTX_SUFFIX = open64
else 
	PTX_SUFFIX = nvvm
endif

# Verbose toolchain output from nvcc
ifeq ($(verbose), 1)
	NVCCFLAGS += -v
endif

# Keep intermediate compilation artifacts
ifeq ($(keep), 1)
	NVCCFLAGS += -keep
endif

# Suffix to append to each binary
SUFFIX = sm$(SM_ARCH)_$(PTX_SUFFIX)_$(NVCC_VERSION)_$(ABI_SUFFIX)_$(CNP_SUFFIX)_$(CPU_ARCH_SUFFIX)



#-------------------------------------------------------------------------------
# Includes
#-------------------------------------------------------------------------------

INC += -I. -I.. 



#-------------------------------------------------------------------------------
# Dependency Lists
#-------------------------------------------------------------------------------

rwildcard=$(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2) $(filter $(subst *,%,$2),$d))

DEPS =	./Makefile \
		./test_util.h \
		$(call rwildcard,../cub/,*.cuh)


#-------------------------------------------------------------------------------
# make default
#-------------------------------------------------------------------------------

default:


#-------------------------------------------------------------------------------
# make clean
#-------------------------------------------------------------------------------

clean :
	rm -f bin/*$(CPU_ARCH_SUFFIX)* 
	rm -f *.i* *.cubin *.cu.c *.cudafe* *.fatbin.c *.ptx *.hash *.cu.cpp *.o


#-------------------------------------------------------------------------------
# make test_allocator 
#-------------------------------------------------------------------------------

test_allocator: bin/test_allocator_$(SUFFIX)

bin/test_allocator_$(SUFFIX) : test_allocator.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_allocator_$(SUFFIX) test_allocator.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3
	
	
#-------------------------------------------------------------------------------
# make test_grid_barrier 
#-------------------------------------------------------------------------------

test_grid_barrier: bin/test_grid_barrier_$(SUFFIX)

bin/test_grid_barrier_$(SUFFIX) : test_grid_barrier.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_grid_barrier_$(SUFFIX) test_grid_barrier.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3  
	

#-------------------------------------------------------------------------------
# make test_block_reduce 
#-------------------------------------------------------------------------------

test_block_reduce: bin/test_block_reduce_$(SUFFIX)

bin/test_block_reduce_$(SUFFIX) : test_block_reduce.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_block_reduce_$(SUFFIX) test_block_reduce.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3  


#-------------------------------------------------------------------------------
# make test_warp_scan 
#-------------------------------------------------------------------------------

test_warp_scan: bin/test_warp_scan_$(SUFFIX)

bin/test_warp_scan_$(SUFFIX) : test_warp_scan.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_warp_scan_$(SUFFIX) test_warp_scan.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3  


#-------------------------------------------------------------------------------
# make test_warp_reduce 
#-------------------------------------------------------------------------------

test_warp_reduce: bin/test_warp_reduce_$(SUFFIX)

bin/test_warp_reduce_$(SUFFIX) : test_warp_reduce.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_warp_reduce_$(SUFFIX) test_warp_reduce.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3  


#-------------------------------------------------------------------------------
# make test_block_scan 
#-------------------------------------------------------------------------------

test_block_scan: bin/test_block_scan_$(SUFFIX)

bin/test_block_scan_$(SUFFIX) : test_block_scan.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_block_scan_$(SUFFIX) test_block_scan.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3  


#-------------------------------------------------------------------------------
# make test_block_load_store 
#-------------------------------------------------------------------------------

test_block_load_store: bin/test_block_load_store_$(SUFFIX)

bin/test_block_load_store_$(SUFFIX) : test_block_load_store.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_block_load_store_$(SUFFIX) test_block_load_store.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3
	
	
#-------------------------------------------------------------------------------
# make test_block_radix_sort 
#-------------------------------------------------------------------------------

test_block_radix_sort: bin/test_block_radix_sort_$(SUFFIX)

bin/test_block_radix_sort_$(SUFFIX) : test_block_radix_sort.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_block_radix_sort_$(SUFFIX) test_block_radix_sort.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3


#-------------------------------------------------------------------------------
# make test_block_serialize
#-------------------------------------------------------------------------------

test_block_serialize: bin/test_block_serialize_$(SUFFIX)

bin/test_block_serialize_$(SUFFIX) : test_block_serialize.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_block_serialize_$(SUFFIX) test_block_serialize.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3


#-------------------------------------------------------------------------------
# make test_device_reduce
#-------------------------------------------------------------------------------

test_device_reduce: bin/test_device_reduce_$(SUFFIX)

bin/test_device_reduce_$(SUFFIX) : test_device_reduce.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_device_reduce_$(SUFFIX) test_device_reduce.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3


#-------------------------------------------------------------------------------
# make test_device_histo_256
#-------------------------------------------------------------------------------

test_device_histo_256: bin/test_device_histo_256_$(SUFFIX)

bin/test_device_histo_256_$(SUFFIX) : test_device_histo_256.cu $(DEPS)
	mkdir -p bin
	$(NVCC) $(DEFINES) $(SM_TARGETS) -o bin/test_device_histo_256_$(SUFFIX) test_device_histo_256.cu $(NVCCFLAGS) $(CPU_ARCH) $(INC) -O3


