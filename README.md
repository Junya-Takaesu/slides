# スライド powerd by reveal.js

## 使い方など
- 画像のURLを指定するときは、このリポジトリのルートからの相対パスを指定する方法
  - 例: スライドに背景画像を設定する
    ```html
    <!-- .slide: data-background=lightning-talks/2022-03-13T10:57:27+0900_elasticsearch-to-opensearch/imgs/elastic-search-background.png -->
    ```
    - リポジトリに 以下のディレクトリがあり、そのディレクトリにある画像を参照している
      - ディレクトリ: 
        - `lightning-talks/2022-03-13T10:57:27+0900_elasticsearch-to-opensearch/imgs/`
      - 画像:
        - `elastic-search-background.png`
    - ↑のように指定すると、拡張の `tokiedokie.reveal-markdown` の機能でブラウザでスライドを開くと、以下のURLに変換されてブラウザにロードされるようになる
      ```
      http://localhost:62939/lightning-talks/2022-03-13T10:57:27+0900_elasticsearch-to-opensearch/imgs/elastic-search-background.png
      ```
      - 🔥 スライドの md ファイルを起点にした相対パスを指定すると、上記のような正しいパスにならないので注意！