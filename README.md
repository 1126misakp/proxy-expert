# proxy-expert — 魔法搭建专家技能

一个端到端的魔法搭建Skill，适用于任何可以调用skills的智能体，基于 **VLESS + Reality + sing-box + 本地clash-verge** 方案，支持 Mac、Windows 和手机（安卓 / iOS），自动化 VPS 服务端部署，引导客户端配置，从此告别边问ai边复制黏贴折腾的搭建过程～

### 直接问 vs 用 Skill

| | **直接问 AI** | **用 proxy-expert Skill** |
|---|---|---|
| **服务端部署** | 逐条复制粘贴命令，手动改配置，容易出错 | ✅ **全自动**：SSH 执行，自动装 sing-box、生成密钥、写配置、起服务 |
| **客户端配置** | 手动填 YAML，漏一条规则就 SSH 死锁 | ✅ **半自动**：自动生成带防死锁规则的 YAML，只需粘贴导入 |
| **验收测试** | 自己开浏览器试，不知道通没通 | ✅ **全自动**：7 项连通测试自动执行，结果写入报告 |
| **问题排查** | 每次出问题要从头解释 VPS IP、方案、历史，即使在同一聊天框内也会因会话堆积爆上下文导致越改越错 | ✅ **读报告即知**：`proxy-acceptance-report.md` 记录完整历史和方案，AI 先读报告再排查 |
| **手机端扩展** | 重新问一遍，可能配出不同参数 | ✅ **一键延伸**：电脑端验收通过后，同一套配置直接生成 VLESS 链接导入手机 |
| **持续维护** | 对话一结束，上下文全丢，下次从零开始 | ✅ **可记忆**：报告长期保存，问题记录持续追加，越用越聪明 |

**实测效果**：接近上游出口上限的下载速度，抗 GFW 主动探测，长期稳定运行。

**安全声明**：VPS IP、SSH 密码/密钥、Reality 私钥等敏感信息始终保存在本地文件，不会出现在对话记录中。技能通过读取本地文件获取凭据，全程不落盘聊天记录。

---

## 适用场景

- 想拥有专属魔法链路，长期稳定访问 Google、YouTube、X 等
- 需要以纯净 IP 登录 Claude、ChatGPT 等 AI 服务（避免风控）
- 有一台VPS，想自己搭而不依赖机场

---

## 方案架构

```
电脑（Mac / Windows）        手机（安卓 / iOS）
    │                            │
    └─ Clash Verge ─┐      ┌─ Hiddify ─┘
                    └─► VLESS + Reality 加密隧道 ─►
                              境外 VPS（sing-box）
                                └─[可选 SOCKS5]─►
                                    上游纯净 IP（可选，用于 AI 服务登录）
                                      └─► Internet
```

电脑端和手机端共用同一套 VPS 服务端配置。手机端在电脑端验收通过后才配置，无需修改服务端。

### 为什么选 VLESS + Reality

| 方案 | 问题 |
|---|---|
| 裸 Shadowsocks | GFW 识别流量特征，封端口 5-10 分钟 |
| WireGuard | GFW 精准识别 WG 协议，UDP 严重 QoS |
| 裸 SOCKS5 | 跨境必封，无解 |
| **VLESS + Reality** ✅ | GFW 探测时看到真实 apple.com 响应，无法区分 |

---

## 前置要求

- **境外 VPS**：Ubuntu 20.04+ 推荐，至少 512MB 内存
  - 推荐服务商：搬瓦工（CN2 GIA 线路）、Vultr、Hetzner
  - ⚠️ 尽量不要用国内云厂商的海外节点（腾讯云轻量、阿里云），晚高峰丢包严重
