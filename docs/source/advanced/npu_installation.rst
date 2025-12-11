NPU安装及配置
=================

LLaMA-Factory 支持华为昇腾 NPU (A2/A3) 设备。您可以选择以下三种方式之一进行环境配置及使用：

- :ref:`install_form_pip`
- :ref:`use_form_docker`
- :ref:`install_form_docker`


核心依赖说明
----------------

所有安装方式均依赖以下组件：

- **HDK**：固件及驱动
- **CANN**：异构计算架构
- **torch_npu**：PyTorch 的昇腾适配插件

根据安装方式不同，所需操作有所区别：

- **手动安装**：需手动安装 HDK、CANN 和 torch_npu。
- **Docker 镜像/构建**：宿主机仅需安装 HDK (驱动/固件)，CANN 和 torch_npu 已集成在镜像中。


.. _install_form_pip:

方式一：手动安装环境
-------------------

本方式需要您手动安装 HDK、CANN 和 torch_npu。


1. 版本及下载链接
~~~~~~~~~~~~~~~~~~~~

本文档列举了最新的依赖版本及下载链接，请根据设备型号选择：

.. list-table::
   :align: left
   :widths: 5 10 50
   :header-rows: 1

   * - 设备
     - 依赖
     - 链接
   * - A3
     - HDK
     - https://www.hiascend.com/hardware/firmware-drivers/community?product=7&model=34&cann=8.3.RC1&driver=Ascend+HDK+25.0.RC1.3
   * -
     - CANN
     - https://www.hiascend.com/developer/download/community/result?module=pt+cann&product=7&model=34
   * -
     - torch_npu
     - https://www.hiascend.com/developer/download/community/result?module=pt+cann&product=7&model=34
   * - A2
     - HDK
     - https://www.hiascend.com/hardware/firmware-drivers/community?product=4&model=26&cann=8.3.RC1&driver=Ascend+HDK+25.3.RC1
   * -
     - CANN
     - https://www.hiascend.com/developer/download/community/result?module=pt+cann&pt=7.2.0&cann=8.3.RC1
   * -
     - torch_npu
     - https://www.hiascend.com/developer/download/community/result?module=pt+cann&pt=7.2.0&cann=8.3.RC1

.. note::

  - CANN 版本可根据服务器 CPU 型号进行选择；若为 aarch64 架构，请选择尾缀为 ``run`` 的包
  - ``torch_npu`` 下载链接跳转页面，请点击所需的 PyTorch 版本的“获取源码”，链接将自动跳转至 `torch_npu 发行版本 <https://gitcode.com/Ascend/pytorch/releases>`__
  - 推荐使用 ``torch 2.7.1`` + ``python 3.11``，该组合作为 ``torch_npu`` 的稳定版本长期演进

2. 驱动及固件
~~~~~~~~~~~~~~~~~~~~

请根据设备型号（aarch64/x86）下载对应的 ``.run`` 或 ``.deb`` 包。

(1) 上传安装包
    以 root 用户登录，将驱动和固件包上传至服务器（如 ``/home``）。

(2) 增加执行权限
    进入安装包目录，执行以下命令：

    .. code-block:: shell

        chmod +x Ascend-hdk-<chip_type>-npu-driver_<version>_linux-<arch>.run
        chmod +x Ascend-hdk-<chip_type>-npu-firmware_<version>.run

(3) 安装驱动与固件
    默认安装路径为 ``/usr/local/Ascend``。

    **安装驱动**：

    .. code-block:: shell

        ./Ascend-hdk-<chip_type>-npu-driver_<version>_linux-<arch>.run --full --install-for-all

    出现 ``Driver package installed successfully!`` 表示成功。

    **安装固件**：

    .. code-block:: shell

        ./Ascend-hdk-<chip_type>-npu-firmware_<version>.run --full

    出现 ``Firmware package installed successfully!`` 表示成功。

    .. note::

        若未创建默认用户 ``HwHiAiUser``，需在安装命令中指定用户和组：
        ``./Ascend-hdk-*.run --full --install-username=<username> --install-usergroup=<usergroup>``

