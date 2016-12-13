FROM ubuntu:latest

RUN apt-get update -y && \
    apt-get install -y \
      build-essential \
      wget \
      unzip

RUN sudo apt-get install cmake cmake-curses-gui -y

WORKDIR /tmp/downloads

RUN wget -O opencv-2.4.11.zip http://sourceforge.net/projects/opencvlibrary/files/opencv-unix/2.4.11/opencv-2.4.11.zip/download

RUN unzip opencv-2.4.11.zip && \
      cd opencv-2.4.11 && \
      mkdir build && \
      cd build && \
      cmake -DCMAKE_BUILD_TYPE=Release .. && \
      make -j4 && \
      sudo make install && \
      cd ../.. && \
      rm -rf opencv-2.4.11*
