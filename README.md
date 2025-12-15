# 📘 Obsidian 多端同步终极方案 (Android + PC + Git)

> **彻底解决 Android 端的 Git 权限问题、仓库损坏问题以及多端插件配置冲突。**

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Obsidian](https://img.shields.io/badge/Obsidian-%23483699.svg?style=flat&logo=obsidian&logoColor=white)
![Termux](https://img.shields.io/badge/Termux-black?style=flat&logo=linux&logoColor=white)

## 🧐 痛点与背景

在 Android 设备上直接对 Obsidian Vault 使用 Git 同步经常会遇到以下“地狱级”问题：

1.  ❌ **权限地狱**：Android 的 `/storage/emulated/0` 目录对 `.git` 文件夹支持极差，导致 push/pull 频繁失败。
2.  ❌ **仓库损坏**：`.git/objects` 经常出现 `permission denied` 或 `unable to open loose object`，导致仓库报废。
3.  ❌ **配置打架**：电脑端的插件配置同步到手机端，导致手机卡顿、布局错乱或快捷键冲突。

### 本方案的核心思路

* **🛡️ Vault 与 Git 分离**：Vault (在共享存储) 只存纯文本，Git 仓库 (在 Termux 私有目录) 负责版本控制。
* **🌉 Rsync 桥接**：使用脚本自动双向同步 Vault 和 Git 仓库的数据，避开 Android 权限限制。
* **🧩 配置隔离**：通过精心设计的 `.gitignore` 实现“内容全同步，插件/配置独立”。

---

## 🏗 架构设计

本方案的工作流如下所示：

```mermaid
graph TD
    PC[💻 电脑端 Obsidian] <-->|SSH| GH((GitHub))
    GH <-->|SSH| Termux[📱 Termux 私有仓库]
    
    subgraph Android Device
        Termux <-->|Rsync 脚本| Android[📂 Android Obsidian Vault]
    end
    
    style Termux fill:#f9f,stroke:#333,stroke-width:2px,color:black
    style Android fill:#bbf,stroke:#333,stroke-width:2px,color:black

🚀 快速开始
1. 电脑端准备
安装 Obsidian Git 插件（用于电脑端自动同步）。

确保本地已配置 SSH 连接 GitHub。

关键步骤：将本仓库 templates/.gitignore 中的内容复制到你的笔记仓库根目录。

这可以防止电脑端的 workspace、插件配置污染手机端。

2. Android 端环境准备
安装 Obsidian 和 Termux (推荐使用 F-Droid 版本，Play 商店版已不再更新)。

在 Termux 中安装必要的依赖：
pkg update
pkg install git openssh rsync
termux-setup-storage
