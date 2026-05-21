# Docker + Ubuntu 容器安装 Miniforge 使用教程

本文档总结了在 **Windows 主机 + Docker + Ubuntu 容器** 下安装和使用 **Miniforge（Conda 轻量版）** 的完整流程，包括容器创建、挂载目录、Miniforge 安装、Python 环境管理等详细步骤，适合初学者和开发者参考。

---

## 目录

1. [拉取 Ubuntu 镜像](#拉取-ubuntu-镜像)
2. [创建容器并挂载 Windows 目录](#创建容器并挂载-windows-目录)
3. [退出与重新进入容器](#退出与重新进入容器)
4. [终端复制粘贴快捷键](#终端复制粘贴快捷键)
5. [安装 wget](#安装-wget)
6. [下载并安装 Miniforge](#下载并安装-miniforge)
7. [配置 PATH 并初始化 conda](#配置-path-并初始化-conda)
8. [创建 Python 环境](#创建-python-环境)

---

## 拉取 Ubuntu 镜像

```bash
docker pull ubuntu
docker images
```

* **说明**：

  * 拉取的最小镜像约 78MB，只包含基础文件系统。
  * 检查镜像是否成功拉取。

---

## 创建容器并挂载 Windows 目录

```bash
docker run -it --name my_ubuntu -v C:\Users\Administrator\Desktop\docker_ubuntu_dir:/workspace ubuntu /bin/bash
```

* **参数说明**：

  * `docker run`：创建并启动新容器
  * `-it`：交互式终端（-i 保持 STDIN，-t 分配 TTY）
  * `--name my_ubuntu`：指定容器名称
  * `-v 主机路径:容器路径`：挂载 Windows 目录到容器内
  * `ubuntu`：使用的镜像
  * `/bin/bash`：容器启动后执行的进程

* **注意事项**：

  * Windows CMD 下使用多行 `^` 可能报错 `"invalid reference format"`，推荐一行命令完成。
  * 挂载的 `/workspace` 目录用于数据和代码持久化。

---

## 退出与重新进入容器

```bash
exit                     # 退出容器
docker start my_ubuntu   # 启动已停止容器
docker exec -it my_ubuntu /bin/bash   # 进入容器
```

* **说明**：

  * 容器生命周期由 PID 1 进程决定，退出后容器停止。
  * `start + exec` 可以重新进入已停止容器。

---

## 终端复制粘贴快捷键

* **Ubuntu 终端**：

  * 复制：`Ctrl + Shift + C`
  * 粘贴：`Ctrl + Shift + V`
* **Ubuntu GUI 应用**：

  * 复制：`Ctrl + C`
  * 粘贴：`Ctrl + V`
* **鼠标操作**：

  * 选中即复制（Primary Selection）
  * 中键点击粘贴
* **注意**：

  * `Ctrl + C` 在终端中为中断进程，不是复制。
  * `docker exec` 进入容器终端时仍需使用 `Ctrl + Shift + V` 粘贴。

---

## 安装 wget（容器默认无）

```bash
apt update
apt install -y wget
wget --version
```

* **说明**：

  * Ubuntu 最小镜像没有 `wget`，安装后可正常使用。

---

## 下载并安装 Miniforge

```bash
cd /opt
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
chmod +x Miniforge3-Linux-x86_64.sh
bash Miniforge3-Linux-x86_64.sh
```

* **安装步骤**：

  1. 阅读许可协议并输入 `yes`
  2. 默认安装路径：`/root/miniforge3`
  3. 初始化 shell 时输入 `yes`

* **常见问题**：

  * 安装后直接执行 `conda` 命令可能提示 `command not found`
    → 原因：容器 shell PATH 没包含 Miniforge 的 bin。

---

## 配置 PATH 并初始化 conda

```bash
# 临时生效
export PATH=/root/miniforge3/bin:$PATH

# 永久生效
echo 'export PATH=/root/miniforge3/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 初始化 conda shell
/root/miniforge3/bin/conda init bash
source ~/.bashrc

# 检查
conda --version
which conda
conda info
```

* **说明**：

  * 添加 PATH 确保 `conda` 命令可用。
  * 初始化 shell 让 conda 命令在每次打开终端时生效。

---

## 创建 Python 环境（推荐）

```bash
# 创建 Python 3.10 环境
conda create -n py310 python=3.10 -y

# 激活环境
conda activate py310

# 安装常用库
conda install numpy scipy pandas matplotlib -y
```

* **说明**：

  * 不要直接在 `base` 环境下工作，推荐使用独立环境。
  * `/workspace` 挂载目录用于存放数据和代码，可在容器和 Windows 主机之间共享。

---

## 补充说明

* 建议定期更新 conda：

  ```bash
  conda update -n base -c defaults conda -y
  ```
* 挂载 Windows 目录时注意权限问题，如果遇到写入错误，可尝试：

  ```bash
  chmod -R 777 /workspace
  ```
* 对于多 Python 项目，推荐每个项目创建独立 conda 环境，避免依赖冲突。

---

这份 README.md 可以直接复制到 GitHub，任何初学者跟着步骤即可完成 Docker + Ubuntu + Miniforge 环境搭建。
