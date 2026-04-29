# proxy-expert — Claude Code 搭梯子专家技能

一个端到端的翻墙搭建 Claude Code Skill，基于 **VLESS + Reality + sing-box** 方案，支持 Mac 和 Windows，自动化 VPS 服务端部署，引导客户端配置，最终通过验收标准确认成功。

**实测效果**：940 Mbps 下载速度，抗 GFW 主动探测，长期稳定运行。

---

## 适用场景

- 想翻墙访问 Google、YouTube、GitHub 等
- 需要以纯净 IP 登录 Claude、ChatGPT 等 AI 服务（避免风控）
- 有一台境外 VPS，想自己搭而不依赖机场

---

## 方案架构

```
你的设备（Mac / Windows）
  └─[VLESS+Reality 加密隧道，伪装成访问 apple.com 的 HTTPS]─►
      境外 VPS（sing-box）
        └─[可选 SOCKS5]─►
            上游纯净 IP（可选，用于 AI 服务登录）
              └─► Internet
```

### 为什么选 VLESS + Reality

| 方案 | 问题 |
|---|---|
| 裸 Shadowsocks | GFW 识别流量特征，封端口 5-10 分钟 |
| WireGuard | GFW 精准识别 WG 协议，UDP 严重 QoS |
| 裸 SOCKS5 | 跨境必封，无解 |
| **VLESS + Reality** ✅ | GFW 探测时看到真实 apple.com 响应，无法区分 |

---

## 安装

### 方式一：Claude Code Marketplace

```bash
/plugin marketplace add 1126misakp/proxy-expert
/plugin install proxy-expert
/reload-plugins
```

### 方式二：本地安装 `.skill` 文件

下载 [proxy-expert.skill](https://github.com/1126misakp/proxy-expert/releases) 后：

```bash
/plugin install /path/to/proxy-expert.skill
/reload-plugins
```

---

## 使用方法

安装后，直接对 Claude Code 说：

- `"帮我搭个梯子"`
- `"翻墙出问题了，谷歌上不去"`
- `"帮我在搬瓦工 VPS 上搭代理"`

技能会自动触发，引导你完成全流程。

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
| 验收测试 | ✅ 全自动 | 含 Google / Claude / ChatGPT 连通性检查 |
| 生成验收报告 | ✅ 全自动 | 自动写入 `proxy-acceptance-report.md` |

---

## 前置要求

- **境外 VPS**：Ubuntu 20.04+ 推荐，至少 512MB 内存
  - 推荐服务商：搬瓦工（CN2 GIA 线路）、Vultr、Hetzner
  - ⚠️ 不要用国内云厂商的海外节点（腾讯云轻量、阿里云），晚高峰丢包严重
- **本地客户端**：[Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev/releases)（Mac / Windows 均有）
- **Claude Code**：已安装并登录
- **上游 SOCKS5**（可选）：用于以纯净 IP 登录 AI 服务

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
├── clash-verge-config.yaml     # Clash Verge 客户端配置
└── proxy-acceptance-report.md  # 验收报告和问题记录（无敏感信息）
```

---

## 三种部署方案

### 方案 A：VPS 直接出网
速度最快（享受 VPS 全带宽），适合不需要登录 AI 服务的普通翻墙需求。

### 方案 B：全流量走上游
所有流量通过上游 SOCKS5 出口，IP 纯净但速度受上游限制。

### 方案 C：混合路由（推荐）
AI 站（Claude、ChatGPT）走上游纯净 IP，其他网站直连 VPS 享受高速。

---

## 验收标准

技能会自动执行以下检查，全部通过才算部署成功：

**VPS 直连测试（不经过 Clash）：**
- ✅ VPS 443 端口可达
- ✅ Reality 伪装测试返回 `HTTP/2 200` + `server: Apple`
- ✅ sing-box 服务状态 `active (running)`
- ✅ 日志无 ERROR，可见 GFW 探测被拦截记录

**代理连通测试（经过 Clash）：**
- ✅ Google 连通（2xx/3xx 状态码）
- ✅ Claude.ai 连通（2xx/3xx 状态码）
- ✅ ChatGPT 连通（2xx/3xx 状态码）

---

## 验收报告

部署成功后，技能自动在工作目录生成 `proxy-acceptance-report.md`，记录：

- VPS 信息和部署方案
- 所有验收测试结果（含 VPS 网络质量、七项连通测试）
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

---

## 免责声明

本技能仅供个人学习和合法使用。请遵守所在地区的法律法规。

---

## License

MIT
