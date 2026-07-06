---
name: chrome-webbridge
description: |
  当用户用自然语言要求"操作谷歌浏览器 / 用Chrome打开 / 用谷歌浏览器打开某网站 / 在浏览器里打开XX / 浏览器截图 / 操作Chrome / 在Chrome里搜索、点击、填表、读取页面"时，自动确保 Kimi WebBridge 守护进程运行且 Chrome 扩展已连接，然后用 WebBridge API 操控 Chrome 完成导航、点击、填表、截图、读取页面等操作。这是 kimi-webbridge 技能的"自然语言触发层"——把"打开浏览器做X"这类口语请求自动转成标准 WebBridge 调用流程。
agent_created: true
---

# Chrome WebBridge（自然语言触发层）

把用户口语化的"操作谷歌浏览器做X"请求，自动转成 Kimi WebBridge 的标准调用流程。

**本技能可分享**：朋友装上后，按下方「安装指引」两步即可让 AI 用 WorkBuddy 操控他们的 Chrome/Edge。

⚠️ **前置依赖**：Kimi WebBridge 跑在 Kimi 桌面端 / Kimi Work 内，朋友需先有 **Kimi 账号**（免费注册 https://www.kimi.com ）。

底层依赖：Kimi WebBridge（本地桥接服务 + 浏览器扩展）。它有两种提供方式——**A. Kimi 桌面端（官方推荐，朋友最易上手）** 或 **B. 独立守护进程 `kimi-webbridge.exe`（本机当前用法）**——最终都暴露同一个本地 API（`POST http://127.0.0.1:10086/command`），本技能所有命令通用。

## 安装指引（首次使用 / 分享给朋友必读）

来自官方产品页：https://www.kimi.com/zh-cn/features/webbridge ，帮助中心：https://www.kimi.com/zh-cn/help/kimi-webbridge

### 方式 A：Kimi 桌面端（官方推荐，新用户首选）
1. **下载 Kimi 桌面端**：访问 https://www.kimi.com/zh-cn/features/webbridge → 点「下载 Kimi 桌面端」（Windows / Mac 均有；Mac 直链：https://appsupport.moonshot.cn/api/app/pkg/latest/macos/download）
2. **安装浏览器扩展**：用 Chrome 或 Edge 打开官网 → 从应用商店获取「Kimi WebBridge」扩展，或按「手动安装」提示加载（需 Chrome / Edge，基于 Chrome DevTools Protocol）
3. **启动并连接**：打开 Kimi 桌面端 → 切到 **Kimi Work** 模式 → 发一条指令即自动建桥，例如 `帮我打开小红书，搜索关于 Kimi 发布的帖子`
4. **若提示「插件未连接」**：确认扩展已装 → 在 Kimi Work 重发连接指令 → **重启 Kimi 桌面端**

### 方式 B：独立守护进程（本机当前用法）
- 守护进程路径：`C:/Users/lenovo/.kimi-webbridge/bin/kimi-webbridge.exe`
- 用 `start` / `status` / `upgrade` 等子命令管理（见文末「守护进程管理」）
- Chrome 扩展需手动加载（本机已有，朋友若用此方式需自行准备扩展文件夹）

### 连接自查（兼容两种方式）
```bash
# 方式B机器：用守护进程自带 status
"C:/Users/lenovo/.kimi-webbridge/bin/kimi-webbridge.exe" status
# 方式A机器（无独立 exe）：只要 Kimi 桌面端开着 + Kimi Work 模式，
# 本地 API 已在 10086 上，直接发一条测试 navigate（见下文）收到 ok:true 即连上
```
⚠️ 若 `kimi-webbridge.exe` 不存在，说明走方式 A——无需手动 start，确保 Kimi 桌面端运行且处于 Kimi Work 模式即可。

## 触发词（命中即加载本技能）

- "操作谷歌浏览器 / 用Chrome / 用谷歌浏览器 打开/访问/进入 某网站"
- "在浏览器里打开 XX"、"打开 XX 网站"
- "浏览器截图"、"给网页截图"、"看看这个页面长啥样"
- "在Chrome里 搜索/点击/填表/登录/读取页面内容"
- 任何"用真实浏览器（带登录态）做X"的语境

## 自动前置流程（每次操作前必做，顺序不可省）

### 第1步：健康检查（兼容两种方式）
```bash
# 方式B（有独立exe）：
"C:/Users/lenovo/.kimi-webbridge/bin/kimi-webbridge.exe" status
```
- 若 `kimi-webbridge.exe` 不存在（方式A：走 Kimi 桌面端）→ 改用端口探针确认本地 API 是否在线：
```bash
curl -s -m 3 http://127.0.0.1:10086/ 2>&1 | head -c 200
# 或发一条 status 动作（返回含 running/extension_connected 字段即在线）
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" -d "{\"action\":\"status\"}" 2>&1
```
- 方式A 机器上若探针失败：提醒用户打开 Kimi 桌面端并切到 **Kimi Work** 模式（桥接服务由桌面端提供）。

