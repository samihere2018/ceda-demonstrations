# -*- makefile-gmake -*-

# Sample Makefile to use installed gkylzero library: copy and modify
# for your needs

# Obtain the git commit ID.
GIT_COMMIT_HASH := "$(shell git describe --dirty=+ --always --tags)"

ARCH_FLAGS ?= -march=native
CUDA_ARCH ?= 70
# Warning flags: -Wall -Wno-unused-variable -Wno-unused-function -Wno-missing-braces
CFLAGS ?= -O3 -g -ffast-math -fPIC -MMD -MP -DGIT_COMMIT_ID=\"$(GIT_COMMIT_HASH)\"
LDFLAGS =
PREFIX = ${PWD}/../deps/gkylsoft
SUN_PREFIX = $(shell dirname ${PREFIX})

# determine OS we are running on
UNAME = $(shell uname)

# Default lapack include and libraries: we prefer linking to static library
LAPACK_INC = $(PREFIX)/OpenBLAS/include
LAPACK_LIB_DIR = $(PREFIX)/OpenBLAS/lib
LAPACK_LIB = -lopenblas
# SuperLU includes and librararies
SUPERLU_INC = $(PREFIX)/superlu/include
ifeq ($(UNAME_S),Linux)
	SUPERLU_LIB_DIR = $(PREFIX)/superlu/lib64
	SUPERLU_LIB = $(PREFIX)/superlu/lib64/libsuperlu.a
else
	SUPERLU_LIB_DIR = $(PREFIX)/superlu/lib
	SUPERLU_LIB = $(PREFIX)/superlu/lib/libsuperlu.a
endif

# Include config.mak file (if it exists)
-include $(PREFIX)/gkylzero/share/config.mak

CFLAGS = -O3 -g -ffast-math -I. -DGIT_COMMIT_ID=\"$(GIT_COMMIT_HASH)\"

G0_INC_DIR = ${PREFIX}/gkylzero/include
G0_LIB_DIR = ${PREFIX}/gkylzero/lib
G0_LIB = -lgkylzero

SUNDIALS_INC_DIR = ${SUN_PREFIX}/sundials-install/include
SUNDIALS_LIB_DIR = ${SUN_PREFIX}/sundials-install/lib64
SUNDIALS_LIB = -lsundials_arkode -lsundials_core
SUNDIALS_RPATH = -Wl,-rpath,${SUNDIALS_LIB_DIR}

USING_NVCC =
NVCC_FLAGS =
CUDA_LIBS =
ifeq ($(CC), nvcc)
       USING_NVCC = yes
       CFLAGS = -O3 -g --forward-unknown-to-host-compiler --use_fast_math -ffast-math -MMD -MP -fPIC -DGIT_COMMIT_ID=\"$(GIT_COMMIT_HASH)\"
       NVCC_FLAGS = -x cu -dc -arch=sm_${CUDA_ARCH} --compiler-options="-fPIC"
       LDFLAGS += -arch=sm_${CUDA_ARCH}
       ifdef CUDAMATH_LIBDIR
              CUDA_LIBS = -L${CUDAMATH_LIBDIR}
       else
              CUDA_LIBS =
       endif
       CUDA_LIBS += -lcublas -lcusparse -lcusolver
endif

G0_LIBS = ${G0_LIB} ${CUDA_LIBS} -lm -lpthread
G0_RPATH = -Wl,-rpath,${G0_LIB_DIR}

# determine OS we are running on
UNAME = $(shell uname)

# On OSX we should use Accelerate framework
ifeq ($(UNAME), Darwin)
	LAPACK_INC = . # dummy
	LAPACK_LIB = -framework Accelerate
	CFLAGS += -DGKYL_USING_FRAMEWORK_ACCELERATE
endif

# Read MPI paths and flags if needed
USING_MPI =
MPI_RPATH =
MPI_INC_DIR = zero # dummy
MPI_LIB_DIR = .
ifeq (${USE_MPI}, 1)
	USING_MPI = yes
	MPI_INC_DIR = ${CONF_MPI_INC_DIR}
	MPI_LIB_DIR = ${CONF_MPI_LIB_DIR}
	MPI_RPATH = -Wl,-rpath,${CONF_MPI_LIB_DIR}
	MPI_LIBS = -lmpi
	CFLAGS += -DGKYL_HAVE_MPI
