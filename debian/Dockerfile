FROM debian:bullseye-slim

# Install required packages
RUN apt-get update && apt-get install -y --no-install-recommends \
  wget \
  make \
  cmake \
  gfortran \
  netcdf-bin \
  libnetcdf-dev \
  libnetcdff-dev \
  mpich \
  libmpich-dev \
  git \
  ca-certificates \
  pkg-config \
  && rm -rf /var/lib/apt/lists/*

# Download and install FMS
RUN wget -qO- https://github.com/NOAA-GFDL/FMS/archive/2022.02.tar.gz | tar xvz
WORKDIR FMS-2022.02/build
ENV MPI_FORTRAN_MOD_DIR="/usr/include"
RUN cmake .. -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DCMAKE_C_COMPILER="mpicc" -DCMAKE_C_FLAGS="-O2 -ftree-vectorize -march=native -fno-math-errno -fPIC" -DCMAKE_Fortran_COMPILER="mpifort" -DCMAKE_Fortran_FLAGS="-I$MPI_FORTRAN_MOD_DIR -O2 -ftree-vectorize -march=native -fno-math-errno -fPIC" -DCMAKE_VERBOSE_MAKEFILE=ON -D64BIT=ON -DOPENMP=ON -DFPIC=ON
RUN make -j install

# Cleanup
WORKDIR /
RUN rm -rf FMS-2022.02
