FROM nvidia/cuda:11.5.0-cudnn8-devel-ubuntu18.04

ENV LD_LIBRARY_PATH "/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
ENV TZ "Asia/Tokyo"
ENV DEBIAN_FRONTEND=noninteractive
# ENV http_proxy "http://proxy.ac.jp:8080"
# ENV https_proxy "http://proxy.ac.jp:8080"

# 画像処理したい人向け

RUN apt-get update &&\
        apt-get -y install software-properties-common
RUN add-apt-repository -y ppa:alex-p/tesseract-ocr-devel

RUN apt-get update && apt-get -y upgrade
RUN apt-get -y install python3-dev python3-pip python3-setuptools jupyter && \
        apt-get -y install libjpeg-dev zlib1g-dev libopenblas-dev liblapack-dev && \
        apt-get -y install git cmake curl wget p7zip-full && \
        apt-get -y install build-essential gcc g++ make libtool texinfo dpkg-dev pkg-config && \
        apt-get -y install libopencv-dev libleptonica-dev && \
        apt-get -y install liblog4cplus-dev libcurl3-dev

RUN unlink /etc/localtime && ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
RUN pip3 install --upgrade numpy scipy matplotlib

### dlib and face_recognition install ### begin
RUN mkdir -p /usr/local/dlib/build
WORKDIR /usr/local/dlib
RUN git init \
        && git remote add origin https://github.com/davisking/dlib \
        && git fetch --depth 1 origin 21651f498edb167792d65c4c280007cfece84b1a \
        && git reset --hard FETCH_HEAD

WORKDIR /usr/local/dlib/build
RUN CUDA_PATH="/usr/local/cuda-11.5" CFLAGS="-I/usr/local/cuda-11.5/include" LDFLAGS="-L/usr/local/cuda-11.5/lib64" \
        cmake -DUSE_AVX_INSTRUCTIONS=YES -DDLIB_USE_CUDA=YES ..
RUN CUDA_PATH="/usr/local/cuda-11.5" CFLAGS="-I/usr/local/cuda-11.5/include" LDFLAGS="-L/usr/local/cuda-11.5/lib64" \
        cmake --build .
RUN make install 

WORKDIR /usr/local/dlib
RUN CUDA_PATH="/usr/local/cuda-11.5" CFLAGS="-I/usr/local/cuda-11.5/include" LDFLAGS="-L/usr/local/cuda-11.5/lib64" \
        python3 setup.py build \
        && python3 setup.py install 

RUN pip3 install face_recognition
### dlib and face_recognition install ### end

### OpenALPR install ### begin
RUN apt-get -y install tesseract-ocr tesseract-ocr-jpn libtesseract-dev tesseract-ocr-script-jpan tesseract-ocr-script-jpan-vert

WORKDIR /tmp
RUN git clone --depth 1 https://github.com/openalpr/openalpr.git
RUN mkdir openalpr/src/build
WORKDIR /tmp/openalpr/src/build
RUN cmake -DCMAKE_CXX_FLAGS='-fopenmp' -DCMAKE_C_FLAGS='-fopenmp' -DCMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_INSTALL_SYSCONFDIR:PATH=/etc ..
RUN make
RUN make install

WORKDIR /tmp/openalpr/src/bindings/python
RUN python3 setup.py install 
### OpenALPR install ### end

RUN pip3 install opencv-python \
        && pip3 install opencv-contrib-python

# 図表で使うフォントを入れる
RUN echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections \
    && apt-get install -y ttf-mscorefonts-installer

EXPOSE 22

# docker build -t psymonmarkrine:img_prcs .
# docker run --name img --gpus all -d -p 10022:22 -it psymonmarkrine:img_prcs