(4) 重启系统
    根据提示决定是否重启。如需重启：

    .. code-block:: shell

        reboot

(5) 验证安装
    执行以下命令查看驱动加载状态：

    .. code-block:: shell

        npu-smi info

    .. image:: ../assets/advanced/npu-smi.png

3. CANN
~~~~~~~~~~~~~~~~~~~~~

请下载对应架构（aarch64 ``.run`` / x86 ``.deb``）的 CANN 安装包。

(1) 安装 Toolkit 开发套件
"""""""""""""""""""""""""

Toolkit 用于训练、推理及开发。

.. note::
    请确保安装目录可用空间大于 10G。

**步骤**：

1. **授权与安装**：
   默认安装路径：以root 用户安装，安装路径为 ``/usr/local/Ascend``；以普通用户安装，安装路径为 ``${HOME}/Ascend``。

   .. code-block:: shell

       chmod +x Ascend-cann-toolkit_8.3.RC1_linux-aarch64.run
       ./Ascend-cann-toolkit_8.3.RC1_linux-aarch64.run --install

2. **配置环境变量**：
   以root用户为例，建议写入 ``~/.bashrc``。

   .. code-block:: shell

       source /usr/local/Ascend/ascend-toolkit/set_env.sh


(2) 安装 Kernels 算子包
"""""""""""""""""""""""""

需在安装 Toolkit 后执行。

.. code-block:: shell

    chmod +x Atlas-A3-cann-kernels_8.3.RC1_linux-aarch64.run
    ./Atlas-A3-cann-kernels_8.3.RC1_linux-aarch64.run --install

*注：如需安装静态库，请将 ``--install`` 改为 ``--devel``。*


(3) 安装 NNAL 神经网络加速库（可选）
""""""""""""""""""""""""""""""""""

包含 ATB 和 SiP 加速库。需在安装 Toolkit 后执行。

1. **授权与安装**：

   .. code-block:: shell

       chmod +x Ascend-cann-nnal_8.3.RC1_linux-aarch64.run
       ./Ascend-cann-nnal_8.3.RC1_linux-aarch64.run --install

2. **配置环境变量**：
   （二选一，不可同时配置）

   .. code-block:: shell

       # ATB
       source ${HOME}/Ascend/nnal/atb/set_env.sh

       # SiP
       source ${HOME}/Ascend/nnal/asdsip/set_env.sh




4. torch-npu
~~~~~~~~~~~~~~~~~~~~

建议在安装 LLaMA-Factory 时一并安装 ``torch-npu`` 插件：

.. code-block:: bash

    pip install -e ".[torch-npu,metrics]"

5. 验证安装
~~~~~~~~~~~~~~~~

(1) 验证 torch-npu
""""""""""""""""""""""

执行以下 Python 脚本：

.. code-block:: python

    import torch
    import torch_npu
    print(torch.npu.is_available())

预期输出：``True``

.. image:: ../assets/advanced/npu-torch.png

