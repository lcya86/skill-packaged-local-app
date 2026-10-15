# skill-packaged-local-app

**给本机 H5 应用配一份 skill** —— 一份能被 Agent 直接加载的「创建 + 维护」手册（`SKILL.md` + `references/`）。

一句话说清它是什么：普通 H5 只是一个页面；这套做法交付的是 **H5 应用 + 一份配套 skill**——
应用给它自己的人用，skill 给下一个 Agent 用。没有配套 skill 的应用，就是一堆没人敢改的代码。

**适用判据**：数据只在本机、需要「Agent 能改 + 人能看」、界面是单页 H5（手机能加到桌面当 App）、生命周期以年计。
典型成品：持仓/资产台账、家庭账本、装修报价与预算、读书/训练记录、家庭设备面板。

**不适用**：一次性脚本；纯 CLI 无界面；必须多用户/云同步；要实时协作的；依赖原生能力（推送、后台定位、重度离线）的。

## 覆盖的完整生命周期

| 阶段 | 内容 | 文件 |
| --- | --- | --- |
| **C0 需求采集** | 苏格拉底式三轮提问（场景判据 / 数据血统 / 边界与非目标）+ 复述纠错 + 需求文档四张表 | `references/creation-flow.md` |
| **C1 原型先行** | 单文件假数据原型、移动端 H5 形态约定、「用户改口才算确认」 | `references/creation-flow.md` |
| **C2 开发** | 骨架分层、九条不变量、F0→F6 分阶段交付、配套 skill 的出货清单 | `SKILL.md`、`references/architecture-blueprint.md` |
| **C3 验收与交付** | 验收纪律五条、隔离实例、真机回放、默认不变对撞；交付到手机（隧道 + 口令 + 安卓/iOS/鸿蒙加桌面） | `references/verification-recipes.md`、`references/mobile-delivery.md` |
| **C4 长期迭代** | 五类变更风险表、六步安全改动工序、变更单模板、什么时候必须停下来重构 | `references/change-workflow.md` |

## 它解决什么问题

「本地小工具」通常死在七件事上，这份手册逐条给了可照抄的做法：

| 死法 | 对策 |
| --- | --- |
| 一开始没把需求问清楚，做完发现不是用户要的 | C0/C1：**先问后做、先原型后实现**；原型确认的判据是「用户改口」 |
| 三个地方各算一遍，数字对不上，用户谁都不信 | 不变量 1：**一份算法，两个消费方**（页面直接加载 CLI 那份文件） |
| 数据被写坏、被测试写脏、被重装冲掉 | 不变量 2/6/8：写操作唯一通道 + 写前备份 + 代码与数据分家 |
| 加个开关就把老行为改了，升级当天全线报错 | 不变量 3/4：**默认必须等于旧行为**；事实字段不随开关变形 |
| 改一个需求把别的功能改坏 | C4：冻结基线 + 双向对撞 + 断言只多不少——**防「改坏」靠闸门，不靠小心** |
| 服务跑在自己电脑上，出门就看不了 | C3：本地服务 → 公网隧道 → 口令门禁 → 手机桌面图标（含 iOS/安卓/鸿蒙步骤） |
| 同一类 bug 反复复发（改了又犯） | §十一 **复发防治**：同类缺陷第 2 次出现就停止打补丁，改成结构性消除 |

## 内容

| 文件 | 讲什么 |
| --- | --- |
| `SKILL.md` | 创建链 C0→C4 / 骨架 / 九条不变量 / F0→F6 / 验收纪律 / 交付到手机 / 变更 workflow / 维护手册 / 坑库 / 复发防治 / 交付礼仪 |
| `references/creation-flow.md` | C0/C1：苏格拉底提问题库与追问范本、需求文档四张表、原型规格与移动端形态约定、配套 skill 出货清单 |
| `references/change-workflow.md` | C4：五类变更的风险与必跑验证、六步安全改动工序、变更单与回滚、何时停下来重构 |
| `references/mobile-delivery.md` | 交付到手机：quick/named tunnel、口令门禁三档、manifest 与图标、安卓/iOS/鸿蒙「添加到桌面」、交付卡片与二维码 |
| `references/architecture-blueprint.md` | 目录骨架、数据契约与 API 契约模板、验收基线、何时该拆 skill |
| `references/verification-recipes.md` | 隔离实例、真键盘回放、公网侧五连、对撞比对、写脏后的还原流程 |
| `references/maintenance-playbook.md` | 本地补丁清单模板、进程与端口、隧道与口令门禁、数据血统、分发边界 |
| `references/pitfalls-catalog.md` | 坑库全量（现象 → 根因 → 修法 → 通用规则，含「复发型坑」分类） |

## 安装

Agent Skills 通用（Claude Code / Codex / 支持 `SKILL.md` 的宿主）：

```bash
npx skills add https://github.com/lcya86/skill-packaged-local-app --skill skill-packaged-local-app
```

或直接放进宿主的技能目录：

```bash
git clone https://github.com/lcya86/skill-packaged-local-app ~/.claude/skills/skill-packaged-local-app      # Claude Code
git clone https://github.com/lcya86/skill-packaged-local-app ~/.hermes/skills/software-development/skill-packaged-local-app   # Hermes
```

装好后，Agent 在你说「做个本机的看板/台账/工具，并让它自带一份 skill」时会自动加载它。

## 怎么用最省时间

1. **先从 C0 问需求**：问出场景与判据 → 再问数据血统 → 最后问「绝对不要什么」；每轮结束压成一句话让他纠错。
2. **用原型定稿**：单文件、假数据（标 DEMO）、手机视口可点；用户改口之后才写正式代码。
3. **从 F0 开始**：先写数据契约（含一张人手算的期望值表），别急着写页面。
4. **每个新功能走一遍成对文档**：`DESIGN-<功能>.md` → 实现 → `INTEGRATION-TEST-<功能>.md`。
5. **验证永远在隔离实例上**：临时数据目录 + 独立端口；真实数据只用于只读比对。
6. **改完必做「默认不变」对撞**：拿改代码前冻结的输出，剔掉新增键后深度比对。
7. **要上手机就按 C3 出交付卡片**：地址 + 口令 + 三步加桌面 + 打不开时的自查 + 别转发别截图。

## 来源

提炼自一个自用的本机应用（持仓/再平衡看板）在 2026 年的持续迭代：首版上线 → 四次口径类缺陷复发 →
加装远程访问口令 → 把运维沉淀成补丁清单。所有条目都对应一次真实事故；业务口径类内容刻意剔除，
只保留**可迁移的架构与纪律**。

## 许可

MIT，见 [LICENSE](LICENSE)。
