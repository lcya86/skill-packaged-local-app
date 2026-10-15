# skill-packaged-local-app

**把本地应用做成 Agent Skill** —— 一份可被 Agent 直接加载的架构与维护手册（`SKILL.md` + `references/`）。

适用判据：数据只在本机、需要「Agent 能改 + 人能看」、界面要能装进手机、生命周期以年计。
典型成品：持仓/资产台账、家庭账本、装修报价与预算、读书/训练记录、家庭设备面板。

不适用：一次性脚本；纯 CLI 无界面；必须多用户/云同步；要实时协作的。

## 它解决什么问题

「本地小工具」通常死在四件事上，这份手册逐条给了可照抄的做法：

| 死法 | 手册里的对策 |
| --- | --- |
| 三个地方各算一遍，数字对不上，用户谁都不信 | 不变量 1：**一份算法，两个消费方**（页面直接加载 CLI 那份文件） |
| 数据被写坏、被测试写脏、被重装冲掉 | 不变量 2/6/8：写操作唯一通道 + 写前备份 + 代码与数据分家 |
| 加个开关就把老行为改了，升级当天全线报错 | 不变量 3/4：**默认必须等于旧行为**；事实字段不随开关变形 |
| 同一类 bug 反复复发（改了又犯） | §七 **复发防治**：同类缺陷第 2 次出现就停止打补丁，改成结构性消除 |

## 内容

| 文件 | 讲什么 |
| --- | --- |
| `SKILL.md` | 骨架 / 九条不变量 / F0→F6 分阶段交付 / 验收纪律 / 维护手册 / 坑库 / 复发防治 / 交付礼仪 |
| `references/architecture-blueprint.md` | 目录骨架、数据契约模板、API 契约模板、验收基线、何时该拆 skill |
| `references/verification-recipes.md` | 隔离实例、真键盘回放、公网侧五连、对撞比对、写脏后的还原流程 |
| `references/maintenance-playbook.md` | 本地补丁清单模板、进程与端口、隧道与口令门禁、数据血统、分发边界 |
| `references/pitfalls-catalog.md` | 坑库全量（现象 → 根因 → 修法 → 通用规则，含「复发型坑」分类） |

## 安装

Agent Skills 通用（Claude Code / Codex / 支持 `SKILL.md` 的宿主）：

```bash
npx skills add <this-repo-url> --skill skill-packaged-local-app
```

或直接放进宿主的技能目录：

```bash
git clone <this-repo-url> ~/.claude/skills/skill-packaged-local-app      # Claude Code
git clone <this-repo-url> ~/.hermes/skills/software-development/skill-packaged-local-app   # Hermes
```

装好后，Agent 在你要「做个本机看板/台账/工具并让它变成 skill」时会自动加载它。

## 怎么用最省时间

1. **从 F0 开始**：先写数据契约（含一张人手算的期望值表），别急着写页面。
2. **每个新功能走一遍成对文档**：`DESIGN-<功能>.md` → 实现 → `INTEGRATION-TEST-<功能>.md`。
3. **验证永远在隔离实例上**：临时数据目录 + 独立端口；真实数据只用于只读比对。
4. **改完必做「默认不变」对撞**：拿改代码前冻结的输出，剔掉新增键后深度比对。

## 来源

提炼自一个自用的本机应用（持仓/再平衡看板）在 2026 年的持续迭代：首版上线 → 四次口径类缺陷复发 →
加装远程访问口令 → 把运维沉淀成补丁清单。所有条目都对应一次真实事故；业务口径类内容刻意剔除，
只保留**可迁移的架构与纪律**。

## 许可

MIT，见 [LICENSE](LICENSE)。