(2) 验证 LLaMA-Factory 环境
"""""""""""""""""""""""""""""

执行以下命令检查环境信息：

.. code-block:: shell

  llamafactory-cli env

若正确显示 NPU 型号及 CANN 版本，说明安装成功：

.. code-block:: shell

    - `llamafactory` version: 0.9.4.dev0
    - Platform: Linux-5.10.0-60.18.0.50.r865_35.hce2.aarch64-aarch64-with-glibc2.35
    - Python version: 3.11.13
    - PyTorch version: 2.7.1+cpu (NPU)
    - Transformers version: 4.57.1
    - Datasets version: 4.0.0
    - Accelerate version: 1.11.0
    - PEFT version: 0.17.1
    - NPU type: Ascend910B1
    - CANN version: 8.3.RC1
    - TRL version: 0.9.6
    - Default data directory: detected


.. _use_form_docker:

方式二：Docker 预安装镜像
---------------------

.. note::
  请确保宿主机已安装固件和驱动，可参考前文进行安装。

LLaMA-Factory 的官方镜像托管于 `Docker Hub <https://hub.docker.com/r/hiyouga/llamafactory/tags>`__ 和 `quay.io <https://quay.io/repository/ascend/llamafactory?tab=tags>`__。

1. 拉取镜像
~~~~~~~~~~~~~~~~~~~~

下载 main 分支最新镜像（请根据设备选择 A2 或 A3）：

.. code-block:: shell

    # Docker Hub
    docker pull hiyouga/llamafactory:latest-npu-a2
    docker pull hiyouga/llamafactory:latest-npu-a3

    # quay.io
    docker pull quay.io/ascend/llamafactory:latest-npu-a2
    docker pull quay.io/ascend/llamafactory:latest-npu-a3

*注：如需特定版本镜像，请访问镜像仓库查看 Tag。*

2. 启动容器
~~~~~~~~~~~~~~~~~~~~

使用以下命令启动容器（请根据实际情况修改 ``DOCKER_IMAGE`` 和 ``device``）：

.. code-block:: bash

  CONTAINER_NAME=llama_factory_npu
  DOCKER_IMAGE=hiyouga/llamafactory:latest-npu-a2

  docker run -itd \
      --cap-add=SYS_PTRACE \
      --net=host \
      --device=/dev/davinci0 \
      --device=/dev/davinci1 \
      --device=/dev/davinci2 \
      --device=/dev/davinci3 \
      --device=/dev/davinci4 \
      --device=/dev/davinci5 \
      --device=/dev/davinci6 \
      --device=/dev/davinci7 \
      --device=/dev/davinci_manager \
      --device=/dev/devmm_svm \
      --device=/dev/hisi_hdc \
      --shm-size=1200g \
      -v /usr/local/sbin/npu-smi:/usr/local/sbin/npu-smi \
      -v /usr/local/dcmi:/usr/local/dcmi \
      -v /etc/ascend_install.info:/etc/ascend_install.info \
      -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
      -v /usr/local/Ascend/driver:/usr/local/Ascend/driver \
      -v /data:/data \
      --name "$CONTAINER_NAME" \
      "$DOCKER_IMAGE" \
      /bin/bash

.. note::

    配置 --privileged=true 可开启特权模式，赋予容器对底层硬件管理设备（如 /dev/davinci_manager）的完整访问权限。这能解决多容器并行场景下，因权限限制导致的驱动初始化失败问题，确保 NPU 资源能被多个容器正常复用。

    注意：若未配置该参数，可能会出现首个容器占用后，后续容器因无权限而无法读取设备的情况。鉴于特权模式的权限过大，生产环境中请务必评估安全风险后慎重使用。


3. 进入容器
~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   docker exec -it llama_factory_npu bash

.. note::
   **NPU 设备挂载说明**：
   通过 ``--device /dev/davinci<N>`` 挂载指定 NPU 卡（支持 0-7）。容器内设备编号会自动重新映射（例如物理机 davinci6 -> 容器内设备 0）。

进入容器后，可直接使用 ``llamafactory-cli train`` 启动训练，无需额外配置。

.. _install_form_docker:

方式三：Docker 本地构建
---------------------

.. note::
  请确保宿主机已安装固件和驱动。

LLaMA-Factory 提供 :ref:`docker_compose` 和 :ref:`docker_build` 两种构建方式。


.. _docker_compose:

1. 使用 Docker Compose 构建（推荐）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(1) 进入目录
    .. code-block:: shell

      cd docker/docker-npu

(2) 构建并启动
    请根据设备型号选择命令：

    .. code-block:: shell

      # Ascend-A2
      docker-compose up -d

      # Ascend-A3
      docker-compose --profile a3 up -d

(3) 进入容器
    .. code-block:: shell

      docker exec -it llamafactory bash

    .. note::
      构建前，请检查 `docker-compose.yml` 中的 `devices` 列表，确保与本机实际 NPU 卡数一致。

.. _docker_build:

2. 使用 Docker Build 构建
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(1) 构建镜像
    在项目根目录下执行：

    .. code-block:: shell

      # Ascend-A2
      docker build -f ./docker/docker-npu/Dockerfile --build-arg INSTALL_DEEPSPEED=false --build-arg PIP_INDEX=https://pypi.org/simple -t llamafactory:latest .

      # Ascend-A3
      docker build -f ./docker/docker-npu/Dockerfile --build-arg BASE_IMAGE=quay.io/ascend/cann:8.3.rc2-a3-ubuntu22.04-py3.11 --build-arg INSTALL_DEEPSPEED=false --build-arg PIP_INDEX=https://pypi.org/simple -t llamafactory:latest .

    *提示：可修改 ``BASE_IMAGE`` 参数指定其他 CANN 版本（参考 `ascend/cann <https://quay.io/repository/ascend/cann?tab=tags&tag=latest>`__）。*

(2) 启动容器
    .. code-block:: shell

      CONTAINER_NAME=llama_factory_npu
      DOCKER_IMAGE=llamafactory:latest
      docker run -itd \
          --cap-add=SYS_PTRACE \
          --net=host \
          --device=/dev/davinci0 \
          --device=/dev/davinci1 \
          --device=/dev/davinci2 \
          --device=/dev/davinci3 \
          --device=/dev/davinci4 \
          --device=/dev/davinci5 \
          --device=/dev/davinci6 \
          --device=/dev/davinci7 \
          --device=/dev/davinci_manager \
          --device=/dev/devmm_svm \
          --device=/dev/hisi_hdc \
          --shm-size=1200g \
          -v /usr/local/sbin/npu-smi:/usr/local/sbin/npu-smi \
          -v /usr/local/dcmi:/usr/local/dcmi \
          -v /etc/ascend_install.info:/etc/ascend_install.info \
          -v /sys/fs/cgroup:/sys/fs/cgroup:ro \
          -v /usr/local/Ascend/driver:/usr/local/Ascend/driver \
          -v /data:/data \
          --name "$CONTAINER_NAME" \
          "$DOCKER_IMAGE" \
          /bin/bash

.. note::

    配置 --privileged=true 可开启特权模式，赋予容器对底层硬件管理设备（如 /dev/davinci_manager）的完整访问权限。这能解决多容器并行场景下，因权限限制导致的驱动初始化失败问题，确保 NPU 资源能被多个容器正常复用。

    注意：若未配置该参数，可能会出现首个容器占用后，后续容器因无权限而无法读取设备的情况。鉴于特权模式的权限过大，生产环境中请务必评估安全风险后慎重使用。

(3) 进入容器
    .. code-block:: shell

      docker exec -it llama_factory_npu bash



在 LLaMA-Factory 中使用 NPU
----------------------------------

LLaMA-Factory 对 NPU 的支持是无侵入的，使用方式与 GPU 基本一致。

**关键区别**：需将环境变量 ``CUDA_VISIBLE_DEVICES`` 替换为 ``ASCEND_RT_VISIBLE_DEVICES``。

**示例**（使用卡 0 和 卡 1）：

.. code-block:: bash

    ASCEND_RT_VISIBLE_DEVICES=0,1 llamafactory-cli train examples/train_lora/llama3_lora_sft.yaml

*更多详情请参考 :doc:`NPU训练 <./npu_training>`。*


昇腾实践参考
-----------------

*   `全流程昇腾实践 <https://ascend.github.io/docs/sources/llamafactory/example.html>`_
