ARG PYTORCH="1.6.0"
ARG CUDA="10.1"
ARG CUDNN="7"

FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0+PTX"
ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

RUN apt-get update && apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6  wget unzip sudo lsof \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install MMCV
RUN pip install Cython
RUN pip install mmcv-full==latest+torch1.6.0+cu101 -f https://openmmlab.oss-accelerate.aliyuncs.com/mmcv/dist/index.html
RUN pip install mmdet

# Install MMDetection
RUN conda clean --all
RUN git clone https://github.com/open-mmlab/mmdetection3d.git /mmdetection3d
WORKDIR /mmdetection3d
ENV FORCE_CUDA="1"
RUN pip install -r requirements/build.txt
RUN pip install --no-cache-dir -e .

# uninstall pycocotools installed by nuscenes-devkit and reinstall mmpycocotools
RUN pip uninstall pycocotools --no-cache-dir -y
RUN pip install mmpycocotools --no-cache-dir --force --no-deps

## Ubuntu 配置admin账户信息, sudo默认密码admin
RUN groupadd -r -g 505 admin && useradd --no-log-init -m -r -g 505 -u 505 admin -s /bin/bash -p admin && mkdir -p /data && chown -fR admin:admin /data && \
echo admin:admin | chpasswd

## AliOS add admin user as sudoer
#RUN groupadd -g 505 admin && useradd -g 505 -u 505 admin && mkdir -p /data && chown -fR admin:admin /data
#RUN echo admin:admin | chpasswd
#RUN adduser admin sudo

# Install jupterlab, default access password is "passw0rd"
RUN pip install jupyterlab
RUN ln -s /opt/conda/bin/jupyter /usr/local/bin/jupyter

RUN mkdir /result && mkdir /model && mkdir /log && mkdir /summary
RUN chmod -R 777 /result && chmod -R 777 /model && chmod -R 777 /log && chmod -R 777 /summary

RUN wget https://pitaya-data-for-annotation.oss-cn-hangzhou.aliyuncs.com/software/code-server_3.4.1_amd64.deb && dpkg -i code-server_3.4.1_amd64.deb
RUN code-server  --extensions-dir /usr/lib/code-server/lib/vscode/extensions/ --install-extension ms-python.python@2020.5.86806

# Clear
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN /opt/conda/bin/conda clean --all -y

USER admin
WORKDIR /
CMD ["/bin/bash","-c","tail -f /dev/null"]