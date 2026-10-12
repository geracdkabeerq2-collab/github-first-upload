---
name: github-first-upload
description: 引导用户把本地项目第一次上传到 GitHub 的完整流程。当用户说"第一次上传 GitHub"、"把项目传到 GitHub"、"初始化仓库并推送"、"发布到 GitHub"且该项目尚无远程仓库时使用。涵盖：检查/安装 Git 与 GitHub CLI、浏览器授权登录、初始化 main 分支、隐私扫描、确认后创建远程仓库并推送、可选的 tag 与 Release。不适用于已连接远程仓库的日常 pull/push 同步。Guide a first-time upload of a local project to GitHub: environment check, auth, repo init, privacy scan, confirmed push, optional release.
license: MIT
metadata:
  version: "0.1.0"
  author: Tanay Gera
agent_created: true
---

# GitHub First Upload（首次上传 GitHub 引导）

## Overview

带领一个从未推送过 GitHub 的本地项目完成首次发布：环境检查 → 授权登录 → 仓库初始化 → 隐私扫描 → 用户确认 → 创建远程仓库并推送 →（可选）打 tag 发 Release。全程在关键节点向用户确认，凭据永不进入对话。

## 输入与边界

适用：本地项目目录（尚无远程仓库，或用户明确表示要"第一次上传"）。

不适用（告知用户后改用普通 git 流程）：

- 已有远程仓库的日常 pull / push / 同步
- GitLab、Gitee 等其他平台
- 向已有仓库追加新协作者、改权限等仓库管理操作

## 安全与确认原则（贯穿全流程）

- **凭据不入对话**：绝不要求用户把密码、Token、验证码发到对话里；浏览器授权、2FA 一律由用户本人完成，进程挂后台等待。
- **节点确认**：安装软件、弹管理员授权窗（UAC/sudo）、创建公开仓库、推送，执行前必须先说明并获得用户确认。
- **公开前必扫**：仓库设为 Public 前必须完成隐私扫描（见步骤 4）。
- **仓库级配置**：Git 身份（user.name / user.email）写入仓库本地配置，不碰全局。

## Workflow

### 步骤 0：环境检查

1. `git --version`。未安装则按操作系统选官方方式安装（Windows: winget；macOS: brew；Linux: apt/dnf）。涉及管理员权限或安装新包管理器时，先说明方案并等用户确认。
2. `gh --version`（GitHub CLI）。未安装同样按官方方式处理并先确认。

### 步骤 1：登录检查

1. `gh auth status`。已登录则报账号与协议，跳到步骤 2。
2. 未登录时后台启动：`gh auth login --hostname github.com --git-protocol https --web --skip-ssh-key`。
3. 把输出中的一次性验证码和 https://github.com/login/device 告知用户，由用户本人在浏览器完成授权（含 2FA）。
4. 授权完成后重跑 `gh auth status` 确认，向用户报告账号名、协议、Token 权限范围。

### 步骤 2：Git 身份与仓库初始化

1. 检查 `git config user.name / user.email`（仓库级优先）。缺失时询问用户；提醒邮箱会随提交历史公开，建议可用 GitHub noreply 邮箱，尊重用户选择。
2. 若目录尚不是仓库：`git init -b main`（默认分支 main，除非用户另行要求）。
3. 确保存在 `.gitignore`：至少排除本地内部目录（如 `.workbuddy/`）、系统临时文件（`.DS_Store`、`Thumbs.db`、`*.tmp`、`*.bak`）。

### 步骤 3：首次提交

1. `git add -A` 后检查 `git status`，向用户展示将提交的文件清单。
2. 创建首次提交，提交信息说明项目用途。

### 步骤 4：隐私扫描（Public 前必做）

扫描待提交内容：

- 密码 / Token / API Key / 凭据类字符串
- 个人隐私（邮箱、电话、住址、真实姓名等非自愿公开信息）
- 本地绝对路径（`C:\Users\...`、`/home/...`）
- 临时文件、垃圾文件（`.tmp`、`.log`、`.bak`）
- 不应公开的内部目录（记忆、日志、IDE 配置等）

发现问题先报告并给出处理建议（加 `.gitignore`、删文件、改占位），处理完再继续；无问题则明确告知扫描通过。

### 步骤 5：上传确认（必须停下等用户）

向用户完整列出并确认：

- 仓库名称（默认取项目目录名，可建议更贴切的名字）
- 可见性：Public 还是 Private
- 远程名称（默认 origin）
- 推送分支（默认 main）与提交内容摘要

用户确认前不得执行任何创建/推送操作。

### 步骤 6：创建远程仓库并推送

1. 确认后执行：`gh repo create <仓库名> --public|--private --source <项目路径> --remote origin --push`。
2. 验证：`git ls-remote origin` 确认远程分支与提交一致；`git status` 确认本地与 origin 同步（若 remote-tracking 引用未建立，手动补 `refs/remotes/origin/<分支>` 指向对应提交）。
3. 向用户报告仓库 URL。

### 步骤 7（可选）：发布首个版本

用户要求发布版本时：

1. 就绪检查：工作区干净、验证通过、文档齐全；版本号与项目内声明（如 SKILL.md metadata）对齐，不一致先修正并提交。
2. `git tag -a v<版本> -m "<版本>: <一句话>"` 并推送 tag。
3. `gh release create v<版本> --title "<版本> — <标题>" --notes-file <说明文件>`，发布说明含：简介、当前功能、安装方法、已知限制。

## 边界情况

- **目录已是仓库且有提交**：跳过 init，直接从隐私扫描（步骤 4）继续。
- **用户不想装 gh**：引导用户在网页手动建仓，`git remote add origin <URL>` 后推送，凭据走系统弹窗登录。
- **推送报锁文件错误**（如 `could not lock config file ... gitconfig.lock`）：删除对应零字节 `.lock` 文件后重试；推送结果以 `ls-remote` 实际状态为准，勿只看本地引用。
- **首次提交已存在但用户又要求"做第一次提交"**：展示现有提交，不重复创建。
- **用户后悔公开**：提醒可改为 Private，但已公开期间的内容可能已被他人获取/克隆，历史中的邮箱等信息需改写历史才能彻底移除。

## 示例

用户指令：

> 请检查当前电脑是否安装 Git……如果没有安装，先向我说明并等待确认。
> 请初始化 Git 仓库，默认分支设为 main，并准备第一次提交。
> 请准备把当前项目发布到 GitHub：仓库名称 xxx、Public、origin、推送 main。执行前先让我确认。

按本 Skill 执行：步骤 0 检出 git 2.55 / gh 未装 → 用户确认后 winget 安装 → 浏览器授权登录 → init -b main + .gitignore → 隐私扫描通过 → 用户确认仓库名与可见性 → `gh repo create xxx --public --source . --remote origin --push` → 报告 https://github.com/<账号>/xxx。
