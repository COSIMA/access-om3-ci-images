# Build stage with Spack pre-installed and ready to be used
FROM spack/ubuntu-jammy:v0.19.0 as builder


# What we want to install and how we want to install it
# is specified in a manifest file (spack.yaml)
RUN mkdir /opt/spack-environment \
&&  (echo "spack:" \
&&   echo "  specs:" \
&&   echo "    - cmake@3.24.2" \
&&   echo "    - netcdf-c@4.9.0+mpi" \
&&   echo "    - netcdf-fortran@4.6.0" \
&&   echo "    - parallelio@2.5.8" \
&&   echo "    - esmf@8.3.1+parallelio~pnetcdf" \
&&   echo "    - fms@2022.04" \
&&   echo "  view: /opt/view" \
&&   echo "  concretizer:" \
&&   echo "    unify: true" \
&&   echo "  config:" \
&&   echo "    install_tree: /opt/software") > /opt/spack-environment/spack.yaml

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
  && rm -rf /var/lib/apt/lists/*

ENV LIBRARY_PATH="/opt/view/lib"
ENV LD_LIBRARY_PATH="/opt/view/lib"

ENTRYPOINT ["/bin/bash", "--rcfile", "/etc/profile", "-l", "-c", "$*", "--" ]
CMD [ "/bin/bash" ]

