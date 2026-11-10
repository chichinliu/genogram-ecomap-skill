---
name: "genogram-ecomap"
description: "依使用者描述的家庭關係，先畫家系圖（Genogram）再畫生態圖（Ecomap），符號依社工實務標準。凡提到家系圖、生態圖、家庭圖、genogram、ecomap、個案家庭評估，或貼上一段家庭成員／個案背景敘述要求畫圖時使用。"
---

# 家系圖 × 生態圖繪製

把一段家庭敘述轉成兩張標準化評估圖：**先家系圖，後生態圖**，最後附一段評估觀察。圖是給社工、諮商、醫療評估用的工作文件，所以「符號正確、忠於敘述、不腦補」比好看重要。

## 流程總覽

1. 讀敘述 → 整理成員與關係（先結構、後關係、再註記）
2. 排版 → 寫成 `case.json`
3. 存下本文件最後的 `genoeco.py`，執行產生 `genogram` 與 `ecomap`（SVG＋PNG）
4. 打開 PNG 目視檢查，有重疊或穿越就調整座標重跑
5. 交付兩張圖，並寫出評估觀察與待補資訊

## 步驟 1：整理資訊

依這個順序從敘述中抽取，因為客觀結構錯了，後面的關係線都會跟著錯：

1. **結構（客觀事實）**：誰是案主、每位成員的性別／年齡／存歿、婚姻或伴侶狀態（結婚、同居、分居、離婚）、親子與手足、領養、雙胞胎、懷孕、目前同住的人。
2. **關係（情感互動）**：親密、過度親密、疏離、衝突、衝突又親密、溝通中斷、關係惡化。
3. **註記**：職業、教育、重大事件、慢性病、身心障礙，寫在符號下方（一行、約 8 字內；更長的內容放在回覆文字，不要塞進圖）。
4. **外部系統（生態圖用）**：正式資源（學校、職場、社福機構、醫療院所、法院、里辦公處）與非正式資源（親戚、朋友、鄰居、宗教團體、社團），以及每個系統的重要性、親疏、關係品質、資源流向。

原則：

- **至少涵蓋三代**（案主、父母、祖父母），以便觀察跨世代模式；但敘述沒提到的人不要虛構，改列在回覆的「待補資訊」。
- 敘述沒說的關係不畫關係線；年齡不明就不填；性別不明用三角形（`sex: "U"`）。
- 只有「案主是誰」無法判斷時才先問一次；其他缺漏先照已知資訊畫，並列出待補問題，讓使用者可以帶回去問個案。
- 敘述完全沒有外部系統資訊時，不要自己編生態圖：先交付家系圖，並列出要補問的外部系統問題。
- 個案資料敏感：標籤預設用稱謂（父親、大兒子），除非使用者自己給了姓名並要求顯示。

## 步驟 2：符號規範

| 項目 | 畫法 | JSON 寫法 |
|---|---|---|
| 男／女 | 方形／圓形 | `sex: "M"` / `"F"` |
| 懷孕或性別不明 | 三角形 | `sex: "U"` |
| 案主 | 符號填灰＋粗框（預設）；使用者要雙層框時改雙框 | `client: "id"`；`client_style: "fill"` 或 `"double"` |
| 已故 | 符號內畫 × | `deceased: true` |
| 年齡 | 寫在符號內 | `age: 32` |
| 身心障礙／慢性病 | 符號內直線對分／右上四分之一框 | `marks: ["disabled"]` / `["chronic"]` |
| 結婚 | 兩人下方實線相連 | `type: "married"` |
| 同居 | 虛線相連 | `type: "cohabit"` |
| 分居／離婚 | 連線上一條斜線／兩條斜線 | `"separated"` / `"divorced"` |
| 子女 | 由伴侶線垂直往下，手足由左到右＝長到幼 | `children: [...]` |
| 領養 | 子女線為虛線 | `adopted: [...]` |
| 雙胞胎 | 由同一點斜分出 | `twins: [["a","b"]]` |
| 同住範圍 | 實線框圈起 | `household: [...]` |

成員間關係線（`relations[].type`）：

| type | 意義 | 線型 |
|---|---|---|
| `positive` | 正向關係 | 單實線 |
| `close` | 關係親密 | 雙實線 |
| `fused` | 過度親密 | 三實線 |
| `distant` | 關係疏離 | 細點線 |
| `conflict` | 衝突緊張 | 鋸齒線 |
| `conflict_close` | 衝突又親密 | 雙線夾鋸齒 |
| `cutoff` | 溝通中斷／斷絕 | 中間斷開 ┤├ |
| `worsening` | 關係惡化 | 實線加斜紋 |

## 步驟 3：排版規則

程式不會自動排版，位置由你用格線座標指定：`gen` 是第幾列（0＝最上一代，可用小數如 2.2 把某列再往下放），`x` 是欄位（可用小數如 1.5；同列兩人至少相差 1）。排得好不好決定圖讀不讀得懂，請照這些原則：

