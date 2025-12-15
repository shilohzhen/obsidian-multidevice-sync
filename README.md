
📘 Obsidian 多端同步终极方案（Android + PC + Git）

解决 Android 端 Git 权限问题、仓库损坏风险，以及多端 Obsidian 插件配置冲突
目标是：稳定、可控、长期可维护


⸻

背景与痛点

在 Android 设备上直接对 Obsidian Vault 使用 Git 同步，实际体验往往不理想，常见问题包括：
	•	权限地狱
/storage/emulated/0 对 .git 目录支持不完整，频繁出现 permission denied
	•	仓库损坏
.git/objects 容易报错，出现 unable to open loose object，仓库直接报废
	•	配置打架
PC 端插件、缓存、布局文件同步到手机端，导致卡顿、闪退、界面错乱

这些问题并非配置失误，而是 Android 文件系统与 Git 设计理念的天然冲突。

⸻

核心思路

这个方案的关键不是“怎么在 Android 上用 Git”，而是让 Git 远离 Android 的共享存储。

整体原则很简单：
	•	Vault 只负责存 Markdown 内容
	•	Git 只运行在 Termux 私有目录
	•	二者通过 rsync 脚本桥接
	•	插件和配置 刻意不参与同步

具体拆解如下：
	•	Vault 与 Git 分离
Obsidian 实际使用的 Vault 不包含 .git 目录
	•	Rsync 桥接同步
使用脚本在 Vault 与 Git 仓库之间做单向/双向同步
	•	配置隔离
通过 .gitignore 控制只同步笔记内容，插件各端独立维护

⸻

🏗 架构设计

graph TD
    PC[电脑端 Obsidian] <-->|SSH| GH((GitHub))
    GH <-->|SSH| Termux[Termux 私有 Git 仓库]
    Termux <-->|Rsync| Android[Android Obsidian Vault]

    style Termux fill:#f9f,stroke:#333,stroke-width:2px
    style Android fill:#bbf,stroke:#333,stroke-width:2px

角色说明：
	•	PC
主编辑端，拥有完整 Git 权限（增删改目录结构）
	•	GitHub
中央仓库，仅作为中转和版本记录
	•	Termux
Git 真正运行的位置，避免 Android 权限问题
	•	Android Vault
只读写 Markdown 内容，不直接接触 Git

⸻

🚀 快速开始

1️⃣ 电脑端准备
	•	安装 Obsidian
	•	安装 Obsidian Git 插件
	•	确保已配置 GitHub SSH 连接
	•	将 templates/.gitignore 内容复制到你的笔记仓库根目录

⸻

2️⃣ Android 端准备
	•	安装：
	•	Obsidian
	•	Termux（F-Droid 版本）

在 Termux 中执行：

pkg update
pkg install git openssh rsync
termux-setup-storage

⚠️ 注意
不要在 /storage/emulated/0 下 clone 仓库

在 Termux 私有目录中克隆仓库：

cd ~
git clone git@github.com:yourname/your-vault.git Obsidian_Sync_repo


⸻

3️⃣ 部署同步脚本

将本仓库 src/ 目录下的脚本复制到 Termux：

mkdir -p ~/bin
# 手动创建或复制 gsync / gsync_pull 到 ~/bin
chmod +x ~/bin/gsync ~/bin/gsync_pull

然后务必修改脚本中的 VAULT 变量，指向手机上真实的 Obsidian Vault 路径，例如：

VAULT="/storage/emulated/0/Documents/Obsidian/YourVaultName"


⸻

📖 使用指南

场景	操作	说明
电脑端写完笔记	Git Push	电脑端拥有最高权限
安卓端只更新内容	gsync_pull	覆盖本地内容
安卓端新增/修改	gsync	仅上传新增和修改
安卓端误删文件	不会同步	防止误删传播

设计原则很明确：
	•	删除权只留在 PC
	•	Android 永远偏保守

⸻

🛠 推荐配置（可选）

Termux:Widget 一键同步

安装 Termux:Widget 后，在桌面创建快捷按钮：

mkdir -p ~/.shortcuts
ln -sf ~/bin/gsync ~/.shortcuts/Obsidian_上传
ln -sf ~/bin/gsync_pull ~/.shortcuts/Obsidian_拉取

之后可以在桌面一键同步，不需要再进 Termux。

⸻

适合人群
	•	多设备使用 Obsidian（PC + Android）
	•	不想再被 Android Git 权限问题折磨
	•	希望笔记长期可控、可迁移、可回滚
	•	对“自动同步但可控”有明确边界要求