endif

# Directory for storing shared data, like ADAS
GKYL_SHARE_DIR ?= "${INSTALL_PREFIX}/gkylzero/share"
CFLAGS += -DGKYL_SHARE_DIR=$(GKYL_SHARE_DIR)

# Read NCCL paths and flags if needed (needs MPI and NVCC)
USING_NCCL =
NCCL_INC_DIR = zero # dummy
NCCL_LIB_DIR = .
ifeq (${USE_NCCL}, 1)
ifdef USING_MPI
ifdef USING_NVCC
	USING_NCCL = yes
	NCCL_INC_DIR = ${CONF_NCCL_INC_DIR}
	NCCL_LIB_DIR = ${CONF_NCCL_LIB_DIR}
	NCCL_LIBS = -lnccl
	CFLAGS += -DGKYL_HAVE_NCCL
endif
endif
endif

# Read CUDSS paths and flags if needed (needs MPI and NVCC)
USING_CUDSS =
CUDSS_INC_DIR = zero # dummy
CUDSS_LIB_DIR = .
CUDSS_RPATH =
ifeq (${USE_CUDSS}, 1)
ifdef USING_NVCC
	USING_CUDSS = yes
	CUDSS_INC_DIR = ${CONF_CUDSS_INC_DIR}
	CUDSS_LIB_DIR = ${CONF_CUDSS_LIB_DIR}
	CUDSS_RPATH = -Xlinker "-rpath,${CONF_CUDSS_LIB_DIR}"
	CUDSS_LIBS = -lcudss
	CFLAGS += -DGKYL_HAVE_CUDSS
endif
endif

# Read LUA paths and flags if needed
LUA_RPATH =
LUA_INC_DIR = zero # dummy
LUA_LIB_DIR = .
ifeq (${USE_LUA}, 1)
	USING_LUA = yes
	LUA_INC_DIR = ${CONF_LUA_INC_DIR}
	LUA_LIB_DIR = ${CONF_LUA_LIB_DIR}
	LUA_RPATH = -Wl,-rpath,${CONF_LUA_LIB_DIR}
	LUA_LIBS = -l${CONF_LUA_LIB}
	CFLAGS += -DGKYL_HAVE_LUA
endif

INCLUDES = -I${G0_INC_DIR} -I${SUNDIALS_INC_DIR} -I${LAPACK_INC} -I${SUPERLU_INC} -I${MPI_INC_DIR} -I${LUA_INC_DIR} -I${NCCL_INC_DIR} -I${CUDSS_INC_DIR}
LIB_DIRS = -L${SUNDIALS_LIB_DIR} -L${LAPACK_LIB_DIR} -L${SUPERLU_LIB_DIR} -L${MPI_LIB_DIR} -L${LUA_LIB_DIR} -L${NCCL_LIB_DIR} -L${CUDSS_LIB_DIR}
EXT_LIBS = ${LAPACK_LIB} ${SUNDIALS_LIB} ${SUNDIALS_RPATH} ${SUPERLU_LIB} ${MPI_RPATH} ${MPI_LIBS} ${LUA_RPATH} ${LUA_LIBS} ${NCCL_LIBS} ${CUDSS_RPATH} ${CUDSS_LIBS} -lm -lpthread -ldl

all: gk_diffusion_1x1v_p1

gk_diffusion_1x1v_p1: gk_diffusion_1x1v_p1.c nvector_gkylzero.o
	${CC} ${CFLAGS} ${INCLUDES} gk_diffusion_1x1v_p1.c -o gk_diffusion_1x1v_p1 -L${G0_LIB_DIR} ${G0_RPATH} ${G0_LIBS} ${LIB_DIRS} ${EXT_LIBS} nvector_gkylzero.o

nvector_gkylzero.o: nvector_gkylzero.c
	${CC} ${CFLAGS} -c ${INCLUDES} nvector_gkylzero.c -o nvector_gkylzero.o

clean:
	rm -rf gk_diffusion_1x1v_p1 gk_diffusion_1x1v_p1.dSYM *.gkyl *.o