- 同一代放同一列；子女的 `gen` 一定大於父母。
- 夫妻相鄰，**男左女右**；手足由左到右依長幼。
- 子女群放在父母連線中點的正下方附近，線才不會歪斜。
- 父系家族放左半、母系放右半；案主的父母各自是手足列的一員時，把兩人排在各自手足列中**彼此靠近的那一端**，中間不要夾其他人，連線才不會穿過別人。
- 同一列有兩條伴侶線會上下相撞時，把其中一條設 `level: 1`（往下錯開一階）；子女垂直線要偏離中點時用 `drop_x` 指定欄位。
- 同住成員盡量排在一起，同住框才不會圈到非同住的人。
- 關係線是兩人之間的直線：兩人中間若隔著第三人，調整座標讓它不要穿過。
- 再婚、多段關係：每段關係各寫一筆 `unions`，前任排在當事人外側。
- 單親或另一方不明：`unions` 只寫 `a`，子女線直接從該人往下。

## 步驟 4：生態圖規則

生態圖中央大圓放核心家庭的小家系圖，四周是外部系統：

- `eco.center`：要放進中央的成員 id（通常是同住的核心家庭；省略時用 `household`）。
- **中央的成員必須用親屬線連成一張圖。** 同住成員之間的連接者不住在家裡時（例如隔代教養中的父母、已故的祖父），也要把他們放進 `eco.center`，否則祖孫之間不會有任何線。程式偵測到中央斷開時會印出 ⚠ 警告。
- 每個外部系統一筆 `eco.systems`：

| 欄位 | 值 | 意義 |
|---|---|---|
| `name` | 文字 | 系統名稱 |
| `size` | `L` / `M` / `S` | 圓越大＝對案主影響力或重要性越大 |
| `distance` | `near` / `mid` / `far` | 離中心越近＝越親近、越便利 |
| `line` | `strong` 粗實線＝強烈正向；`normal` 實線＝正向支持；`weak` 虛線＝薄弱疏離；`stress` 鋸齒線＝緊張衝突壓力 | 關係品質 |
| `flow` | `in` 資源流向案主；`out` 案主付出；`both` 雙向；`none` 不畫箭頭 | 資源／能量流向 |
| `target` | 成員 id（預設案主）或 `"family"` | 連到特定成員，或連到整個家庭（從大圓邊緣出發） |
| `angle` | 度數，0＝右、90＝下、−90＝上（選填，通常不要填） | 強制指定方位；省略時程式會自動挑一個連線不壓到中央家系圖的方位 |
| `note` | 短文字（選填） | 線旁註記，例如「每月回診」 |

- **生態圖的連線不可以壓到中央的家系圖**（成員符號、標籤、婚姻線與親子線都不行）。程式會自動替每個外部系統挑方位來避開；若某位成員被家人包圍、找不到乾淨的路徑，該條線會改從大圓邊緣出發，並在線旁標出成員稱謂。所以 `angle` 通常不要填，只有使用者指定方位時才用。
- 敘述沒說重要性或遠近時用 `M`、`mid`，不要自行加碼；沒說流向時用 `none`。
- 圖例會自動產生在角落，不需另畫。

## 步驟 5：case.json 範例

```json
{
  "title": "王小姐家庭 家系圖",
  "client": "mom",
  "client_style": "fill",
  "household": ["mom", "d1", "s1"],
  "persons": [
    {"id": "mgf", "sex": "M", "deceased": true, "gen": 0, "x": 3},
    {"id": "mgm", "sex": "F", "age": 70, "label": "外婆", "gen": 0, "x": 4},
    {"id": "dad", "sex": "M", "age": 35, "deceased": true, "label": "父親", "note": "倉庫管理員", "gen": 1, "x": 1},
    {"id": "mom", "sex": "F", "age": 32, "label": "母親", "gen": 1, "x": 3},
    {"id": "bro", "sex": "M", "label": "舅舅", "note": "卡車司機", "gen": 1, "x": 4.5},
    {"id": "d1", "sex": "F", "age": 10, "label": "女兒", "gen": 2.2, "x": 1.5},
    {"id": "s1", "sex": "M", "age": 8, "label": "兒子", "marks": ["chronic"], "note": "氣喘", "gen": 2.2, "x": 2.5}
  ],
  "unions": [
    {"a": "mgf", "b": "mgm", "children": ["mom", "bro"]},
    {"a": "dad", "b": "mom", "type": "married", "note": "2013 結婚", "level": 1, "children": ["d1", "s1"]}
  ],
  "relations": [
    {"from": "mom", "to": "d1", "type": "close"},
    {"from": "mom", "to": "bro", "type": "conflict"}
  ],
  "eco": {
    "title": "王小姐家庭 生態圖",
    "center": ["mom", "d1", "s1"],
    "systems": [
      {"name": "天主教堂", "size": "M", "distance": "near", "line": "strong", "flow": "both"},
      {"name": "精神科醫院", "size": "M", "distance": "mid", "line": "normal", "flow": "in", "note": "每月回診"},
      {"name": "社福單位", "size": "L", "distance": "mid", "line": "normal", "flow": "in"},
      {"name": "國小", "size": "S", "distance": "near", "line": "weak", "target": "d1"},
      {"name": "男友", "size": "M", "distance": "far", "line": "stress"}
    ]
  }
}
```

