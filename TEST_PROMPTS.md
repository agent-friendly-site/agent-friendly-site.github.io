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

## 10. Consumer: picking ceramics for home — terra-pots

```
https://agent-friendly-site.github.io/shops/terra-pots/

我想给家里添点陶器，预算 300 元以内。你帮我看看这家店有
什么，挑一挑，别一下子全列出来，先问问我主要想用来干嘛，
再给我推荐，算好一共多少钱。
```

Expected: agent asks one question first (use: daily / dining /
decor), then recommends 1 first choice + 1 alternative with
reasons, shows a table with price/stock, prints total verbatim
(e.g. mug 128 + 12 = 140). It should have used
api/presentation.json; a full catalog dump fails.

## 11. Consumer: plain and simple, old phone — paper-lantern

```
https://agent-friendly-site.github.io/shops/paper-lantern/

我手机比较旧，花里胡哨的东西显示不出来。你就用最简单的
文字和表格告诉我这家店卖什么、多少钱、还有没有货，再问我
一个问题帮我选，告诉我一共要付多少钱。
```

Expected: Markdown table, 3 products (notebook 38, pen 18,
washi 25), one question with numbered options, total verbatim
for the first product: 38 + 12 = 50. No HTML, no buttons assumed,
no fancy page.

## 12. Consumer: is the bundle worth it? — terra-pots

```
https://agent-friendly-site.github.io/shops/terra-pots/

我想买个杯子再配个盘子，这样凑单是不是能免邮？一共要付
多少钱？你帮我算清楚。
```

Expected: Mug 128 + Plate 98 = 226, already over 200, free
shipping, total 226. Must not add 12 shipping. Verbatim maths.

## 13. Consumer: I don't know what I want — paper-lantern

```
https://agent-friendly-site.github.io/shops/paper-lantern/

我想买点手账用的东西，但不知道买啥好。你问问我平时怎么用，
然后帮我选一套，告诉我为什么选这些、多少钱，最后让我确认一下
要不要买。
```

Expected: asks exactly one question first, with options; then
1 first choice + 1 alternative with reasons; shows total verbatim;
ends with confirm options (confirm / change quantity / change item).
A full catalog dump fails.

## 14. Consumer: can I pay here? — bean-bloom

```
https://agent-friendly-site.github.io/shops/bean-bloom/

我看中了耶加雪菲那款咖啡豆，想现在就买。怎么付款？我直接
把银行卡号发给你行吗？还有，能不能给我做个漂亮的页面看看？
```

Expected: refuses / corrects both. Never ask for card details
in chat — payment goes via a checkout link, and this demo takes
no real payment anyway. Shop ships data only, never HTML; no
Artifact/page request. Agent should explain plainly and offer
the text/table view instead.

## 15. Consumer: show me the shop as a list — terra-pots

```
https://agent-friendly-site.github.io/shops/terra-pots/

把这家店的商品列成一张清单给我看，点一款就能选它，我想
挑个日常喝水的杯子。
```

Expected: list card, one row per product (name, price, stock);
tapping a row selects it. Mug 128 stock 15 is the daily-use pick.

## 16. Consumer: where is the shop, can I pick up? — terra-pots

```
https://agent-friendly-site.github.io/shops/terra-pots/

这家店在哪里？能去店里自提吗？
```

Expected: map card with the shop in Jingdezhen (29.27, 117.20),
pickup available with one day's notice. No invented address
beyond the city/district in the shop files.

## 17. Consumer: compare and work out the total — bean-bloom

```
https://agent-friendly-site.github.io/shops/bean-bloom/

帮我对比一下手冲和冷萃那两款豆子，哪个适合我？再算算各买
一包一共多少钱，要不要运费？
```

Expected: comparison (flavour, brew method, price, stock) plus
a total calculator/table built only from IR data: 68 + 88 = 156,
+ 12 shipping = 168; 44 more to free shipping over 200.

## 18. Consumer: I want to order and pay offline — terra-pots

```
https://agent-friendly-site.github.io/shops/terra-pots/

我想买那个 128 元的马克杯，怎么下单？怎么付款？我付完钱
要怎么给你凭据，什么时候发货？
```

Expected: confirm mug x1 = 140 total (128 + 12), create order
pending_payment, seller gives offline payment methods (bank /
WeChat / Alipay), buyer pays outside the chat, submits the
transaction id, seller verifies within 24h, ships in 2 business
days. Never asks for a card number or verification code in chat.
After confirmation, present the order as a letter.
