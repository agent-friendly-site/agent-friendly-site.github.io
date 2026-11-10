# Blind-test prompts — copy one into a fresh session

Rules for a fair test: open a new session, paste exactly one prompt,
do not paste any file list or prior knowledge. The agent may only use
what it can fetch live from the shop URL. Grade it against the
"Expected" line under each prompt.

## 1. Basic discovery — terra-pots

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/terra-pots/

这家店是什么店？卖什么商品？每款的价格和库存是多少？
```

Expected: Terra Pots Ceramics / 陶土手作, Jingdezhen stoneware.
Mug 300ml 128 CNY stock 15; Plate 20cm 98 CNY stock 22;
Bud Vase 158 CNY stock 8.

## 2. Buying decision — bean-bloom

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/bean-bloom/

我想买咖啡豆，预算 100 元以内。推荐一款有货的，告诉我价格、
库存、运费是多少，满多少免邮，总共要付多少钱？
```

Expected: Yirgacheffe 68 CNY stock 42, or Cold Brew 88 CNY stock 27.
Shipping flat 12 CNY, free over 200 CNY. Total is price + 12 CNY
if under 200 CNY. Do not recommend the decaf — it is out of stock.

## 3. Out-of-stock trap — bean-bloom

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/bean-bloom/

我想买低因咖啡豆，能买吗？如果不能，原因是什么？不要猜，
没有就说没有。
```

Expected: No. Decaf Medium Roast 250g, 72 CNY, stock 0.
The agent must not invent stock or an alternative decaf product.

## 4. Cross-shop comparison — no mixing shops

```
只给你下面两个网址，不要用你之前知道的任何信息，只根据这两个
网址在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/paper-lantern/
https://agent-friendly-site.github.io/shops/terra-pots/

这两家店各卖什么？有没有重叠的商品？各举一款商品，写出价格
和库存，并说明你是从哪家店的哪个文件里查到的。
```

Expected: Paper Lantern = stationery (notebook 38 CNY, pen 18 CNY,
washi 25 CNY). Terra Pots = ceramics (mug 128 CNY, plate 98 CNY,
vase 158 CNY). No overlap. The agent must not mix the catalogs.

## 5. Policy — terra-pots

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/terra-pots/

如果我买一个陶土马克杯，运费多少？满多少免邮？如果不满意，
几天内能退，退货运费谁付？
```

Expected: Flat 12 CNY, free over 200 CNY, 7-day returns,
unused condition, buyer pays return shipping.
A single mug (128 CNY) does not reach free shipping.

## 6. How to interact via MCP — terra-pots

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/terra-pots/

如果我想用 MCP 和这家店交互，端点是什么？用什么协议？
有哪些工具？在 GitHub Pages 上能不能直接调用这个 MCP？
为什么？
```

Expected: POST https://agent-friendly-site.github.io/shops/terra-pots/mcp,
Streamable HTTP, protocol 2025-06-18. Tools: list_products,
get_product, create_order, get_order. No — not live on GitHub
Pages, Pages is static-only; the endpoint is documented in
mcp.json only. create_order is a mock and needs idempotency_key.

## 7. Non-existent product trap — terra-pots

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/terra-pots/

这家店有咖啡豆卖吗？如果有，是哪一款、多少钱？如果没有，
就直接说没有，不要编造。
```

Expected: No. Terra Pots sells ceramics only. The agent must
not invent a coffee product, and must not answer from the
bean-bloom shop.

## 8. Guided shopping — bean-bloom (tests the guide layer)

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/bean-bloom/

我想买咖啡豆，但不知道选哪款。你先问我 1-2 个问题了解我的
需求（比如怎么冲、喜欢什么风味、预算），再推荐一款首选和
一款备选，讲清楚理由，算好总价（含运费），最后问我是否确认。
不要一上来就把所有商品都列出来。
```

Expected: the agent asks about brew method / taste / budget
first, then recommends 1 first choice + 1 alternative with
reasons, shows the maths, and asks for confirmation. It must
have read guide.md / recommendations.json (or llms.txt's
shopping-guide section) to do this — a plain catalog dump
fails this prompt.

## 9. Rich-format compatibility — bean-bloom (tests the presentation IR)

```
只给你这一个网址，不要用你之前知道的任何信息，只根据这个网址
在线上能查到的内容回答，并列出你实际打开过的链接：

https://agent-friendly-site.github.io/shops/bean-bloom/

请用 Markdown 表格列出这家店的商品（商品 | 价格 | 库存），
一次只问一个问题，用编号选项让我选（比如 1. 手冲 2. 冷萃
3. 意式），再算一次总价并原样写出来。不要发 HTML，不要
要求我打开 Artifact 或画布，也不要让我在聊天里输入支付信息。
```

Expected: the agent reads api/presentation.json, renders a
Markdown table with at most 5 products, asks one question with
numbered options, and prints the total verbatim (e.g. 68 + 12 = 80).
Any HTML, Artifact request, or payment-data request fails.
