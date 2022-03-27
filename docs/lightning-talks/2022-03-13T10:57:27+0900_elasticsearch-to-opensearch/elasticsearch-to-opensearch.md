---
theme : "night"
transition : "convex"
highlightTheme : "monokai"
logoImg : "https://www.ca-adv.co.jp/common/images/share/logo.svg"
slideNumber : true
title: "Elasticsearch から Opensearch への爆発アップデート"
---

<link rel="stylesheet" href="css/custom.css" id="theme">

## 爆発アップデート💥の話 <br><br> <small>Elasticsearch v1.3 💩</small><br> ↓ <br><small>Opensearch v1.1 👍</small>

---

## LT の内容
- Elasticsearch / Opensearch とは?
- なんでアップデートが必要だったのか？
- アップデートする上で苦労したこと
- まとめ

---

<!-- .slide: data-background="imgs/elastic-search-background.png" -->

<div class="black-glass-effect">

## Elasticsearch とは


Elasticsearchは、分散型RESTful検索/分析エンジン
- ___超高速検索___
  - CX+ 1億件以上のデータの中から、対象のデータを即座に検索する
  - RDBMSよりも高性能な全文検索機能
- ___パワフルな分析___
  - kibana などのGUIツールでデータをビジュアライズできる

</div>



--

<!-- .slide: data-background="imgs/opensearch.png" -->


<div class="black-glass-effect">

## Opensearch とは

Elasticsearch の OSS バージョン
- 基本的な機能は Elasticsearch と同じ
- Opensearch は Apache ライセンス バージョン 2.0 で公開されている


</div>

---




## なんでアップデートが必要だったの？

--

## 障害発生

Elasticsearch サーバーへの負荷が高まったことで、レスポンスが遅くなり、<br>

CX+にアクセスしにくくなる障害が発生しました 😢

--

#### 原因 1: <br>Elasticsearch サーバーがリクエストを処理しきれず、レスポンスできなくなった

<hr>

- 特定の検索条件でリクエストが来ると、レスポンスまでに時間がかかってしまう
- その間にも様々なリクエストが送られ、Elasticsearch サーバーに対する負荷が雪だるま式に増大していった

--

#### 原因 2: <br>Elasticsearch のサーバーがスケールできる作りになっていなかった

<hr>

- 基本的に EC2 インスタンス 1個で稼働で、負荷分散など出来ていなかった
- AutoScaling を使って、EC2 インスタンスを複数稼働させることを以前試みたが、当時の Elasticsearch の仕様的に複数のインスタンスが同一ネットワークに存在すると、おかしな挙動をしていた


--

## 負荷が高まったときに、スケールさせたい・・・

--

<div class="black-glass-effect">

<!-- .slide: data-background="imgs/rocket.jpg" -->

## 3/9 <br> CX+ で使っていた Elasticsearch サーバーを Opensearch に爆発アップデートさせました！

</div>

--

#### ✅ マネージドなので、スケールについて心配しなくても良くなった
#### ✅ Opesearch のパッチ適用なども自動的にされるようなのでセキュリティ的にも安心
#### ✅ 今後のシステム運用が楽になるはず🙏

---

## アップデートする上で苦労したこと

--

### Elasticsearch 難しい😢

--

#### 概念・仕組みの理解が難しい

- document, shard, analyzer など初めての概念・仕組みがいろいろありました
- index は RDBMS のインデックスとは似て非なるもの → 混乱した 🤯

--

#### API の使い方が難しい 

```bash
curl -XPOST -H "content-type: application/json" \
http://localhost:9200/employees/_search -d '
{
    "query": {
        "multi_match" : {
            "query" : "heursitic reserch",
            "fields": ["phrase","position"],
            "fuzziness": 2
        }
    },
    "size": 10
}'
```
- 〇〇クエリという機能が多い
  - `Bool Query`,
  - `Multi-Match Query`, 
  - `Match Phrase Query`, etc...

--

### 環境構築 難しい😢

--

#### 検証環境、本番環境を作るためには、AWS とかネットワークに関する知識が必要

- EC2 インスタンスにパブリックIP を設定し忘れて、docker のダウンロードが出来ない
  - パブリックなサブネットにある EC2 インスタンス自体にパブリックIP(Elastic IP)が割り当てられていないと、インターネットと通信が出来ないので、yum install が出来なかった

More ...

--

- サブネットのルートテーブルのレコードを間違えて、全く通信出来ない
  - igw にルーティングするレコードで、
    - 0.0.0.0/0 とすべきところを
    - 0.0.0.0/32 にしていた


- EC2、ALB(NLB)、Opensearch それぞれにセキュリティグループの設定が必要

--

### 現行サーバーから Opensearch へのデータ移行がうまくいかない 😢

--

#### データを移行する方法

- スナップショットによるデータ移行
- Reindex API によるデータ移行

--

### スナップショットによるデータ移行

1. Elasticsearch からスナップショットを生成
2. S3 バケットにスナップショットを保存
3. S3 バケットにあるスナップショットを Opensearch でリストア

<br>
<br>

#### → 現行の Elasticsearch サーバーが古すぎて、スナップショットの生成が出来ない 😢 {.fragment} 

--

#### Reindex API によるデータ移行

Elasticsearch のデータを直接 Opensearch にコピーする方法

<br><br>

#### → Elaseticsearch v1.3 が古すぎて、データの互換性が無かった{.fragment}
データの構造や型が、v1.3 と Opensearch では異なっているため Reindex API を使うとエラーが起きて全然データのコピーが出来ない  😢{.fragment}

--

じゃあどうしたか

### elasticdump というツールを使いました 🎉{.fragment}

前述の方法では全く出来なかったデータのコピーが、elastidump を使うことでうまくいきました{.fragment}

データ移行後、本番の CX+ と Opensearch を接続しても、基本的に問題無し<br><small>軽微なバグは見つかったので、目下対応中です</small>{.fragment}


---

## まとめ

- Opensearch にアップデートしたことで、CX+ の安定性が向上します(するはずです)
- マネージドサービスといえど、インフラやそのサービスに対する一定の知識がないと、なかなか難しい
- 既存のツールで一発で問題解決できることがある

---

# E O L