- **电脑客户端**：[Clash Verge Rev](https://clashvergerev.com/install)（Mac / Windows 均有）
- **手机客户端**：[Hiddify](https://github.com/hiddify/hiddify-app/releases)（安卓下载 APK；iOS 需非国区 App Store）
- **AI CLI 工具**（以下任选其一，括号内为已实测组合）：
  - [Claude Code](https://claude.ai/code)（CC + kimi-k2.6）
  - [Kimi CLI](https://www.kimi.com/code)（kimi-cli + kimi-k2.6）
  - 其他任意支持 skill 调用的 CLI 工具
- **上游 SOCKS5**（可选）：用于以纯净 IP 登录 AI 服务

---

## 安装

### 方式一：Claude Code Marketplace

```bash
/plugin marketplace add 1126misakp/proxy-expert
/plugin install proxy-expert
/reload-plugins
```

### 方式二：手动安装

1. 下载本仓库（点 GitHub 页面右上角 Code → Download ZIP）
2. 解压缩后，将文件夹重命名为 `proxy-expert`（去掉末尾的 `-main`）
3. 将 `proxy-expert` 文件夹放入你所用 CLI 工具的 skills 目录：
   - Claude Code：`~/.claude/skills/`
   - Kimi CLI：`~/.kimi/skills/`
   - 其他 CLI：参考对应工具的文档找到 skills 目录，没有则手动创建
4. 重启 CLI 或执行 `/reload-plugins`

---

## 使用方法

安装后，建议专门创建一个文件夹作为根目录启动cli，然后对智能体说：

- `"帮我搭个梯子"`
- `"翻墙出问题了，谷歌上不去"`
- `"帮我在搬瓦工 VPS 上搭代理"`

技能会自动触发，引导你完成全流程。注意，不可以直接搭建手机端，必须先配置完电脑端才可选手机端搭建（因为和电脑端用同一套配置）

> **可选：放入项目规范文件**
> 技能包内附带一份 `CLAUDE.md`，将它复制到上述工作文件夹的根目录，可让智能体在该目录下严格遵守搭建工作流和最快路径排查原则，并在每次完成任务后自动维护验收报告。
> - 使用 **Claude Code**：文件名保持 `CLAUDE.md` 不变
> - 使用 **Kimi CLI**：将文件名改为 `agents.md`
> - 其他 CLI：参考对应工具的文档确认项目级规范文件的命名方式

### Clash Verge 推荐设置

为了获得最佳体验，建议按以下方式配置：

**代理模式选择"规则"**
在 Clash Verge 主界面点击「代理」，右上角选择「规则」。技能生成的 YAML 已内置完整分流规则，国内地址（B站、微信、银行等）自动直连，国外地址自动走代理，无需手动切换节点。

**开启 TUN 模式即可**
由于规则已做好分流，直接开启 TUN 模式（系统代理）即可达到"国内外各走各的流量"的效果：
- 国内网站：直连，速度不受影响
- 国外网站：走 VLESS+Reality 隧道，流畅翻墙
- 无需额外安装浏览器插件或手动配置 PAC

### Hiddify 推荐设置

手机端配置同样简单，导入 VLESS 链接后建议：

**首次调试先用「全局」模式**
确认 Google、GitHub 能打开后，再切换为「规则」+「绕过中国大陆」，实现国内 App 直连、外网走代理。

**关闭 IPv6 和 Fake DNS**
部分手机网络优先 IPv6，代理链路未必稳定支持，关闭可避免异常。Fake DNS 保持关闭，远程 DNS 用 `https://1.1.1.1/dns-query` 即可。

**国内 App 加入「绕过」**
抖音、微信、支付宝、淘宝等国内 App 建议加入分应用代理的绕过列表，避免风控接口异常。

### 完整工作流

```
Step 0    介绍方案架构和适用场景
Step 1    创建 proxy-setup-info.txt 模板，让你填写 VPS 信息
Step 1.5  VPS 连通质量检测（丢包率 + 延迟，不达标建议换 VPS）
Step 2    读取配置，确认部署方案
Step 3    处理 SSH 认证（密钥/密码均支持，全自动注入公钥）
Step 4    自动化部署 VPS 服务端（全程 SSH，约 5-10 分钟）
Step 5    生成 Clash Verge 客户端配置文件（含完整直连规则）
Step 6    自动化验收测试（端口、Reality 伪装、服务状态、Google/Claude/ChatGPT 连通）
Step 7    引导最终验收，自动生成 proxy-acceptance-report.md
Step 7.5  手机端配置（可选，安卓/iOS 用 Hiddify，共用同一套服务端配置）
Step 8    故障排查模式（先读验收报告，排查后追加记录）
```

---

## 自动化 vs 手动

| 步骤 | 自动化程度 | 说明 |
|---|---|---|
| VPS 连通质量检测 | ✅ 全自动 | ping 测丢包率和延迟，不达标自动建议换 VPS |
| VPS 服务端安装 sing-box | ✅ 全自动 | SSH 执行，无需手动 |
| 生成 Reality 密钥 | ✅ 全自动 | 密钥自动保存到 `.proxy-keys.txt` |
| 写服务端配置 | ✅ 全自动 | A/B/C 三种方案模板按选择填入 |
| systemd 服务 + BBR + fail2ban | ✅ 全自动 | |
| 密码认证转密钥认证 | ✅ 全自动 | sshpass 或 paramiko 自动注入公钥，无需进 VPS 终端 |
| 安装 Clash Verge | 🔶 手动 | 提供下载链接和安装指引 |
| Clash Verge 导入配置 | 🔶 半自动 | 自动生成 YAML，引导用户粘贴导入 |
| 安装 Hiddify（手机端） | 🔶 手动 | 安卓从 GitHub 下 APK；iOS 需非国区 App Store |
| 手机端导入配置 | 🔶 半自动 | 自动生成 VLESS 分享链接，引导 Hiddify 导入 |
| 验收测试（电脑+手机） | ✅ 全自动 | 含 Google / Claude / ChatGPT 连通性检查 |
| 生成验收报告 | ✅ 全自动 | 自动写入 `proxy-acceptance-report.md`，含手机端结果 |

---

## 技能文件结构

```
proxy-expert/
├── SKILL.md                    # 主工作流指令
├── evals/
│   └── evals.json              # 测试用例
└── references/
    ├── server-configs.md       # A/B/C 三种服务端配置模板
    ├── client-config.md        # Clash Verge YAML 模板
    └── troubleshooting.md      # 故障排查速查表
```

部署完成后，工作目录会多出以下文件：

```
your-project/
├── proxy-setup-info.txt        # VPS 信息（含敏感信息，勿上传 git）
├── .proxy-keys.txt             # 自动生成的密钥（隐藏文件，勿泄露）
├── clash-verge-config.yaml     # 电脑端配置（Clash Verge）
└── proxy-acceptance-report.md  # 验收报告和问题记录（含手机端结果，无敏感信息）
```

手机端不生成独立配置文件，直接通过 VLESS 分享链接导入 Hiddify。

---

## 三种部署方案

### 方案 A：VPS 直接出网
速度最快（享受 VPS 全带宽），适合不需要登录 AI 服务的普通翻墙需求。

### 方案 B：全流量走上游
所有流量通过上游 SOCKS5 出口，IP 纯净但速度受上游限制。

### 方案 C：混合路由（暂不推荐，当前存在瑕疵，正在完善中）
AI 站（Claude、ChatGPT）走上游纯净 IP，其他网站直连 VPS 享受高速。

---

## 验收标准

技能会自动执行以下检查，全部通过才算部署成功：

**VPS 直连测试（不经过 Clash）：**
- ✅ VPS 443 端口可达
- ✅ Reality 伪装测试返回 `HTTP/2 200` + `server: Apple`
- ✅ sing-box 服务状态 `active (running)`
- ✅ 日志无 ERROR，可见 GFW 探测被拦截记录

**代理连通测试（经过 Clash / Hiddify）：**
- ✅ Google 连通（收到任意 HTTP 响应 = 流量通；4xx 仅提示 challenge，不影响通过）
- ✅ Claude.ai 连通（同上）
- ✅ ChatGPT 连通（同上）
- ❌ 唯一失败条件：超时或 curl 报错（说明代理本身不通）

**手机端额外验收（可选，Step 7.5）：**
- ✅ Hiddify 连接成功，显示正确出口 IP
- ✅ 按调试顺序通过：1.1.1.1 → 8.8.8.8 → GitHub → Google → ChatGPT/Claude
- ✅ 国内 App 正常（抖音、微信等）
- ✅ 分应用代理已配置（国内 App 绕过，外网 App 走代理）

---

## 持续维护

部署成功后，技能自动在工作目录生成 `proxy-acceptance-report.md`，记录：

- VPS 信息和部署方案
- 电脑端验收测试结果（含 VPS 网络质量、七项连通测试）
- 手机端验收测试结果（如配置了手机端）
- 已知风险说明（流式输出偶发中断、AI 服务 IP 风控）
- 问题记录区（后续排障后追加，方便复盘）

后续如果梯子出现问题，技能会**先读取这份报告**了解历史状态，排查完成后追加问题原因和解决方案，形成持续可复用的维护日志。

---

## 已知风险说明

**Claude / ChatGPT 流式输出偶发中断**
SSE 长连接经代理时，偶发断流或输出停止。属 TCP 长连接经代理的正常特性，对功能影响有限，但体验略有下降。如频繁发生，检查本地→VPS 丢包率，或更换 VPS 服务商。

**AI 服务 IP 风控**
数据中心 IP 可能触发 Claude / GPT 的 IP 信誉检测（弹验证码或拒绝登录）。方案 C 通过上游 SOCKS5 缓解此风险。

---

## 常见问题

**Q：Windows 也支持吗？**
Windows 10/11 内置 OpenSSH，Claude Code 的 Bash 工具可直接执行 `ssh` 命令，服务端部署完全自动化。客户端使用同款 Clash Verge Rev（Windows 版），YAML 配置完全一致。

**Q：密码认证能用吗？**
可以，全自动处理。技能会在本地生成临时 SSH 密钥，通过 sshpass 或 Python paramiko 自动将公钥注入 VPS，用户无需打开 VPS 终端执行任何命令。

**Q：不需要上游也能用吗？**
可以，选方案 A 直接出网即可，省去上游配置。

**Q：为什么不推荐 WireGuard？**
GFW 完全识别 WireGuard 协议特征，UDP 出境会被严重 QoS，不适合翻墙使用。

**Q：VPS 网络质量怎么算合格？**
丢包率 < 5%、平均延迟 < 300ms 为可接受范围。丢包超过 10% 时技能会强烈建议更换 VPS，你更换后技能会自动重测确认。

**Q：验收报告有什么用？**
它是梯子的"健康档案"——记录初始验收状态和历史故障处理。下次梯子出问题时，技能会先读这份报告再排查，避免从头问诊。

**Q：手机端能和电脑端同时用吗？**
可以，两者共用同一套 VPS 服务端配置。电脑端用 Clash Verge，手机端用 Hiddify，互不干扰。服务端带宽足够的情况下，多台设备同时使用没有问题。

**Q：iOS 怎么下载 Hiddify？**
Hiddify 在国区 App Store 没有上架，需要切换到非国区 Apple ID（美区、港区、日区等均可）。如果没有非国区账号，可以在 Apple ID 官网免费切换地区（无需绑定付款方式），或使用已有的非国区账号。

**Q：手机端连不上但电脑端正常，怎么办？**
这是常见情况。电脑端正常说明 VPS 服务端配置正确，手机端问题通常出在：安卓 VPN 权限、Private DNS、Hiddify TUN 接管、Chrome 安全 DNS 或蜂窝网络链路。技能会按 troubleshooting.md 中的"状况 6-9"逐层帮你排查，不要一上来就改 Reality 参数。

---

## 免责声明

本技能仅供个人学习和合法使用。请遵守所在地区的法律法规。

---

## License

MIT
