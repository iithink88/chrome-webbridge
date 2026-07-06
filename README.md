# chrome-webbridge

> WorkBuddy 技能：让 AI 用真实浏览器（带登录态）操控你的 Chrome / Edge。
> 基于 Kimi WebBridge 本地桥接服务（127.0.0.1:10086）。

✅ **本技能已把官网与安装流程完整融进 SKILL.md。**

---

## 关键发现

官网显示：**Kimi WebBridge 现在主要走 Kimi 桌面端（Kimi Work 模式）** —— 本地桥接服务由桌面端提供，**不是必须独立装守护进程**。

但两种方式的本地 API（`127.0.0.1:10086`）**完全通用**，所以下面所有命令朋友也能直接用。

---

## 本次技能新增内容

| 改动 | 说明 |
|------|------|
| 安装指引章节 | 完整两步流程（下载桌面端 → 装扩展 → Kimi Work 建桥） |
| 官网链接 | https://www.kimi.com/zh-cn/features/webbridge + 帮助中心链接 |
| 两种方式兼容 | 方式A（Kimi 桌面端，推荐给朋友）/ 方式B（独立 exe，本机用法），命令通用 |
| 连接自查 | exe 不存在时改用端口探针，兼容方式A机器 |
| 守护进程管理 | 标注「仅方式B适用」，朋友用方式A不会困惑 |
| 前置依赖 | 明确提醒朋友需先有 Kimi 账号（免费注册） |

---

## 朋友现在的使用路径

1. 拿到 `chrome-webbridge` 技能文件夹 → 放进 `~/.workbuddy/skills/`
2. 看 `SKILL.md` 里的「安装指引」→ 下载 Kimi 桌面端 + 装浏览器扩展
3. 对 AI 说「操作谷歌浏览器打开 XX」→ AI 自动走健康检查 + WebBridge 操控

---

## 朋友怎么装这个技能

仓库里的 `SKILL.md` 已经把官网与安装流程融进去了，朋友拿到后任选其一即可：

1. **下载 `SKILL.md` 直接拖进 WorkBuddy 聊天框**
   - 最轻量：只传一个文件，AI 自动识别为技能
2. **把整个仓库文件夹放进 `~/.workbuddy/skills/`**
   - 把本仓库 clone / 解压后，整个 `chrome-webbridge/` 目录移到 `~/.workbuddy/skills/` 下即可
3. **用 npx 命令一键安装**
   ```bash
   npx skills add iithink88/chrome-webbridge@chrome-webbridge
   ```

装好之后，按 `SKILL.md` 里的「安装指引」下载 Kimi 桌面端 + 装浏览器扩展，然后对我说「操作谷歌浏览器打开 XX」，我就会自动走健康检查 + WebBridge 操控。

---

## 官方资源

- 产品页：https://www.kimi.com/zh-cn/features/webbridge
- 帮助中心：https://www.kimi.com/zh-cn/help/kimi-webbridge
