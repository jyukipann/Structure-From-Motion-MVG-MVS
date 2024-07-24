FROM ubuntu:22.04


# Get dependencies
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y g++-12 gcc-12

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
  cmake \
  build-essential \
  graphviz \
  git \
  coinor-libclp-dev \
  libceres-dev \
  libjpeg-dev \
  libpng-dev \
  libtiff-dev \
  libxi-dev \
  libxinerama-dev \
  libxcursor-dev \
  libxxf86vm-dev; \
  apt-get autoclean && apt-get clean

# Boost
RUN apt-get -y install libboost-iostreams-dev libboost-program-options-dev libboost-system-dev libboost-serialization-dev

# CGAL
RUN apt-get -y install libcgal-dev libcgal-qt5-dev

#GLFW3 (Optional)
RUN apt-get -y install freeglut3-dev libglew-dev libglfw3-dev

# Install COLMAP dependencies
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install \
  libboost-program-options-dev \
  libboost-filesystem-dev \
  libboost-graph-dev \
  libboost-regex-dev \
  libboost-system-dev \
  libboost-test-dev \
  # libeigen3-dev \
  libsuitesparse-dev \
  libfreeimage-dev \
  libgoogle-glog-dev \
  libgflags-dev \
  libglew-dev \
  qtbase5-dev \
  libqt5opengl5-dev

RUN DEBIAN_FRONTEND=noninteractive apt-get -y install \
  libflann-dev \
  libsqlite3-dev \
  libmetis-dev

# Build latest COLMAP
RUN git clone https://github.com/colmap/colmap.git -b release/3.10
RUN mkdir comap_build
RUN cd comap_build && cmake . ../colmap -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX="/opt" && \
    make -j4 && make install

# # OpenCV
RUN DEBIAN_FRONTEND=noninteractive apt-get install -yq libopencv-dev

# # Eigen (Known issues with eigen 3.3.7 as of 12/10/2019, so using this tested branch/commit instead) 
RUN git clone https://gitlab.com/libeigen/eigen -b 3.4
RUN mkdir eigen_build
RUN cd eigen_build && \
    cmake -DCMAKE_BUILD_TYPE=RELEASE -DCMAKE_INSTALL_PREFIX="/usr/local/include/eigen34" . ../eigen && \
    make && make install

RUN ln -s /usr/local/include/eigen34/include/eigen3 /usr/include/eigen3

# # Build latest openvMVG
RUN git clone --recursive https://github.com/openMVG/openMVG.git --branch develop
RUN mkdir openMVG_build

# VCGLib
RUN git clone https://github.com/cdcseacave/VCG.git vcglib


RUN DEBIAN_FRONTEND=noninteractive apt-get -y install python3-dev

# Build latest openMVS
RUN git clone https://github.com/cdcseacave/openMVS.git --branch develop
RUN mkdir openMVS_build

# Install cmvs-pmvs
RUN git clone https://github.com/pmoulon/CMVS-PMVS cmvs-pmvs
RUN mkdir cmvs_pmvs_build
RUN cd cmvs_pmvs_build && \
  cmake ../cmvs-pmvs/program -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt && \
  make -j4 && make install

RUN cd /openMVG_build && cmake \
  -DCMAKE_BUILD_TYPE=RELEASE \
  -DCMAKE_INSTALL_PREFIX="/opt" \
  -DOpenMVG_BUILD_TESTS=OFF \
  -DOpenMVG_BUILD_EXAMPLES=OFF \
  -DOpenMVG_BUILD_DOC=OFF \
  -DOpenMVG_BUILD_OPENGL_EXAMPLES=ON \
  -DOpenMVG_USE_OPENCV=ON \
  -DOpenMVG_USE_OCVSIFT=OFF \
  -DCOINUTILS_INCLUDE_DIR_HINTS=/usr/include \
  -DCLP_INCLUDE_DIR_HINTS=/usr/include \
  -DOSI_INCLUDE_DIR_HINTS=/usr/include \
  ../openMVG/src && \
  make -j 4 && make install && \
  cp ../openMVG/src/openMVG/exif/sensor_width_database/sensor_width_camera_database.txt /opt/bin/


RUN cd /openMVS_build && cmake \
  -DCMAKE_BUILD_TYPE=Release \
  -DVCG_ROOT=/vcglib \
  -DEIGEN3_INCLUDE_DIR=/usr/local/include/eigen34/include/eigen3 \
  -DCMAKE_INSTALL_PREFIX="/opt" \
  ../openMVS && \
  make -j4 && make install && \
  cp /openMVS/scripts/python/MvgMvsPipeline.py /opt/bin/

# Add binaries to path
ENV PATH $PATH:/opt/bin:/opt/bin/OpenMVS