---
layout: post
title: "幫老師抓 FAJ 的引用資料，最後做成任何期刊都能用的小工具"
date: 2026-09-30
tags: [Python, OpenAlex, 自動化]
excerpt_separator: <!--more-->
---

我之前擔任老師的研究助理，老師在研究《Financial Analysts Journal》（FAJ），他想知道引用 FAJ 的文章都是甚麼樣的背景。所以老師請我整理出表格，展示 FAJ 歷年來的每一篇文章，分別被哪些文章引用過，那些引用的文章都來自甚麼期刊。

這種資料一篇一篇手動查根本不可能，所以我決定寫程式來抓。FAJ 從 1946 年到 2024 年，我最後抓到大約 20 萬筆引用關係，存成 79 個 CSV，合計大約 39 MB。

<!--more-->

## 我怎麼做

資料來源我用 OpenAlex，主要是因為它資料免費開放，API 只要申請一個免費的 key 就能用，門檻很低。它可以直接用期刊的 ISSN 找出文章，再查每一篇文章被哪些作品引用，剛好符合老師的需要。

因為每一篇文章都要各別去查引用者，總量很大，耗時也長，所以我讓每個年份各自存成一個檔案。這樣某一年出問題，只要重跑那一年就好，不用整份重來。之後 FAJ 出了新的年份，也只要補抓新的。

## 為什麼做成 GUI

既然以後一定會補新年份，我就不想每次都打開程式碼改參數。所以我用 Python 寫了一個簡單的 GUI，輸入期刊的 ISSN 和年份，按開始就會自動抓，且不會重複寫入已經抓到過的資料。

寫完之後我發現，這個工具從頭到尾都沒有綁定 FAJ。只要期刊在 OpenAlex 有資料，輸入 ISSN 就能用。

## 它現在能做什麼

你輸入期刊的 ISSN 和起訖年份，它會先找出這些年份的文章，再找出引用每一篇文章的作品，最後輸出成 CSV，一個年份一個檔案：

```
output/
└── 0022-1082/
    └── citations_2025.csv
```

表格中的每一列是一組引用關係，被引用的是哪一篇文章，引用它的文章標題、期刊、作者、發表年份是什麼。

## 從自己用到給別人用

這套工具起初只是我自己用的，功能很陽春。後來準備寄資料給老師時，想到如果能把工具一併附上，老師之後要更新資料也會方便許多。不過既然要交到別人手上，就不能再用能跑就行的標準，於是我花了點時間把細節補齊，陸續加了幾項更穩固的機制：

- 重跑同一個期刊和年份時，只會補上新的引用，不會重複
- 已經存在的檔案如果壞掉或格式不對，會先擋下來，不會直接寫進去
- 跑完會告訴你是完整完成、部分失敗還是失敗
- 檔案用 UTF-8 with BOM 儲存，用 Excel 開不會亂碼

## 怎麼使用

需要 Python 3.10 以上。

```bash
git clone https://github.com/vinacherryhsui/Journal-Citation-Crawler.git
cd Journal-Citation-Crawler
python -m pip install -r requirements.txt
python -m citation_crawler
```

打開之後填這幾個欄位：

- 期刊的 ISSN（例如 Journal of Finance 是 `0022-1082`）
- 起始年份和結束年份
- 輸出資料夾
- OpenAlex API key（選填，但建議申請一個，免費，到 [openalex.org/settings/api](https://openalex.org/settings/api) 取得；沒有 key 的話每天額度很少，只夠測試）

按下 Start，進度會顯示在下面的視窗。

## 使用上要注意的地方

- 結果取決於 OpenAlex 收錄了什麼，和 Web of Science、Scopus 的資料不完全一樣
- 有些欄位（DOI、標題、期刊、作者）可能是空的
- OpenAlex 一直在更新，不同時間跑，同一篇文章的引用數可能不一樣
- 免費的 API key 每天有固定的使用額度，抓很大的期刊時，可能要分幾天、分年份區間來跑
- 這個工具只負責把引用資料抓下來，不包含額外分析

## 最後

如果你也需要某一本期刊的被引用資料，歡迎試試看，有問題或想法可以直接在 GitHub 留言：
[Journal-Citation-Crawler](https://github.com/vinacherryhsui/Journal-Citation-Crawler)
