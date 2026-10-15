# 验收配方（可复制粘贴）

原则：**被测实例必须与真实数据隔离**；**证据必须来自真实路径**（真键盘、真点击、公网侧）。

## 1. 隔离实例（所有 UI/接口验证的起点）

```bash
APP=~/path/to/app-skill
DIR=/tmp/app-verify-$(date +%s)
mkdir -p "$DIR"
APP_DATA_DIR="$DIR" node "$APP/scripts/cli.mjs" init --demo   # 或用 fixture 拷一份假数据
APP_DATA_DIR="$DIR" node "$APP/scripts/server.mjs" --port 8802 # 用 terminal(background=true) 起
curl -s localhost:8802/api/health                              # 起来了吗
# …在这里跑一切会“写”的验证…
kill <pid>; rm -rf "$DIR"                                      # 收尾，别留孤儿
```

- 独立端口避免撞到生产实例；独立数据目录避免污染；收尾时确认端口已释放。
- **真实数据只有两种用途**：只读比对、以及（批准后的）正式操作。

## 2. 真键盘回放（验证输入类缺陷的唯一有效手段）

`browser-use` CLI（`--json` 是**全局开关，必须写在子命令之前**）：

```bash
browser-use open http://127.0.0.1:8802/
browser-use state                       # 从 _raw_text 里正则捞元素索引：\[\d+\]<input id=…
browser-use input <索引> "20.5"         # 清空后真实输入（逐键触发事件）
browser-use keys 5                      # 单键（复现“打一位就脱焦”）
browser-use eval "(function(){…; return JSON.stringify(x);})()"   # 同步 IIFE，返回 JSON
browser-use screenshot --full /tmp/page.png                        # 传 path 才落文件
```

- `eval` **不 await Promise**（async IIFE 只会回 `{}`）→ 要读服务端就用**同步 XHR**：
  `var x=new XMLHttpRequest(); x.open('GET','/api/state',false); x.send(); JSON.parse(x.responseText)`。
- 打字后**不要先跑 `state` 再查焦点**：`state` 会移动焦点，你会读到假阴性。查焦点要紧接打字那一步。
- 只在**隔离实例**上真点页面；真页面上任何输入都会即时落库。

## 3. 公网侧验收（交付链接前的五连）

```bash
U="https://<随机域名>.trycloudflare.com"
curl -sS -m 25 -o /dev/null -w '%{http_code}\n' "$U/"                       # 无口令：期望 401
curl -sS -m 25 -o /dev/null -w '%{http_code}\n' "$U/api/state"              # 无口令：期望 401
curl -sS -m 25 -i "$U/?token=<口令>" | grep -i 'set-cookie'                  # 期望 302 + HttpOnly cookie
curl -sS -m 25 -o /dev/null -w '%{http_code}\n' -H 'Cookie: <cookie>' "$U/"             # 期望 200
curl -sS -m 25 -o /dev/null -w '%{http_code}\n' -H 'Cookie: <cookie>' "$U/app.js"       # 静态资源也要 200
```

补齐三件事：

- **本机回环仍免口令**：`curl -s 127.0.0.1:<port>/api/health` → 200（保证 CLI 探活/进程复用不受影响）。
- **页面引用的静态资源逐个探**（相对路径先补 `/`，否则 curl 会把 `域名app.css` 当主机名误报）。
- **公网实测优先于 devtools**：外部抓取器不跑 JS，HTML 面只能看到模板占位，别据此判“页面坏了”。

## 4. 「默认行为不变」对撞比对

```bash
# 1) 从备份里挑一帧“改代码前”的真实数据（别用当前 state：用户随时会改）
ls -lt <data_dir>/backups | head
# 2) 用旧代码与新代码分别对同一帧跑只读命令，输出落盘
# 3) 剔掉新增键后深度比对（jq / python），而不是只看断言数
```

- 对撞要**双向都跑**：新代码 + 旧数据、旧代码 + 新数据（第二种最容易漏）。

## 5. 数据被写脏后的还原（必须背下来）

1. 找快照：`<data_dir>/backups/` 里时间戳最新那份（写入前自动生成）。
2. **逐字段 diff** 当前 vs 快照（不只盯你改的字段：列表类、设置类、现金类都过一遍），界定污染面。
3. 用 **CLI** 还原（例：`target:set <code> <原值>` / `dca:set <code> <原值>`），**绝不手改 state.json**。
4. 还原后**再 diff 一次**确认一致；说明 `rev` 会继续涨（写入已发生，属正常）。
5. 残项如实讲（例：`dca:set 0` 只把 base 置 0、删不掉那个键；页面显示“未设置”）——别假装全清干净了。
6. 复盘一句：**为什么这次会写脏**（真机测试没隔离），把结论写进验收纪律。

## 6. 截图取证

```bash
browser-use screenshot --full /tmp/<app>-<view>.png   # 传 path 才落文件
```

- 发出去之前**自己先看一眼**；放大读小字用 `region=[x1,y1,x2,y2]`，坐标是**原图像素**
  （视觉输入会被降采样，看到的小图坐标要乘缩比再传）。
- full-page 截图会有顶栏重复的拼接痕迹（sticky 元素的已知现象）——汇报时说明，别让用户以为页面坏了。

## 7. 环境隔离与「看不见的污染」

- **长驻 shell 会继承上一次的 `export`**：同一个后端会话里，sub-agent 或上一步设的环境变量会残留，
  让「未配置分支」的验证得出反结论（实测：`export APP_DATA_DIR=…` 之后，未配置分支误报 `source:none`）。
  → 每个验证用**独立子 shell**，或写成 `APP_DATA_DIR=… cmd` 的显式前缀，别依赖环境变量的现状。
- **冻结基线里含「当天日期」的对照物必挂**：Golden/对撞 fixture 出现日期就要归一化
  （`20\d{2}-\d{2}-\d{2}` → `DATE`）或注入固定时钟，否则过一天就红。
- **对撞要双向跑**：新代码 + 旧数据、旧代码 + 新数据，两向都可能出问题。
- **用户说「链接挂了」= 先双向举证，别急着重开**：① 本机 → 公网域名连续 10~15 次探测；
  ② 换一台不在用户网络里的机器再抓一次（这条才证明“对公网可达”）；③ DNS 分别问两个公共解析；
  ④ 从服务自己的写入记录（rev / 备份 mtime）推「最后一次是好的」时间。
  **隧道好着的时候重开 = 换掉一个本来可用的域名，问题原样复发。**
