# github-first-upload

一个手把手教人**第一次把本地项目上传到 GitHub** 的 [Agent Skill](https://agentskills.io)，适用于 Claude Code、Codex 等 30+ 支持技能标准的智能体。

## 解决什么问题

第一次往 GitHub 传项目，新手通常会连环卡住：Git 装没装？GitHub CLI 是什么？怎么登录授权？默认分支怎么设？传上去会不会泄露隐私？仓库公开前要确认什么？这个技能把整条路径固化成一套引导流程，一步一步带着走，每个关键节点先确认再执行，全程不需要把任何密码或 Token 粘贴到对话里。

## 主要功能

- **环境检查**：检测 Git 与 GitHub CLI 是否安装；缺失时按操作系统给出官方安装方案，涉及管理员权限先征求确认
- **授权登录**：引导浏览器设备码登录 GitHub（HTTPS 协议 + 为 Git 配置凭据），二次验证由用户本人完成
- **仓库初始化**：`git init -b main`、配置仓库级提交身份（提醒邮箱公开问题，支持 noreply 邮箱）、生成 `.gitignore`
- **隐私扫描**：公开前自动检查密码/Token、个人隐私、本地绝对路径、临时文件、不应公开的内部目录
- **确认后上传**：列出仓库名、可见性、远程名、分支，用户确认后才创建远程仓库并推送
- **可选发版**：就绪检查、版本号对齐、annotated tag、GitHub Release

明确不适用：已连接远程仓库的日常 pull/push 同步、GitLab/Gitee 等其他平台。

## 安装方法

纯指令型技能（无脚本、无依赖），复制到所用智能体的技能目录即可：

```bash
# Claude Code（用户级）
cp -r . ~/.claude/skills/github-first-upload

# Codex CLI（用户级）
cp -r . ~/.codex/skills/github-first-upload

# WorkBuddy（用户级 / 项目级）
cp -r . ~/.workbuddy/skills/github-first-upload
```

安装后新开会话即可触发；Codex 中也可用 `$github-first-upload` 显式调用。

## 使用方法

对智能体说：

```
帮我把这个项目第一次上传到 GitHub
```

或者更具体：

```
请检查电脑是否装了 Git 和 GitHub CLI，没有的话先说明再装。
然后把当前项目发布到 GitHub：公开仓库、远程名 origin、推送 main 分支。
执行前让我确认。
```

技能会按固定顺序引导：环境检查 → 登录 → 初始化 → 首次提交 → 隐私扫描 → 上传确认 → 推送 →（可选）发版。

## 输入输出示例

**输入**（用户指令）：

> 请初始化 Git 仓库，默认分支设为 main，并准备第一次提交。

**输出**（技能引导的执行过程，节选）：

```
Git 版本：git version 2.55.0.windows.3   ← 检查通过
gh auth status：未登录
  → 启动浏览器授权，一次性验证码：XXXX-XXXX
  → 用户本人在 https://github.com/login/device 完成授权
git init -b main                          ← 默认分支 main
隐私扫描：5 项检查通过，无敏感内容
⚠️ 上传确认：仓库 article-to-knowledge-cards / Public / origin / main
  → 用户确认后执行 gh repo create ... --push
✅ 已发布：https://github.com/<账号>/article-to-knowledge-cards
```

## License

MIT
