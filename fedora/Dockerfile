FROM fedora:36

# Install required packages
RUN dnf install -y \
  wget \
  cmake \
  gfortran \
  openmpi-devel \
  netcdf-fortran-devel \
  netcdf-fortran-openmpi-devel \
  && dnf clean all \
  && rm -rf /var/cache/yum

# We will run everything else with a non-root user for extra security
RUN useradd -ms /bin/bash cosima
USER cosima
WORKDIR /home/cosima

# Download and install FMS
RUN wget -qO- https://github.com/NOAA-GFDL/FMS/archive/2022.02.tar.gz | tar xvz
RUN mkdir FMS-2022.02/build
WORKDIR FMS-2022.02/build
ENV PATH="/usr/lib64/openmpi/bin:$PATH"
RUN cmake .. -DCMAKE_INSTALL_PREFIX=/home/cosima -DCMAKE_BUILD_TYPE=Release -DCMAKE_POSITION_INDEPENDENT_CODE=ON -DCMAKE_C_COMPILER='mpicc' -DCMAKE_C_FLAGS='-O2 -ftree-vectorize -march=native -fno-math-errno -fPIC' -DCMAKE_Fortran_COMPILER='mpifort' -DCMAKE_Fortran_FLAGS='-I/usr/lib64/gfortran/modules/openmpi -O2 -ftree-vectorize -march=native -fno-math-errno -fPIC' -DCMAKE_VERBOSE_MAKEFILE=ON -D64BIT=ON -DOPENMP=ON -DFPIC=ON
RUN make -j install

# Cleanup and finish
WORKDIR /home/cosima
RUN rm -rf  FMS-2022.02
#CMD ["/bin/bash"]
