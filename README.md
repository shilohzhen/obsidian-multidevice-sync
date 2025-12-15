# 📘 Obsidian 多端同步终极方案 (Android + PC + Git)

> 彻底解决 Android 端的 Git 权限问题、仓库损坏问题以及多端插件配置冲突。

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## 痛点与背景
在 Android 设备上直接对 Obsidian Vault 使用 Git 同步经常会遇到以下问题：
1. **权限地狱**：Android 的 `/storage/emulated/0` 目录对 `.git` 文件夹支持极差，导致 push/pull 失败。
2. **仓库损坏**：`.git/objects` 经常出现 `permission denied` 或 `unable to open loose object`。
3. **配置打架**：电脑端的插件配置同步到手机端，导致手机卡顿或布局错乱。

**本方案的核心思路：**
* **Vault 与 Git 分离**：Vault 只存纯文本，Git 仓库运行在 Termux 私有目录下。
* **Rsync 桥接**：使用脚本自动同步 Vault 和 Git 仓库的数据。
* **配置隔离**：通过 `.gitignore` 实现“内容同步，插件独立”。

## 🏗 架构设计

```mermaid
graph TD
    PC[电脑端 Obsidian] <-->|SSH| GH((GitHub))
    GH <-->|SSH| Termux[Termux 私有仓库]
    Termux <-->|Rsync| Android[Android Obsidian Vault]
    
    style Termux fill:#f9f,stroke:#333,stroke-width:2px
    style Android fill:#bbf,stroke:#333,stroke-width:2px
