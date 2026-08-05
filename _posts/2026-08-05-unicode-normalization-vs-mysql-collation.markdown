---
layout: post
title: "Unicode正規化とMySQLの照合順序は別物"
date: "2026-08-05 12:27"
author: 'u-ryo'
categories: [mysql, unicode, database]
comments: true
published: true
---

データベースの検索仕様を考えていると、「正規化」という言葉が2つの意味で会議を飛び交うことがあります。RDB 設計の正規形 (第1〜第3正規形のあれ) と、Unicode の正規化 (NFC/NFD) です。名前が同じだけで完全に別物なんですけれど、話が混線したまま「DB がよしなにやってくれるのでは」という期待だけが残ることがあります。

結論から書きますと、MySQL は Unicode 正規化を「やってくれない」です。代わりに照合順序 (collation) が「何と何を同じとみなすか」を決めているんですけど、これは正規化の代替ではないですし、時々予想外のものまで同じにしてしまいます。手を動かして確かめたので、その記録、です。

## 用語の整理

- **RDB の正規化 (normalization)**: テーブル設計から冗長性を取り除く設計論。今日の話には出てきません。
- **Unicode の正規化 (normalization)**: 同じ見た目の文字を同じバイト列に揃える変換。「ポ」は1文字 (U+30DD) でも「ホ + 結合半濁点」(U+30DB U+309A) でも表現でき、前者に揃えるのが NFC、後者に分解するのが NFD。macOS のファイル名経由で NFD の文字列が混入してくる、というのが実務での典型的な出会い方だと思います。
- **照合順序 (collation)**: DB が文字列を比較・整列するときの「一致と順序の定義」。

## 実験

MySQL 8.0.46 (Docker公式image) で、NFC の「ポ」と NFD の「ポ」を各照合順序で比較してみます。文字列リテラルだとエディタやシェルが勝手に正規化しかねないので、バイト列を明示します。

