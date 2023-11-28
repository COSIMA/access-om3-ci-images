# Build stage with Spack pre-installed and ready to be used
FROM spack/ubuntu-jammy:v0.20.1 as builder


# What we want to install and how we want to install it
# is specified in a manifest file (spack.yaml)
RUN mkdir /opt/spack-environment \
&&  (echo "spack:" \
&&   echo "  specs:" \
&&   echo "    - cmake@3.24.4" \
&&   echo "    - fortranxml@4.1.2" \
&&   echo "    - netcdf-c@4.9.2+mpi build_system=cmake build_type=RelWithDebInfo" \
&&   echo "    - netcdf-fortran@4.6.0" \
&&   echo "    - parallelio@2.5.10 build_type=RelWithDebInfo" \
&&   echo "    - esmf@8.4.2+debug" \
&&   echo "    - fms@2021.03.01 precision=64 ~gfs_phys ~openmp ~quad_precision build_type=RelWithDebInfo" \
&&   echo "  view: /opt/view" \
&&   echo "  concretizer:" \
&&   echo "    unify: true" \
&&   echo "    targets:" \
&&   echo "      granularity: generic" \
&&   echo "  repos:" \
&&   echo "  - $spack/var/spack/repos/builtin" \
&&   echo "  - /opt/cosima" \
&&   echo "  config:" \
&&   echo "    install_tree: /opt/software") > /opt/spack-environment/spack.yaml

# Add COSIMA repo
RUN git clone --single-branch https://github.com/COSIMA/spack_packages.git /opt/cosima

# Install the software, remove unnecessary deps

RUN cd /opt/spack-environment && spack env activate . && spack install --fail-fast && spack gc -y

# Strip all the binaries
RUN find -L /opt/view/* -type f -exec readlink -f '{}' \; | \
    xargs file -i | \
    grep 'charset=binary' | \
    grep 'x-executable\|x-sharedlib' | \
    awk -F: '{print $1}' | xargs strip -s

# Modifications to the environment that are necessary to run
RUN cd /opt/spack-environment && \
    spack env activate --sh -d . >> /etc/profile.d/z10_spack_environment.sh

# Bare OS image to run the installed executables
FROM ubuntu:22.04

COPY --from=builder /opt/spack-environment /opt/spack-environment
COPY --from=builder /opt/software /opt/software
COPY --from=builder /opt/._view /opt/._view
COPY --from=builder /opt/view /opt/view
COPY --from=builder /etc/profile.d/z10_spack_environment.sh /etc/profile.d/z10_spack_environment.sh

RUN apt-get update && apt-get install -y --no-install-recommends \
  make \
  gfortran \
  git \
  libstdc++-11-dev \
  ca-certificates \
  && rm -rf /var/lib/apt/lists/*

ENV LIBRARY_PATH="/opt/view/lib"
ENV LD_LIBRARY_PATH="/opt/view/lib"

ENTRYPOINT ["/bin/bash", "--rcfile", "/etc/profile", "-l", "-c", "$*", "--" ]
CMD [ "/bin/bash" ]

