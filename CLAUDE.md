# proxy-expert 项目规范

本目录用于 VLESS+Reality+sing-box 梯子的端到端部署与维护。
智能体必须调用 `proxy-expert` skill，并严格遵守以下规范。

---

## 工作流规范

**新建部署**：必须按 SKILL.md 的 Step 0 → 1 → 1.5 → 2 → 3 → 4 → 5 → 6 → 7 顺序执行，不得跳步、合并步骤或自行简化。每步完成打印进度，出错立即停下报告，不强行推进。

**故障排查**：用户带着问题来时，走 Step 8 流程，先读 `proxy-acceptance-report.md` 和 `proxy-setup-info.txt` 了解现状，再按 `troubleshooting.md` 对应状况排查。

---

## 故障排查：最快路径优先

排查时优先尝试最高频、最简单的修复，不要一上来就让用户重装或重配：

1. Clash 系统代理/节点是否已开启？
2. sing-box 服务是否 active？（`systemctl is-active sing-box`）
3. 443 端口是否可达？（`nc -z -w 5 {VPS_IP} 443`）
4. 日志是否有 ERROR？（`tail -20 /var/log/sing-box/sing-box.log`）
5. 以上都正常才进入深层排查（配置、密钥、防火墙等）

每步检查出结果后立即判断，能快速定位则直接给出操作，不要让用户做不必要的诊断步骤。

---

## 验收报告：强制检查

以下两种情况**完成后必须检查一次验收报告**：

**A. 完成新建部署（Step 7 通过）**
→ 在当前目录生成 `proxy-acceptance-report.md`（按 SKILL.md Step 7 的模板）

**B. 完成故障排查/问题解决**
→ 在 `proxy-acceptance-report.md` 末尾的"问题记录"区追加本次记录（Step 8.4 格式）

如果 `proxy-acceptance-report.md` 不存在且用户是带故障来的（非新建部署），询问用户是否需要补建，或直接在解决问题后新建一份（填写已知信息，未测试项标为"未知"）。

---

## 敏感信息规范

- **PrivateKey** 只写服务端 `/etc/sing-box/config.json`，不得在对话中展示、不得写入 `proxy-acceptance-report.md`
- **SSH 密码**：只从 `proxy-setup-info.txt` 读取，不在对话中让用户粘贴，用完后不复述
- **每次 SSH 操作前**重新读取 `proxy-setup-info.txt`（用户可能中途修改了 IP、端口或密钥路径）

---

## 本目录文件说明

| 文件 | 说明 |
|---|---|
| `proxy-setup-info.txt` | 用户配置（含敏感信息，不要提交 git）|
| `.proxy-keys.txt` | 自动生成的密钥（隐藏文件，不要泄露）|
| `clash-verge-config.yaml` | 电脑端 Clash Verge 配置 |
| `proxy-acceptance-report.md` | 验收报告与问题记录（无敏感信息）|
