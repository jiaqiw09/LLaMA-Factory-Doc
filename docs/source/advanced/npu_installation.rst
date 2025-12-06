NPU安装及配置
================

目前LLaMA-Factory 通过 torch-npu 库完成了对华为昇腾 A2/A3 训练设备的支持。LLaMA-Factory 提供三种方式使用昇腾进行训练

- 已预装环境的镜像 :ref:`install_form_docker`
- 本地构建环境的镜像
- 自动安装环境

三种安装方式底层都依赖 驱动，cann和torch_npu三种依赖，

- HDK：
- CANN：
- torch_npu：

根据您选择的安装及配置方式，这三个核心依赖在三种方案里，各自有涉及与不涉及的情况。

.. list-table::
   :widths: 30 30 30 30
   :header-rows: 1

   * - 依赖
     - Docker 预安装镜像
     - Docker 本地构建
     - 本地安装依赖
   * - HDK
     - 涉及
     - 不涉及
     - 涉及
   * - CANN
     - 不涉及
     - 不涉及
     - 涉及
   * - torch_npu
     - 不涉及
     - 不涉及
     - 涉及


.. _use_form_docker:

Docker 预安装镜像
---------------------

.. note::
  请确保宿主机已根据昇腾卡型号成功安装对应的固件和驱动，可参考 `快速安装昇腾环境 <https://ascend.github.io/docs/sources/ascend/quick_install.html>`_ 指引。


LLaMA-Factory 已原生支持昇腾镜像构建，并分别推送到 `Dokcer-Hub <https://hub.docker.com/r/hiyouga/llamafactory/tags>`__ 和 `quay.io <https://quay.io/repository/ascend/llamafactory?tab=tags>`__  2个镜像站。

如果您需要main分支最新的镜像，您可以通过以下方式下载A2/A3系列的最新镜像：

.. code-block:: shell

    # Docker-Hub-A2
    docker pull hiyouga/llamafactory:latest-npu-a2

    # Docker-Hub-A3
    docker pull hiyouga/llamafactory:latest-npu-a3

    # quay.io-A3
    docker pull quay.io/ascend/llamafactory:latest-npu-a2

    # quay.io-A3
    docker pull quay.io/ascend/llamafactory:latest-npu-a3

.. note::
  如果您需要定期发布的分支的镜像，您也可以在 `Dokcer-Hub <https://hub.docker.com/r/hiyouga/llamafactory/tags>`__ 或 `quay.io <https://quay.io/repository/ascend/llamafactory?tab=tags>`__ 直接寻找，通过`docker pull`的方式下载到本地使用，2个镜像站的docker images并无差异。

当您成功拉取镜像后，您可以使用以下命令，基于镜像拉起容器，并在容器里进行对应的操作。

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
      --privileged=true \
      --name "$CONTAINER_NAME" \
      "$DOCKER_IMAGE" \
      /bin/bash

进入容器：

.. code-block:: bash

   docker exec -it llama_factory_npu bash

.. note::

   **NPU 设备挂载说明**：

   - 通过 ``--device /dev/davinci<N>`` 参数可挂载指定的 NPU 卡，最多可挂载全部 8 卡
   - 昇腾 NPU 设备从 0 开始编号，容器内的设备编号会自动重新映射
   - 例如：将物理机上的 davinci6 和 davinci7 挂载到容器，容器内对应的设备编号将为 0 和 1

进入容器后，LLaMA-Factory 已预装完成，可直接使用 ``llamafactory-cli train`` 命令启动训练。该命令会自动识别容器内所有挂载的 NPU 设备并启用分布式训练。

.. _install_form_docker:

Docker 本地构建
---------------------

.. note::
  请确保宿主机已根据昇腾卡型号成功安装对应的固件和驱动，可参考 `快速安装昇腾环境 <https://ascend.github.io/docs/sources/ascend/quick_install.html>`_ 指引。

LLaMA-Factory 提供 :ref:`docker_compose` 和 :ref:`docker_build` 两种构建方式，请根据需求选择其一。


.. _docker_compose:

使用 docker-compose 构建并启动 docker 容器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

进入 LLaMA-Factory 项目中存放 Dockerfile 及 docker-compose.yaml 的 docker-npu 目录：

.. code-block:: shell

  cd docker/docker-npu


构建 docker 镜像并启动 docker 容器：

.. code-block:: shell

  # Ascend-A2
  docker-compose up -d

  # Ascend-A3
  docker-compose --profile a3 up -d


进入 docker 容器：

.. code-block:: shell

  docker exec -it llamafactory bash

.. note::
  在构建容器前，请根据您需要的卡数，动态修改`docker-compose.yml`下的devices。

.. _docker_build:

不使用 docker-compose
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

使用 docker build 直接构建 docker 镜像：

