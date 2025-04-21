# 使用Ubuntu 22.04作为基础镜像
FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/xinghanxu/mcsm:ubuntu22.04

# 设置环境变量
ARG VENV_NAME="cosyvoice"
ENV VENV=$VENV_NAME
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1

# 使用Bash作为默认Shell
SHELL ["/bin/bash", "--login", "-c"]

# 更新包列表并安装依赖
RUN apt-get update -y --fix-missing && \
    apt-get install -y git build-essential curl wget ffmpeg unzip git-lfs sox libsox-dev && \
    apt-get clean && \
    git lfs install

# 安装Miniforge并配置Conda
RUN wget --quiet https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh -O ~/miniforge.sh && \
    /bin/bash ~/miniforge.sh -b -p /opt/conda && \
    rm ~/miniforge.sh && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo "source /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate ${VENV}" >> ~/.bashrc

# 将Conda路径添加到系统PATH中
ENV PATH /opt/conda/bin:$PATH

# 下载预训练模型
RUN mkdir -p pretrained_models && \
    git clone https://www.modelscope.cn/iic/CosyVoice-300M.git pretrained_models/CosyVoice-300M

# 配置Conda渠道
RUN conda config --add channels conda-forge && \
    conda config --set channel_priority strict

# 创建虚拟环境并激活
RUN conda create -y -n ${VENV} python=3.10
ENV CONDA_DEFAULT_ENV=${VENV}
ENV PATH /opt/conda/envs/${VENV}/bin:$PATH

# 设置工作目录
WORKDIR /workspace

# 设置Python路径
ENV PYTHONPATH="${PYTHONPATH}:/workspace/CosyVoice:/workspace/CosyVoice/third_party/Matcha-TTS"

# 克隆项目代码
RUN git clone --recursive https://github.com/FunAudioLLM/CosyVoice.git

# 安装Python依赖
RUN conda activate ${VENV} && \
    conda install -y -c conda-forge pynini==2.1.5 && \
    cd CosyVoice && \
    pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com

# 设置容器启动命令
CMD ["conda", "run", "--no-capture-output", "-n", "${VENV}", "python3", "webui.py", "--port", "5000", "--model_dir", "/workspace/pretrained_models/CosyVoice-300M"]