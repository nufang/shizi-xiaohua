# 糖果城堡识字页 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 做一个双击就能打开的糖果城堡识字页，让幼儿园小朋友看图点汉字，过关后给小公主选发饰或裙子。

**Architecture:** 全部写在 `index.html`。字库、进度、出题、朗读、画面是同一文件里的独立函数。进度存在 `localStorage` 键 `shizi-castle`。规格规定这一版不做自动测试，纯规则用一次 Node 检查，画面用手动清单验收。

**Tech Stack:** 一个静态 HTML 文件，内联 CSS 和 JavaScript。朗读用浏览器 `speechSynthesis`，语言 `zh-CN`。

## Global Constraints

- 成品路径是 `C:\Users\zhangq-bm\shizi-xiaohua\index.html`，样式和脚本不拆文件。
- 四关从第一次打开就能进：小动物、花朵、水果、家人，每关 8 个字。
- 每次一张图、三个大字。题目出现时朗读这个字一次，点对后再朗读一次。
- 点错轻轻晃一下，留在这一题。
- 画面是糖果城堡：亮粉、紫色、金色。
- 认字时不换衣服。第一次过完 8 个字才打开衣橱。发饰和裙子分开放。
- 必须点一件才离开衣橱。点已经穿着的那一件表示继续穿它。
- 已过关的关再玩不发新衣服，也不打开衣橱。
- 四关都过关后，礼花出现在城堡大门上。
- 认字页可以返回城堡。中途返回不记住题号。
- 衣橱未选完就刷新时，下次仍停在衣橱。
- 没有中文语音时仍可点字，并显示「这台设备读不出中文，仍然可以点字。」
- 朗读没有正常结束时，1 秒后进入下一题。
- 保存失败或存档损坏时不弹错误框。
- 不做拼音、笔顺、账号、联网字库、音效文件和自动测试套件。

---

### Task 1: 字库和纯规则

**Files:**
- Create: `index.html`
- Test: 用 Node 载入脚本中的纯函数做一次检查，不另存测试文件

**Interfaces:**
- Consumes: 无
- Produces: `LEVELS`、`CHAR_ART`、`ITEM_ART`、`emptyProgress()`、`loadProgress()`、`saveProgress(state)`、`shuffle(list)`、`makeOptions(level, answer)`、`applyLevelClear(progress, level)`、`applyEquip(progress, item)`

- [x] **Step 1: 写纯规则**

`emptyProgress` 返回 `clearedLevels: []`、`ownedItemIds: []`、`equipped: { hair: null, dress: null }`、`wardrobePending: false`。

`makeOptions(level, answer)` 从同一关其余字里抽 2 个，和正确答案一起打乱，返回 3 项。

`applyLevelClear` 在第一次过关时加入 3 件奖励、写入关卡编号、把 `wardrobePending` 设为 `true`。已经过关则原样返回且 `openWardrobe` 为 `false`。

`applyEquip` 只改对应的 `hair` 或 `dress`，并把 `wardrobePending` 设为 `false`。

`loadProgress` 遇到抛错、空值、坏 JSON、未知编号、发饰和裙子放错位置时，丢掉坏字段或整份存档。`wardrobePending` 仅在仍有已拥有衣服时保留。

- [x] **Step 2: 用 Node 检查纯规则**

运行内联 Node 脚本，预期打印 `pure-rules-ok`。

- [x] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add candy-castle literacy page rules and screens."
```

### Task 2: 城堡、认字、衣橱和朗读

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: Task 1 的函数和字库
- Produces: `startLevel(id)`、`onPick(char)`、`finishLevel()`、`equipItem(id)`、`backHome()`、`speak(text)`、`render()`、`boot()`

- [x] **Step 1: 画出三块画面**

城堡大门有小公主、四扇门和已过关金星。四关都过关时显示礼花。认字页有返回、8 个进度点、图案和三个大字。衣橱只显示已拥有的衣服，没有别的离开方式。

- [x] **Step 2: 接上对错、朗读和保存**

点错只晃动该按钮。点对后锁定点击，再读一遍，然后进入下一题。8 题完成时调用 `applyLevelClear` 并保存。衣橱点选调用 `applyEquip` 并保存。`speak` 在 1 秒没有正常结束时放开锁定。返回城堡会增加 `quizEpoch`，使旧朗读不能推进新题目。

- [x] **Step 3: 按规格清单核对**

打开 `index.html`，核对点错留在本题、第一次过关进入衣橱、刷新后衣服和金星还在、再玩不发衣服、手机宽度下三个大字可点。

- [x] **Step 4: Commit**

包含 `index.html` 与本计划。提交说明写明为什么：让小朋友能离线看图认字并保存装扮。