## 步驟 6：產圖與檢查

1. 把本文件最後的程式**原樣**存成 `genoeco.py`（不要改寫或精簡，它已測試過）。
2. 執行 `python3 genoeco.py case.json out`。程式會印出 ⚠ 警告（成員重疊、id 打錯、子女沒排在父母下方、未知線型），有警告就先修 JSON。
3. 開啟 `out/genogram.png` 與 `out/ecomap.png` 親眼檢查：有沒有線穿過不相干的人、標籤互相蓋住、同住框圈錯人、案主標示正確，以及生態圖的連線有沒有壓到中央家系圖。有問題就調 `x`／`gen`／`level`／`angle` 重跑，直到乾淨為止。
4. 若環境沒有 playwright，程式只會輸出 SVG，直接交付 SVG 即可。
5. 若完全無法執行程式，依上面的符號規範手寫 SVG，兩張圖都要附圖例。

## 步驟 7：交付

依序提供：

1. **家系圖**（PNG，另附 SVG 供編輯）
2. **生態圖**（PNG，另附 SVG）
3. **評估觀察**（簡短，分三點）：可用資源／資源缺口／壓力與衝突來源。這些是依圖面整理的**初步假設**，供社工與督導或案主討論、擬定處遇計畫時參考，用「可能」「值得進一步了解」的語氣，不下診斷。
4. **待補資訊**：畫圖時發現缺少、建議回頭向個案確認的項目（例如祖父母輩存歿、某段關係的品質、某系統的資源流向）。

使用者之後補充或更正資訊時，改 `case.json` 重跑即可，兩張圖一起更新以保持一致。

## genoeco.py