.. code-block:: shell

  # Ascend-A2
  docker build -f ./docker/docker-npu/Dockerfile --build-arg INSTALL_DEEPSPEED=false --build-arg PIP_INDEX=https://pypi.org/simple -t llamafactory:latest .

  # Ascend-A3
  docker build -f ./docker/docker-npu/Dockerfile --build-arg BASE_IMAGE=quay.io/ascend/cann:8.3.rc2-a3-ubuntu22.04-py3.11 INSTALL_DEEPSPEED=false --build-arg PIP_INDEX=https://pypi.org/simple -t llamafactory:latest .


.. note::
  您可以根据您对cann版本的需求，自行在 `ascend/cann <https://quay.io/repository/ascend/cann?tab=tags&tag=latest>__` 镜像站里选择合适的BAE_IMAGE构建 docker 镜像。

启动 docker 容器：

.. code-block:: shell

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
      --privileged=true \
      --name "$CONTAINER_NAME" \
      "$DOCKER_IMAGE" \
      /bin/bash


进入 docker 容器：

.. code:: shell

  docker exec -it llama_factory_npu bash


.. _install_form_pip:

自行 pip 安装
-------------------

自行 pip 安装时， python 版本建议使用3.10， 目前该版本对于 NPU 的使用情况会相对稳定，其他版本可能会遇到一些未知的情况

依赖1: NPU 驱动
~~~~~~~~~~~~~~~~~~~~

根据昇腾卡型号安装对应的固件和驱动，可参考 `快速安装昇腾环境 <https://ascend.github.io/docs/sources/ascend/quick_install.html>`_ 指引，使用 ``npu-smi info`` 验证如下

.. image:: ../assets/advanced/npu-smi.png

依赖2: NPU 开发包
~~~~~~~~~~~~~~~~~~~~~

.. list-table:: 相关包建议版本
   :widths: 30 10 60
   :header-rows: 1

   * - Requirement
     - Minimum
     - Recommend
   * - CANN
     - 8.3.RC1
     - 8.3.RC1
   * - torch
     - 2.5.1
     - 2.7.1
   * - torch-npu
     - 2.5.1
     - 2.7.1
   * - deepspeed
     - 0.16.9
     - 0.16.9

可以按照 `快速安装昇腾环境 <https://ascend.github.io/docs/sources/ascend/quick_install.html>`_ 指引，或者使用以下命令完成快速安装：


.. code-block:: bash

    # Atlas A2 Training Series*
    # https://www.hiascend.com/developer/download/community/result
    # 1. Ascend-cann-toolkit_8.3.RC1_linux-aarch64.run
    # 2. Ascend-cann-kernels-910b_8.3.RC1_linux-aarch64.run
    # 3. Ascend-cann-nnal_8.3.RC1_linux-aarch64.run
    # CANN Toolkit
    bash Ascend-cann-toolkit_8.3.RC1_linux-aarch64.run --install

    # CANN Kernels
    bash Ascend-cann-kernels-910b_8.3.RC1_linux-aarch64.run --install

    # nnal
    source /usr/local/Ascend/ascend-toolkit/set_env.sh
    bash Ascend-cann-nnal_8.3.RC1_linux-aarch64.run --install


    # set env variables
    source /usr/local/Ascend/ascend-toolkit/set_env.sh
    source /usr/local/Ascend/nnal/atb/set_env.sh



依赖3: torch-npu
~~~~~~~~~~~~~~~~~~~~

依赖3建议在安装 LLaMA-Factory 的时候一起选配安装， 把 ``torch-npu`` 一起加入安装目标，命令如下

.. code-block:: bash

    pip install -e ".[torch-npu,metrics]"

依赖校验
~~~~~~~~~~~~~~~~
3个依赖都安装后，可以通过如下的 python 脚本对 ``torch_npu`` 的可用情况做一下校验

.. code-block:: python

    import torch
    import torch_npu
    print(torch.npu.is_available())

预期结果是打印true

.. image:: ../assets/advanced/npu-torch.png

安装校验
----------------------

使用以下指令对 LLaMA-Factory × 昇腾的安装进行校验：

.. code-block:: shell
  
  llamafactory-cli env

如下所示，正确显示 LLaMA-Factory、PyTorch NPU 和 CANN 版本号及 NPU 型号等信息即说明安装成功。

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


在 LLaMA-Factory 中使用 NPU 
----------------------------------

前面依赖安装完毕和完成校验后，即可像文档的其他部分一样正常使用 ``llamafactory-cli`` 的相关功能， NPU 的使用是无侵入的。主要的区别是需要修改一下命令行中 设备变量使用
将原来的 Nvidia 卡的变量 ``CUDA_VISIBLE_DEVICES`` 替换为 ``ASCEND_RT_VISIBLE_DEVICES``， 类似如下命令

.. code-block:: bash

    ASCEND_RT_VISIBLE_DEVICES=0,1 llamafactory-cli train examples/train_lora/llama3_lora_sft.yaml


昇腾实践参考
-----------------

如需更多 LLaMA-Factory × 昇腾实践指引，可参考 `全流程昇腾实践 <https://ascend.github.io/docs/sources/llamafactory/example.html>`_ 。
