# Copyright (c) 1993-2015, NVIDIA CORPORATION. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#  * Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
#  * Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#  * Neither the name of NVIDIA CORPORATION nor the names of its
#    contributors may be used to endorse or promote products derived
#    from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS ``AS IS'' AND ANY
# EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
# PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE COPYRIGHT OWNER OR
# CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
# EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
# PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
# PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
# OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

# Compilers
MPICC=$(PREP) mpic++
MPILD=$(PREP) hipcc
HIPCC=$(PREP) hipcc

MPI_HOME=/usr/lib/openmpi/
HIP_PATH=/opt/rocm/hip


# Flags
CFLAGS=-O3 -march=native -Wall
MPICFLAGS=-I${MPI_HOME}/include
MPILDFLAGS=$(shell mpic++ --showme:link)
HIPFLAGS=-L$(HIP_PATH)/lib/ -lhip_hcc
HIPCFLAGS=$(shell $(HIP_PATH)/bin/hipconfig --cpp_config)

# Description of binaries
BINDIR=.
JACOBI_HIP_NORMAL_MPI=$(BINDIR)/jacobi_hip_normal_mpi
JACOBI_HIP_AWARE_MPI=$(BINDIR)/jacobi_hip_aware_mpi
BINARIES=$(JACOBI_HIP_NORMAL_MPI) $(JACOBI_HIP_AWARE_MPI)

# Commands
all: $(BINARIES)

device.o: Jacobi.h Device.cpp
	$(HIPCC) $(MPICFLAGS) -c Device.cpp -o device.o

input.o: Jacobi.h Input.c
	$(MPICC) $(MPICFLAGS) $(HIPCFLAGS) $(CFLAGS) -c Input.c -o input.o

jacobi.o: Jacobi.h Jacobi.c
	$(MPICC) $(MPICFLAGS) $(HIPCFLAGS) $(CFLAGS) -c Jacobi.c -o jacobi.o

host.o: Jacobi.h Host.c
	$(MPICC) $(MPICFLAGS) $(CFLAGS) $(HIPCFLAGS) -c Host.c -o host.o

hip_normal_mpi.o: Jacobi.h HIP_Normal_MPI.c
	$(MPICC) $(MPICFLAGS) $(CFLAGS) $(HIPCFLAGS) -c HIP_Normal_MPI.c -o hip_normal_mpi.o

hip_aware_mpi.o: Jacobi.h HIP_Aware_MPI.c
	$(MPICC) $(MPICFLAGS) $(CFLAGS) $(HIPCFLAGS) -c HIP_Aware_MPI.c -o hip_aware_mpi.o

$(JACOBI_HIP_NORMAL_MPI): jacobi.o input.o host.o device.o hip_normal_mpi.o
	$(MPILD) $(MPICFLAGS) -o $(JACOBI_HIP_NORMAL_MPI)  jacobi.o input.o host.o device.o  hip_normal_mpi.o $(MPILDFLAGS)

$(JACOBI_HIP_AWARE_MPI): jacobi.o input.o host.o device.o hip_aware_mpi.o
	$(MPILD) $(MPICFLAGS) -o $(JACOBI_HIP_AWARE_MPI) jacobi.o input.o host.o device.o hip_aware_mpi.o $(MPILDFLAGS)

clean:
	rm -rf *.o *~ $(BINARIES)