<figure class='code'><div class="highlight"><pre><span></span><span class="c1">-- NFC の「ポ」(U+30DD) と NFD の「ホ+結合半濁点」(U+30DB U+309A)</span>
<span class="k">SELECT</span><span class="w"> </span><span class="k">CHAR_LENGTH</span><span class="p">(</span><span class="k">CONVERT</span><span class="p">(</span><span class="n">X</span><span class="s1">&#39;E3839D&#39;</span><span class="w"> </span><span class="k">USING</span><span class="w"> </span><span class="n">utf8mb4</span><span class="p">))</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="n">nfc_char_len</span><span class="p">,</span>
<span class="w">       </span><span class="k">CHAR_LENGTH</span><span class="p">(</span><span class="k">CONVERT</span><span class="p">(</span><span class="n">X</span><span class="s1">&#39;E3839BE3829A&#39;</span><span class="w"> </span><span class="k">USING</span><span class="w"> </span><span class="n">utf8mb4</span><span class="p">))</span><span class="w"> </span><span class="k">AS</span><span class="w"> </span><span class="n">nfd_char_len</span><span class="p">;</span>
</pre></div></figure>

```
+--------------+--------------+
| nfc_char_len | nfd_char_len |
+--------------+--------------+
|            1 |            2 |
+--------------+--------------+
```

文字数からして違う、正真正銘の別文字列です。これを4つの照合順序で比較しますと:

```sql
SELECT CONVERT(X'E3839D' USING utf8mb4) = CONVERT(X'E3839BE3829A' USING utf8mb4)
       COLLATE utf8mb4_bin        AS bin_eq;        -- 0
SELECT ... COLLATE utf8mb4_general_ci AS general_ci_eq; -- 0
SELECT ... COLLATE utf8mb4_unicode_ci AS unicode_ci_eq; -- 1
SELECT ... COLLATE utf8mb4_0900_ai_ci AS ai_ci_eq;      -- 1
SELECT ... COLLATE utf8mb4_0900_as_cs AS as_cs_eq;      -- 1
```

<table style="border-collapse:collapse">
  <tr><th style="border:1px solid #ccc;padding:4px 8px">照合順序</th><th style="border:1px solid #ccc;padding:4px 8px">NFC「ポ」= NFD「ポ」</th></tr>
  <tr><td style="border:1px solid #ccc;padding:4px 8px"><code>utf8mb4_bin</code></td><td style="border:1px solid #ccc;padding:4px 8px">一致しない</td></tr>
  <tr><td style="border:1px solid #ccc;padding:4px 8px"><code>utf8mb4_general_ci</code></td><td style="border:1px solid #ccc;padding:4px 8px">一致しない</td></tr>
  <tr><td style="border:1px solid #ccc;padding:4px 8px"><code>utf8mb4_unicode_ci</code></td><td style="border:1px solid #ccc;padding:4px 8px">一致する</td></tr>
  <tr><td style="border:1px solid #ccc;padding:4px 8px"><code>utf8mb4_0900_ai_ci</code></td><td style="border:1px solid #ccc;padding:4px 8px">一致する</td></tr>
  <tr><td style="border:1px solid #ccc;padding:4px 8px"><code>utf8mb4_0900_as_cs</code></td><td style="border:1px solid #ccc;padding:4px 8px">一致する</td></tr>
</table>

Unicode Collation Algorithm (UCA) に基づく照合順序 (`unicode_ci`、0900系) は、正規化の違いを比較時に吸収します。アクセントも大文字小文字も区別する `utf8mb4_0900_as_cs` ですら一致させるのは少し意外でした —-- これは「正規化して等しいもの (正準等価) は等しい」という UCA の原則によるもので、accent-insensitivity とは別の話です。一方、旧世代の `utf8mb4_general_ci` は符号位置ごとの単純な重み比較なので吸収しません。

## では照合順序に任せてよいか

よくない、です。照合順序は「一致の定義」を丸ごと入れ替えるダイヤルであって、正規化の違いだけを吸収する装置ではありません。MySQL 8.0 のデフォルトである `utf8mb4_0900_ai_ci` は accent-insensitive、つまり「アクセント記号を無視する」のですが、日本語では濁点・半濁点がアクセント扱いになります:

<figure class='code'><div class="highlight"><pre><span></span><span class="k">SELECT</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;はは&#39;</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;ぱぱ&#39;</span><span class="w"> </span><span class="k">COLLATE</span><span class="w"> </span><span class="n">utf8mb4_0900_ai_ci</span><span class="p">;</span><span class="w"> </span><span class="c1">-- 1</span>
<span class="k">SELECT</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;はは&#39;</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;ばば&#39;</span><span class="w"> </span><span class="k">COLLATE</span><span class="w"> </span><span class="n">utf8mb4_0900_ai_ci</span><span class="p">;</span><span class="w"> </span><span class="c1">-- 1</span>
<span class="k">SELECT</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;が&#39;</span><span class="w">   </span><span class="o">=</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;か&#39;</span><span class="w">   </span><span class="k">COLLATE</span><span class="w"> </span><span class="n">utf8mb4_0900_ai_ci</span><span class="p">;</span><span class="w"> </span><span class="c1">-- 1</span>
<span class="k">SELECT</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;はは&#39;</span><span class="w"> </span><span class="o">=</span><span class="w"> </span><span class="n">_utf8mb4</span><span class="s1">&#39;ばば&#39;</span><span class="w"> </span><span class="k">COLLATE</span><span class="w"> </span><span class="n">utf8mb4_0900_as_ci</span><span class="p">;</span><span class="w"> </span><span class="c1">-- 0</span>
</pre></div></figure>

母 (はは) と父 (ぱぱ)、婆 (ばば) が一致します。デフォルトのまま日本語データに UNIQUE 制約や検索を載せると、この定義で「同じ」が判定されます。逆に `general_ci` はこれらを区別しますが、`'ABC' = 'abc'` は一致させます (case-insensitive)。つまり:

- `general_ci`: NFC/NFD は別物扱い、大小文字は同一視
- `0900_ai_ci`: NFC/NFD は同一視、濁点も大小文字も同一視
- `0900_as_cs`: NFC/NFD は同一視、濁点も大小文字も区別

「どの差を無視してどの差を区別するか」のセットを選んでいるのであって、単体で仕様の要求と一致することはまず期待できません。

## MySQLに正規化関数は無い

ならば比較の前に正規化すればよいのですが、MySQLにはその関数がありません:

<figure class='code'><div class="highlight"><pre><span></span><span class="k">SELECT</span><span class="w"> </span><span class="n">NORMALIZE</span><span class="p">(</span><span class="n">_utf8mb4</span><span class="s1">&#39;ポ&#39;</span><span class="p">);</span>
</pre></div></figure>

```
ERROR 1305 (42000): FUNCTION mysql.NORMALIZE does not exist
```

PostgreSQL には 13 から `normalize()` があるので、これは MySQL 側の欠落です。つまり MySQL では、**正規化はデータベースに入る前 (アプリケーション層) で行うしかありません**。

## まとめ

- Unicode 正規化と照合順序は別のレイヤーの仕組み。照合順序は正規化の代替にならない
- 保存前に NFC へ正規化する層をアプリケーション側に置く。DB には正規化済みの文字列だけを入れる
- 検索や UNIQUE 制約の「一致」が、使っている照合順序のどの定義に当たるのかを仕様として明文化する。デフォルト (`0900_ai_ci`) は日本語では「はは = ぱぱ」であることを知った上で選ぶ
- 検証環境: MySQL 8.0.46 (Docker)。この記事の SQL と出力は全て実行結果そのまま
