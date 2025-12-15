# Obsidian 多端 Git 同步（电脑 + 安卓手机 + 安卓平板）

这个仓库记录一套在真实使用中跑通的 Obsidian 多端同步方案：电脑作为主编辑环境，安卓手机/平板通过 Termux + Git + rsync 快速同步 Vault。
重点覆盖：
- 设备角色分工（电脑 / Git 仓库 / 安卓）
- 安卓端 Termux 脚本（gsync / gsync_pull / gsync_push）
- Obsidian 目录排除策略（避免把缓存、插件临时文件同步出去）
- 安卓端常见坑：rsync Permission denied、/." 路径误判、storage 权限

---

## 适用场景

- Vault 体积不小，插件多，直接用网盘容易冲突
- 电脑要稳定写作，手机/平板主要查看、轻量修改
- 想把同步链路做成“1 条命令完成”

---

## 设备分工（非常关键）

### 电脑
- 主要写作、重度编辑、批量改结构
- 负责解决冲突、做大规模 refactor
- 作为“最终一致性”的仲裁点

### Git 仓库（GitHub / 自建 Git 服务都可）
- 只负责版本与合并，不参与“实时锁”
- 历史可追溯，误删可回滚
- 作为多端交换站

### 安卓手机 / 安卓平板（Termux）
- 主要用途：随手记、查阅、轻改
- 同步动作尽量短平快：pull -> rsync 到 Vault；改完再 push
- 避免在安卓端做大量文件移动/重命名（容易冲突）

---

## 快速开始

### 1）电脑端准备
- 初始化仓库，把 Vault 内容放进去（或把已有 Vault 推到远端）
- 确保 .gitignore 排除掉 Obsidian 的缓存类目录（见下方）

电脑端建议阅读：docs/desktop-setup.md

### 2）安卓端准备（手机/平板都一样）
安装 Termux 后执行：
- 开启存储权限：termux-setup-storage
- 安装依赖：pkg update && pkg install git rsync openssh

安卓端详细步骤见：docs/android-setup.md

### 3）同步命令
- 拉取并覆盖 Vault（推荐日常用）：scripts/gsync_pull
- 推送改动到远端：scripts/gsync_push
- 一键状态检查：scripts/gsync

工作流建议见：docs/workflows.md

---

## 建议的 .gitignore

把下面这段复制到仓库根目录 .gitignore：

- 忽略 .obsidian 缓存、workspace、trash
- 保留插件本体还是不保留，需要按团队习惯选（这里默认不同步插件目录）

详见仓库文件：.gitignore

---

## 常见问题速查

### rsync: [sender] opendir "/." failed: Permission denied (13)
这类报错的本质通常是 rsync 的源路径被解析成了根目录 / 或者出现了 "/." 这种异常路径，Termux 没权限读整个根目录，导致 Permission denied。
处理思路：
- 确认脚本里 REPO / VAULT 变量是否为空
- 给路径加双引号
- 在脚本开头加 set -euo pipefail，变量空会直接报错，不会误同步到 /

详见：docs/troubleshooting.md

---

## 文档导航
- docs/architecture.md：整体架构、数据流
- docs/android-setup.md：安卓端落地步骤
- docs/desktop-setup.md：电脑端建议
- docs/workflows.md：推荐日常操作节奏
- docs/troubleshooting.md：你遇到的典型坑复盘

---

## 许可
个人自用为主，随意 fork 改造。