```python
#!/usr/bin/env python3
"""genoeco.py — 家系圖（Genogram）＋生態圖（Ecomap）SVG 產生器
用法: python3 genoeco.py case.json 輸出資料夾
輸出: genogram.svg / ecomap.svg（若環境有 playwright 會另存 PNG）"""
import json, math, sys, os, html

FONT = "'Noto Sans CJK TC','Microsoft JhengHei','PingFang TC',sans-serif"
DX, DY, H = 92, 150, 22          # 欄距、列距、符號半徑
INK, GREY = "#222", "#bdbdbd"
REL = {"positive": "正向關係", "close": "關係親密", "fused": "過度親密", "distant": "關係疏離",
       "conflict": "衝突", "conflict_close": "衝突又親密", "cutoff": "溝通中斷／斷絕", "worsening": "關係惡化"}
UNI = {"married": "結婚", "cohabit": "同居", "separated": "分居", "divorced": "離婚"}
SEGS = []                        # 已畫出的線段，供生態圖避讓家系圖用
ECO = {"strong": "強烈正向連結", "normal": "正向／支持", "weak": "薄弱／疏離", "stress": "緊張／衝突／壓力"}


def esc(s): return html.escape(str(s))


def line(x1, y1, x2, y2, w=1.6, dash=None, color=INK):
    d = f' stroke-dasharray="{dash}"' if dash else ''
    SEGS.append((x1, y1, x2, y2))
    return (f'<line x1="{x1:.1f}" y1="{y1:.1f}" x2="{x2:.1f}" y2="{y2:.1f}" stroke="{color}" '
            f'stroke-width="{w}" stroke-linecap="round"{d}/>')


def poly(pts, w=1.6, dash=None, color=INK):
    d = f' stroke-dasharray="{dash}"' if dash else ''
    p = " ".join(f"{x:.1f},{y:.1f}" for x, y in pts)
    SEGS.extend(a + b for a, b in zip(pts, pts[1:]))
    return f'<polyline points="{p}" fill="none" stroke="{color}" stroke-width="{w}" stroke-linejoin="round"{d}/>'


def text(x, y, s, size=13, anchor="middle", weight="normal", color=INK, halo=False):
    h = ' stroke="#fff" stroke-width="4" paint-order="stroke"' if halo else ''
    return (f'<text x="{x:.1f}" y="{y:.1f}" font-size="{size}" text-anchor="{anchor}" '
            f'font-weight="{weight}" fill="{color}"{h}>{esc(s)}</text>')


def zig(x1, y1, x2, y2, amp=6, step=8, lead=0):
    L = math.hypot(x2 - x1, y2 - y1) or 1
    ux, uy = (x2 - x1) / L, (y2 - y1) / L
    n = max(2, int((L - 2 * lead) / step))
    pts = [(x1, y1)]
    for i in range(n + 1):
        t = lead + (L - 2 * lead) * i / n
        s = 0 if i in (0, n) else (amp if i % 2 else -amp)
        pts.append((x1 + ux * t - uy * s, y1 + uy * t + ux * s))
    return pts + [(x2, y2)]


def rel_line(x1, y1, x2, y2, t, lead=0):
    """關係線：家系圖成員互動、生態圖系統連結共用"""
    L = math.hypot(x2 - x1, y2 - y1) or 1
    ux, uy = (x2 - x1) / L, (y2 - y1) / L
    nx, ny = -uy, ux
    par = lambda d, w=1.6, dash=None: line(x1 + nx * d, y1 + ny * d, x2 + nx * d, y2 + ny * d, w, dash)
    mx, my = (x1 + x2) / 2, (y1 + y2) / 2
    if t == "close": return [par(-2.5), par(2.5)]
    if t == "fused": return [par(-4), par(0), par(4)]
    if t in ("distant", "weak"): return [par(0, 1.4, "3 6" if t == "distant" else "8 6")]
    if t in ("conflict", "stress"): return [poly(zig(x1, y1, x2, y2, lead=lead))]
    if t == "conflict_close": return [par(-5), par(5), poly(zig(x1, y1, x2, y2, amp=4))]
    if t == "strong": return [par(0, 5)]
    if t == "cutoff":
        o = []
        for s in (-1, 1):
            gx, gy = mx + s * ux * 6, my + s * uy * 6
            ex, ey = (x1, y1) if s < 0 else (x2, y2)
            o += [line(ex, ey, gx, gy), line(gx - nx * 8, gy - ny * 8, gx + nx * 8, gy + ny * 8)]
        return o
    if t == "worsening":
        o = [par(0)]
        k = -L * 0.2
        while k <= L * 0.2:
            cx, cy = mx + ux * k, my + uy * k
            o.append(line(cx - (ux + nx) * 5, cy - (uy + ny) * 5, cx + (ux + nx) * 5, cy + (uy + ny) * 5, 1.2))
            k += 8
        return o
    return [par(0)]


def person(p, x, y, client=False, cstyle="fill", h=H, mini=False):
    o, sex = [], p.get("sex", "U")
    fill = GREY if client and cstyle == "fill" else "#fff"
    sw = 2.6 if client else 1.6

    def shape(r, f, w):
        if sex == "M":
            return f'<rect x="{x-r:.1f}" y="{y-r:.1f}" width="{2*r}" height="{2*r}" fill="{f}" stroke="{INK}" stroke-width="{w}"/>'
        if sex == "F":
            return f'<circle cx="{x:.1f}" cy="{y:.1f}" r="{r}" fill="{f}" stroke="{INK}" stroke-width="{w}"/>'
        return (f'<polygon points="{x:.1f},{y-r:.1f} {x-r:.1f},{y+r:.1f} {x+r:.1f},{y+r:.1f}" fill="{f}" '
                f'stroke="{INK}" stroke-width="{w}"/>')        # 懷孕或性別不明
    o.append(shape(h, fill, sw))
    if client and cstyle == "double": o.append(shape(h - 5, "none", 1.4))
    marks = p.get("marks", [])
    if "disabled" in marks: o.append(line(x, y - h, x, y + h, 1.3))
    if "chronic" in marks: o += [line(x, y - h, x, y, 1.3), line(x, y, x + h, y, 1.3)]
    if p.get("deceased"):
        d = h if sex == "M" else h * 0.707
        o += [line(x - d, y - d, x + d, y + d, 1.4), line(x - d, y + d, x + d, y - d, 1.4)]
    if p.get("age") not in (None, "") and h >= 14:
        mk = "chronic" in marks or "disabled" in marks      # 有註記線時年齡移到左下角
        o.append(text(x - 10 if mk else x, y + 15 if mk else y + 5, p["age"], 10 if mk else 13, halo=bool(p.get("deceased"))))
    ty = y + h + 17
    if p.get("label"):
        o.append(text(x, ty, p["label"], 13, weight="bold", halo=True)); ty += 16
    if not mini:
        for n in str(p.get("note", "")).split("\n"):
            if n: o.append(text(x, ty, n, 11, color="#555", halo=True)); ty += 14
    return o


def geno(data, ids=None, mini=False):
    """回傳 (svg元素, 座標表, bbox)。ids 用於生態圖只畫核心家庭"""
    P = {p["id"]: p for p in data["persons"] if ids is None or p["id"] in ids}
    pos = {i: (p["x"] * DX, p["gen"] * DY) for i, p in P.items()}
    client, cstyle = data.get("client"), data.get("client_style", "fill")
    o = []
    hh = [i for i in data.get("household", []) if i in P]
    if hh and not mini:
        pts = sorted({(round(pos[i][0] + dx), round(pos[i][1] + dy)) for i in hh
                      for dx in (-H - 14, H + 14) for dy in (-H - 10, H + 46)})
        cr = lambda a, b, c: (b[0] - a[0]) * (c[1] - a[1]) - (b[1] - a[1]) * (c[0] - a[0])
        hull = []
        for seq in (pts, pts[::-1]):                      # 凸包：只圈同住成員
            part = []
            for q in seq:
                while len(part) >= 2 and cr(part[-2], part[-1], q) <= 0: part.pop()
                part.append(q)
            hull += part[:-1]
        o.append('<polygon points="%s" fill="none" stroke="#666" stroke-width="1.4" stroke-linejoin="round"/>' % " ".join(f"{x},{y}" for x, y in hull))
    for u in data.get("unions", []):
        a = u.get("a") if u.get("a") in P else None
        b = u.get("b") if u.get("b") in P else None
        kids = [c for c in u.get("children", []) if c in P]
        if not a and not b: continue
        if a and b:
            (xa, ya), (xb, yb) = pos[a], pos[b]
            by = max(ya, yb) + H + 44 + 16 * u.get("level", 0)
            t = u.get("type", "married")
            o.append(poly([(xa, ya + H), (xa, by), (xb, by), (xb, yb + H)], dash="6 5" if t == "cohabit" else None))
            mx = (xa + xb) / 2
            ox = u["drop_x"] * DX if "drop_x" in u else mx
            n = {"separated": 1, "divorced": 2}.get(t, 0)
            sx = mx - 22 if kids and abs(ox - mx) < 20 else mx
            for k in range(n):
                cx = sx + (k - (n - 1) / 2) * 9
                o.append(line(cx - 6, by + 9, cx + 6, by - 9))
            if u.get("note") and not mini:
                o.append(text(mx + 8 if kids else mx, by + 15, u["note"], 11, "start" if kids else "middle", color="#555", halo=True))
        else:
            ox, by = pos[a or b][0], pos[a or b][1] + H
        if kids:
            sy = min(pos[c][1] for c in kids) - H - 24
            ad, done, anchors = set(u.get("adopted", [])), set(), []
            for c in kids:
                if c in done: continue
                grp = [k for k in next((t for t in u.get("twins", []) if c in t), [c]) if k in P]
                done |= set(grp)
                anchors.append((sum(pos[k][0] for k in grp) / len(grp), grp))
            xs = [ax for ax, _ in anchors] + [ox]
            o.append(line(ox, by, ox, sy))
            if max(xs) - min(xs) > 1: o.append(line(min(xs), sy, max(xs), sy))
            for ax, grp in anchors:
                for k in grp:
                    o.append(line(ax, sy, pos[k][0], pos[k][1] - H, dash="5 4" if k in ad else None))
    if not mini:
        for r in data.get("relations", []):
            if r.get("from") in P and r.get("to") in P:
                (x1, y1), (x2, y2) = pos[r["from"]], pos[r["to"]]
                if abs(x2 - x1) < 8: x1, x2 = x1 + 12, x2 + 12      # 垂直對齊時右移，避免壓在親子線上
                L = math.hypot(x2 - x1, y2 - y1) or 1
                c = H + 5
                a1, b1 = x1 + (x2 - x1) * c / L, y1 + (y2 - y1) * c / L
                a2, b2 = x2 - (x2 - x1) * c / L, y2 - (y2 - y1) * c / L
                o += rel_line(a1, b1, a2, b2, r.get("type", "positive"))
                if r.get("note"): o.append(text((a1 + a2) / 2, (b1 + b2) / 2 - 10, r["note"], 11, color="#555", halo=True))
    for i, p in P.items():
        o += person(p, *pos[i], client=(i == client), cstyle=cstyle, mini=mini)
    xs, ys = [v[0] for v in pos.values()], [v[1] for v in pos.values()]
    m = 34 if mini else 70
    return o, pos, (min(xs) - m, min(ys) - m + 4, max(xs) + m, max(ys) + (46 if mini else 90))


def legend(x, y, items, cols, title="圖示說明", cw=178, rh=30):
    rows = math.ceil(len(items) / cols)
    o = [f'<rect x="{x}" y="{y}" width="{cols*cw+16}" height="{rows*rh+36}" rx="6" fill="#fff" stroke="#999"/>',
         text(x + 12, y + 20, title, 13, "start", "bold")]
    for i, (fn, label) in enumerate(items):
        ix, iy = x + 12 + (i % cols) * cw, y + 44 + (i // cols) * rh
        o += fn(ix, iy) + [text(ix + 58, iy + 5, label, 12, "start")]
    return o, cols * cw + 16, rows * rh + 36


def svg_doc(w, h, body):
    return (f'<svg xmlns="http://www.w3.org/2000/svg" width="{w:.0f}" height="{h:.0f}" viewBox="0 0 {w:.0f} {h:.0f}" '
            f'font-family="{FONT}"><rect width="100%" height="100%" fill="#fff"/>' + "\n".join(body) + '</svg>')


def genogram_svg(data):
    g, pos, bb = geno(data)
    cs = data.get("client_style", "fill")
    P = data["persons"]
    sm = lambda **kw: (lambda x, y: person(kw, x + 22, y, h=10))
    items = [(sm(sex="M"), "男性"), (sm(sex="F"), "女性"),
             (lambda x, y: person({"sex": "M"}, x + 12, y, True, cs, 10) + person({"sex": "F"}, x + 36, y, True, cs, 10),
              "案主（個案）")]
    if any(p.get("deceased") for p in P): items.append((sm(sex="M", deceased=True), "已故"))
    if any(p.get("age") not in (None, "") for p in P): items.append((lambda x, y: person({"sex": "F", "age": "32"}, x + 22, y, h=14), "符號內數字＝年齡"))
    if any("disabled" in p.get("marks", []) for p in P): items.append((sm(sex="M", marks=["disabled"]), "身心障礙"))
    if any("chronic" in p.get("marks", []) for p in P): items.append((sm(sex="M", marks=["chronic"]), "慢性病"))
    if any(p.get("sex", "U") not in "MF" for p in P): items.append((sm(sex="U"), "懷孕或性別不明"))

    def uni(t):
        def f(x, y):
            o = [poly([(x + 4, y - 8), (x + 4, y + 6), (x + 44, y + 6), (x + 44, y - 8)], dash="5 4" if t == "cohabit" else None)]
            n = {"separated": 1, "divorced": 2}.get(t, 0)
            return o + [line(x + 24 + (k - (n - 1) / 2) * 8 - 4, y + 12, x + 24 + (k - (n - 1) / 2) * 8 + 4, y, 1.4) for k in range(n)]
        return f
    us = data.get("unions", [])
    for t in UNI:
        if any(u.get("type", "married") == t and u.get("a") and u.get("b") for u in us): items.append((uni(t), UNI[t]))
    if any(u.get("adopted") for u in us): items.append((lambda x, y: [line(x + 24, y - 10, x + 24, y + 10, dash="5 4")], "領養"))
    if data.get("household"):
        items.append((lambda x, y: [f'<rect x="{x+2}" y="{y-10}" width="44" height="20" rx="6" fill="none" stroke="#666" stroke-width="1.4"/>'], "同住範圍"))
    for t in REL:
        if any(r.get("type", "positive") == t for r in data.get("relations", [])):
            items.append(((lambda t: lambda x, y: rel_line(x + 2, y, x + 48, y, t))(t), REL[t]))
    top = 70
    bw, bh = bb[2] - bb[0], bb[3] - bb[1]
    lg, lw, lh = legend(30, top + bh + 20, items, 4)
    W = max(bw, lw) + 60
    ox = (W - bw) / 2 - bb[0]
    body = [text(W / 2, 40, data.get("title", "家系圖"), 20, weight="bold"),
            f'<g transform="translate({ox:.1f},{top - bb[1]:.1f})">' + "\n".join(g) + '</g>'] + lg
    return svg_doc(W, top + bh + lh + 50, body)


def arrow(x, y, ux, uy, s=13):
    nx, ny = -uy, ux
    return (f'<polygon points="{x:.1f},{y:.1f} {x-ux*s+nx*s*0.42:.1f},{y-uy*s+ny*s*0.42:.1f} '
            f'{x-ux*s-nx*s*0.42:.1f},{y-uy*s-ny*s*0.42:.1f}" fill="{INK}"/>')


def wrap(s, n):
    out = []
    for part in str(s).split("\n"):
        out += [part[i:i + n] for i in range(0, len(part), n)] or [""]
    return out


def eco_link(x1, y1, x2, y2, t, flow):
    """(x1,y1)=案主/家庭端 (x2,y2)=外部系統端；flow: in=資源流向案主 out=案主付出 both none"""
    L = math.hypot(x2 - x1, y2 - y1) or 1
    ux, uy = (x2 - x1) / L, (y2 - y1) / L
    a_in, a_out = flow in ("in", "both"), flow in ("out", "both")
    s1 = (x1 + ux * 9, y1 + uy * 9) if a_in else (x1, y1)
    s2 = (x2 - ux * 9, y2 - uy * 9) if a_out else (x2, y2)
    o = rel_line(*s1, *s2, t, lead=10)
    if a_in: o.append(arrow(x1, y1, -ux, -uy))
    if a_out: o.append(arrow(x2, y2, ux, uy))
    return o


def eco_angles(S, geom, pos, segs, bb, k, client, P):
    """替未指定 angle 的外部系統挑方位：連線不可壓到中央家系圖的成員、標籤與結構線"""
    cx0, cy0 = (bb[0] + bb[2]) / 2, (bb[1] + bb[3]) / 2
    M = max(len(S), 12)
    slots = [math.radians(-90 + 360 * j / M) for j in range(M)]
    gap = lambda a, b: abs((a - b + math.pi) % (2 * math.pi) - math.pi)

    def cost(i, ang):
        tgt = S[i].get("target", client)
        if tgt not in pos: return 0
        (px, py), d = pos[tgt], geom[i][1]
        qx, qy = cx0 + d * math.cos(ang) / k, cy0 + d * math.sin(ang) / k
        L = math.hypot(qx - px, qy - py) or 1
        ux, uy = (qx - px) / L, (qy - py) / L
        c, t = 0, H + 8
        while t < L:
            x, y = px + ux * t, py + uy * t
            if not (bb[0] - 20 <= x <= bb[2] + 20 and bb[1] - 20 <= y <= bb[3] + 20): break
            hit = any(abs(x - ox) < H + 10 and -H - 10 < y - oy < H + 26 for o, (ox, oy) in pos.items() if o != tgt)
            hit = hit or (abs(x - px) < 7 * len(str(P.get(tgt, {}).get("label", ""))) + 3 and H + 3 < y - py < H + 22)
            for x1, y1, x2, y2 in ([] if hit else segs):
                vx, vy = x2 - x1, y2 - y1
                u = max(0, min(1, ((x - x1) * vx + (y - y1) * vy) / ((vx * vx + vy * vy) or 1)))
                if math.hypot(x - x1 - u * vx, y - y1 - u * vy) < 7: hit = True; break
            c += hit; t += 4
        return c
    ang = {i: math.radians(s["angle"]) for i, s in enumerate(S) if "angle" in s}
    free = [a for a in slots if all(gap(a, f) > math.pi / M for f in ang.values())]
    todo = [i for i in range(len(S)) if i not in ang]
    table = {i: {a: cost(i, a) for a in slots} for i in todo}
    todo.sort(key=lambda i: sum(1 for a in free if table[i][a] == 0))       # 可選方位最少的先挑
    for i in todo:
        cand = free or slots
        best = min(cand, key=lambda a: (table[i][a], -min([gap(a, u) for u in ang.values()] or [0])))
        ang[i] = best
        if best in free: free.remove(best)
    better = True
    while better:                                   # 局部調整：換到空位或兩兩互換，直到壓線數不再下降
        better = False
        for i in todo:
            for a in list(free):
                if table[i][a] < table[i][ang[i]]:
                    free.remove(a); free.append(ang[i]); ang[i] = a; better = True
            for j in todo:
                if table[i][ang[j]] + table[j][ang[i]] < table[i][ang[i]] + table[j][ang[j]]:
                    ang[i], ang[j] = ang[j], ang[i]; better = True
    return [(ang[i], cost(i, ang[i])) for i in range(len(S))]       # (方位, 仍壓到家系圖的取樣點數)


def ecomap_svg(data):
    e = data["eco"]
    R = 215
    client = data.get("client")
    ids = set(e.get("center") or data.get("household") or [client])
    del SEGS[:]
    g, pos, bb = geno(data, ids, mini=True)
    segs = list(SEGS)
    bw, bh = bb[2] - bb[0], bb[3] - bb[1]
    k = min(1.0, 2 * R * 0.74 / max(bw, bh))
    S = e.get("systems", [])
    geom = []
    for s in S:
        r = {"L": 66, "M": 52, "S": 40}.get(s.get("size", "M"), 52)
        geom.append((r, R + {"near": 55, "mid": 120, "far": 190}.get(s.get("distance", "mid"), 120) + r))
    P = {p["id"]: p for p in data["persons"]}
    placed = eco_angles(S, geom, pos, segs, bb, k, client, P)
    blocked = [c > 0 for _, c in placed]      # 找不到不壓圖的方位 → 改由大圓邊緣出發並註明成員
    geom = [(a, r, d) for (a, _), (r, d) in zip(placed, geom)]
    hx = max([R] + [abs(d * math.cos(a)) + r for a, r, d in geom]) + 24
    hy = max([R] + [abs(d * math.sin(a)) + r for a, r, d in geom]) + 24
    C, CY = hx, hy + 56                       # 依內容裁切畫布，圖例固定放右側
    tx, ty = C - k * (bb[0] + bw / 2), CY - k * (bb[1] + bh / 2)
    body = [text(C, 34, e.get("title", "生態圖"), 20, weight="bold"),
            f'<circle cx="{C:.1f}" cy="{CY:.1f}" r="{R}" fill="#fff" stroke="{INK}" stroke-width="2"/>']
    links, nodes = [], []
    for s, (ang, r, d), blk in zip(S, geom, blocked):
        sx, sy = C + d * math.cos(ang), CY + d * math.sin(ang)
        tgt = s.get("target", client)
        note = s.get("note", "")
        if blk:
            who = P.get(tgt, {}).get("label") or tgt
            note = f"{who}｜{note}" if note else str(who)
        if tgt in pos and not blk:
            px, py = pos[tgt][0] * k + tx, pos[tgt][1] * k + ty
            L = math.hypot(sx - px, sy - py)
            c = H * k * 1.42 + 4
            x1, y1 = px + (sx - px) * c / L, py + (sy - py) * c / L
        else:                                   # 連到整個家庭：從大圓邊緣出發
            px, py = C, CY
            x1, y1 = C + (R + 3) * math.cos(ang), CY + (R + 3) * math.sin(ang)
        L = math.hypot(sx - px, sy - py)
        x2, y2 = sx - (sx - px) * (r + 3) / L, sy - (sy - py) * (r + 3) / L
        links += eco_link(x1, y1, x2, y2, s.get("line", "normal"), s.get("flow", "none"))
        if note:
            links.append(text((x1 + x2) / 2 + (x2 - x1) * 0.18, (y1 + y2) / 2 + (y2 - y1) * 0.18 - 12, note, 11, color="#555", halo=True))
        nodes.append(f'<circle cx="{sx:.1f}" cy="{sy:.1f}" r="{r}" fill="#fff" stroke="{INK}" stroke-width="1.8"/>')
        ls = wrap(s["name"], {"L": 6, "M": 5, "S": 4}.get(s.get("size", "M"), 5))
        for j, t in enumerate(ls):
            nodes.append(text(sx, sy + 5 + (j - (len(ls) - 1) / 2) * 17, t, 14))
    body += links + [f'<g transform="translate({tx:.1f},{ty:.1f}) scale({k:.3f})">' + "\n".join(g) + '</g>'] + nodes
    used = [t for t in ECO if any(s.get("line", "normal") == t for s in S)]
    items = [((lambda t: lambda x, y: rel_line(x + 2, y, x + 48, y, t))(t), ECO[t]) for t in used]
    items += [(lambda x, y: [line(x + 2, y, x + 38, y), arrow(x + 48, y, 1, 0, 11)], "箭頭＝資源／能量流向"),
              (lambda x, y: [f'<circle cx="{x+14}" cy="{y}" r="11" fill="none" stroke="{INK}"/>',
                             f'<circle cx="{x+38}" cy="{y}" r="6" fill="none" stroke="{INK}"/>'], "圓越大＝影響力越大"),
              (lambda x, y: [text(x + 24, y + 5, "近 ↔ 遠", 11)], "距離＝親疏／便利性"),
              (lambda x, y: person({"sex": "M"}, x + 12, y, True, data.get("client_style", "fill"), 10)
               + person({"sex": "F"}, x + 36, y, True, data.get("client_style", "fill"), 10), "案主（個案）")]
    lg, lw, lh = legend(2 * hx + 8, 60, items, 1, cw=230)
    return svg_doc(2 * hx + lw + 24, max(2 * hy + 70, lh + 80), body + lg)


def check(data):
    w, ids = [], {p["id"] for p in data["persons"]}
    ps = data["persons"]
    for i, a in enumerate(ps):
        for b in ps[i + 1:]:
            if abs(a["gen"] - b["gen"]) < 0.3 and abs(a["x"] - b["x"]) < 0.95:
                w.append(f'重疊：{a["id"]} 與 {b["id"]} 位置太近')
    gen = {p["id"]: p["gen"] for p in ps}
    for u in data.get("unions", []):
        for c in [u.get("a"), u.get("b")] + u.get("children", []):
            if c and c not in ids: w.append(f'unions 參照到不存在的 id：{c}')
        par = [gen[x] for x in (u.get("a"), u.get("b")) if x in gen]
        for c in u.get("children", []):
            if c in gen and par and gen[c] <= max(par): w.append(f'子女 {c} 沒有排在父母的下一列')
    for r in data.get("relations", []):
        for c in (r.get("from"), r.get("to")):
            if c not in ids: w.append(f'relations 參照到不存在的 id：{c}')
        if r.get("type", "positive") not in REL: w.append(f'未知關係類型：{r.get("type")}')
    if data.get("client") not in ids: w.append("未指定案主（client）或 id 不存在")
    for s in data.get("eco", {}).get("systems", []):
        if s.get("line", "normal") not in ECO: w.append(f'未知生態圖線型：{s.get("line")}')
    if data.get("eco"):                     # 生態圖中央的成員必須由親屬線連成一張圖
        cen = set(data["eco"].get("center") or data.get("household") or [data.get("client")]) & ids
        adj = {i: set() for i in cen}
        for u in data.get("unions", []):
            grp = [x for x in [u.get("a"), u.get("b")] + u.get("children", []) if x in cen]
            par = [x for x in (u.get("a"), u.get("b")) if x in cen]
            for x in grp:
                for y in grp:
                    if x != y and (x in par or y in par): adj[x].add(y)
        seen, comps = set(), []
        for i in cen:
            if i in seen: continue
            st, comp = [i], set()
            while st:
                x = st.pop()
                if x in comp: continue
                comp.add(x); st += list(adj[x])
            seen |= comp; comps.append(sorted(comp))
        if len(comps) > 1:
            w.append("生態圖中央成員沒有親屬線相連：" + "／".join("、".join(c) for c in comps)
                     + "。請把連接他們的人（例如不同住的父母、已故的長輩）也放進 eco.center")
    return w


def to_png(svg_path):
    try:
        from playwright.sync_api import sync_playwright
        with sync_playwright() as pw:
            b = pw.chromium.launch()
            pg = b.new_page(device_scale_factor=2)
            pg.goto("file://" + os.path.abspath(svg_path))
            pg.locator("svg").screenshot(path=svg_path[:-4] + ".png")
            b.close()
        return True
    except Exception as ex:
        print("（略過 PNG 轉檔：%s）" % str(ex).splitlines()[0])
        return False


if __name__ == "__main__":
    data = json.load(open(sys.argv[1], encoding="utf-8"))
    out = sys.argv[2] if len(sys.argv) > 2 else "."
    os.makedirs(out, exist_ok=True)
    for m in check(data): print("⚠", m)
    jobs = [("genogram.svg", genogram_svg)] + ([("ecomap.svg", ecomap_svg)] if data.get("eco") else [])
    for name, fn in jobs:
        path = os.path.join(out, name)
        open(path, "w", encoding="utf-8").write(fn(data))
        print("已輸出", path, "+ PNG" if to_png(path) else "")
```