### 第2步：按状态路由
| 状态 | 动作 |
|------|------|
| `running:false` | 启动：`"...kimi-webbridge.exe" start`（幂等，已运行也安全）|
| `running:true` 但 `extension_connected:false` | 先检查 Chrome 是否运行（见第3步）；若 Chrome 在跑，等 3 秒再查一次 status——扩展通常会自动重连。**不要反复重试命令** |
| `running:true` 且 `extension_connected:true` | 直接执行动作 ✅ |

### 第3步：确保 Chrome 在运行（"自动打开谷歌浏览器"）
如果用户要求"打开浏览器"，而 Chrome 没运行，navigate 会失败。先确认 Chrome 进程：
```bash
tasklist 2>/dev/null | grep -i "chrome" | head -3
```
没有输出（Chrome 未运行）→ 启动它：
```bash
cmd /c start "" "C:/Program Files/Google/Chrome/Application/chrome.exe"
```
启动后等 3 秒让扩展与守护进程建立 WebSocket 连接，再查 `status`。

### 第4步：执行动作（见下方命令模板）

⚠️ **status 延迟陷阱**：守护进程/扩展刚重启或升级后，`status` 可能瞬时显示 `extension_connected:false`，但扩展会自动重连。以实际 `navigate` 命令返回 `{"ok":true}` 为准，**不要被瞬时 status 误导**。

## 常用动作（curl 调 127.0.0.1:10086）

### 打开网页（导航）
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"navigate","args":{"url":"https://www.baidu.com","newTab":true},"session":"s1"}'
```
返回 `{"ok":true,"data":{"success":true,"url":"...","tabId":...}}` 即成功。

### 截图（读取返回的 path，再用 Read 工具打开）
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"screenshot","args":{}}'
```
⚠️ **不要指定 path 到桌面**（Windows 桌面目录被杀软干扰，写入会失败）。省略 `path`，daemon 会自动写到 OS 临时目录并返回路径；再用 `Read` 工具读该路径查看图片。
```bash
# 指定 path 时务必用 temp 目录（别用桌面）：
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"screenshot","args":{"path":"C:/Users/lenovo/AppData/Local/Temp/shot.png"}}'
```

### 读取页面结构（snapshot，找可点击元素 @e 引用）
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"snapshot","args":{}}'
```

### 点击元素（用 @e 引用或 CSS 选择器）
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"click","args":{"selector":"@e123"}}'
```

### 填写输入框
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"fill","args":{"selector":"#kw","value":"测试关键词"}}'
```

### 复用已打开的标签页
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"find_tab","args":{"url":"https://www.kimi.com","active":true}}'
```

### 执行 JS（evaluate）—— 参数名是 `code`，不是 `script`
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"evaluate","args":{"code":"document.title"}}'
```
- 中文返回值需 base64 解码；复杂中文场景改用 `screenshot` 更稳
- 多段 JS 用 IIFE 包一层避免 `const` 重声明报错：`(() => { ...; return x; })()`

### 存为 PDF
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"save_as_pdf","args":{"paper_format":"a4","print_background":true}}'
```

### 任务结束关闭标签页
```bash
curl -s -X POST http://127.0.0.1:10086/command -H "Content-Type: application/json" \
  -d '{"action":"close_session","args":{}}'
```

## session 隔离
每个 session 对应一个标签页组。多网站并行时用不同 session 名：
```bash
-d '{"action":"navigate","args":{"url":"...","newTab":true},"session":"baidu-task"}'
```

## 守护进程管理（仅方式 B 适用）
- 路径：`C:/Users/lenovo/.kimi-webbridge/bin/kimi-webbridge.exe`
- 命令：`status` / `start` / `stop` / `restart` / `logs -n 100` / `upgrade`
- 升级：`kimi-webbridge upgrade`（v1.10.0 → v1.10.3 已验证可用）
  - ⚠️ 首跑可能因 exe 被占用（残留进程/杀软锁文件）失败 → **直接重试即可**，无需手动杀进程
- 方式 A（Kimi 桌面端）用户无需此段，桥接服务随桌面端启停；要重启桥接就重开 Kimi 桌面端切到 Kimi Work。

## 已知坑（Windows 实测）
1. **status 延迟陷阱**：重启/升级后 status 瞬时 false，但扩展会自动重连；以 `navigate` 实测 `ok:true` 为准
2. **升级 exe 占用**：`upgrade` 首跑失败 → 重试成功（无须杀进程）
3. **版本兼容**：守护进程 v1.10.3 + 扩展 v1.10.0 实测完全兼容（navigate 正常）
4. **桌面目录不可写**：`screenshot`/`save_as_pdf` 的 `path` 别用桌面，用 `AppData\Local\Temp`
5. **evaluate 中文**：返回中文需 base64 解码；复杂场景用 screenshot
6. **多浏览器冲突**：status 显示已连但命令失败 → 查 `logs` 看 upgrade rejections
7. **受限站点**：严格校验 `event.isTrusted` 的网站（银行/验证码）会拒绝 fill/click

## 验证清单（open 百度示例）
1. `status` → 若 `running:false` 则 `start`
2. Chrome 未运行则 `cmd /c start chrome`，等 3 秒
3. `navigate baidu.com` → 收到 `ok:true` 即成功
4. 需要视觉确认 → `screenshot` → 读返回 path
