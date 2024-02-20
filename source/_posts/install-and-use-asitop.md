---
title: 基于命令的 macOS 资源监控工具 asitop，可以监控CPU、GPU功耗
date: 2024-02-20 17:39:04
categories: 实用技巧
tags: 
    - Mac
---
![CleanShot 2024-02-20 at 17.23.26](/media/CleanShot%202024-02-20%20at%2017.23.26.png)

`asitop` 是一个用于监视 Apple Silicon Macs 性能的命令行工具，包括 CPU、GPU、ANE（Apple Neural Engine）的使用率，以及功耗和温度信息。它提供了一个实时更新的界面，可以让用户了解系统的性能状态。安装 `asitop` 的步骤如下：

1. **安装 Homebrew**：如果你的 Mac 上还没有安装 [Homebrew](https://brew.sh)，首先需要安装它。Homebrew 是 macOS 的包管理器，可以让你轻松安装、更新和管理软件包。在终端（Terminal）中运行以下命令安装 Homebrew：

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. **安装 asitop**：一旦安装了 Homebrew，接下来可以使用 Homebrew 安装 `asitop`。在终端中运行以下命令：

   ```bash
   brew install asitop
   ```

3. **运行 asitop**：安装完成后，你可以通过在终端中输入 `sudo asitop` 来运行它（需要提供密码）。首次运行时，可能需要在“系统偏好设置”中授予 `asitop` 相关权限，比如“辅助功能”权限，以便它能够监视系统性能。

4. **退出 asitop**：在 `asitop` 运行时，可以通过按 `q` 键来退出程序。

请注意，`asitop` 的安装和使用可能会随着时间和软件版本的更新而变化。如果在安装或使用过程中遇到问题，建议参考 `asitop` 的官方文档或 GitHub 页面获取最新的指导和帮助。