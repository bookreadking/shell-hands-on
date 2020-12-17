# shell-hands-on
日常の業務で使えるシェルのTipsを覚えるハンズオンです

## 準備
dockerでCentOS7のコンテナを立ち上げましょう。`-it`オプション指定でコンテナの中にログインします。

```sh
# 「centos:7.7.1908」イメージからコンテナを起動してbashを実行する
# --rmオプションを指定しているので実行が終わったらコンテナは削除され残りません。
docker run -it --rm centos:7.7.1908 /bin/bash
```

ログインしたらまず必要なコマンドをインストールしてください

```sh
# nkfコマンドをインストール
yum install -y epel-release && yum install -y nkf
```

準備が出来たのでコマンドを試しましょう。

## 改行ありのテキストをターミナル上でコピー＆ペーストで/tmp/work.txtファイルに保存しなさい

下記のテキストをクリップボードにコピーしてターミナル上でテキストファイルに保存してください。  

ヒント： catコマンドとリダイレクト(>)を使用してワンライナーで書けます

```
あのイーハトーヴォのすきとおった風、夏でも底に冷たさをもつ青いそら、うつくしい森で飾られたモリーオ市、郊外のぎらぎらひかる草の波。
またそのなかでいっしょになったたくさんのひとたち、ファゼーロとロザーロ、羊飼のミーロや、顔の赤いこどもたち、
地主のテーモ、山猫博士のボーガント・デストゥパーゴなど、いまこの暗い巨きな石の建物のなかで考えていると、みんなむかし風のなつかしい青い幻燈のように思われます。
では、わたくしはいつかの小さなみだしをつけながら、しずかにあの年のイーハトーヴォの五月から十月までを書きつけましょう。
```

<details>
<summary>答え</summary>

`ヒアドキュメント`という書き方で実現できます。

```sh
cat << EOF > /tmp/work.txt
あのイーハトーヴォのすきとおった風、夏でも底に冷たさをもつ青いそら、うつくしい森で飾られたモリーオ市、郊外のぎらぎらひかる草の波。
またそのなかでいっしょになったたくさんのひとたち、ファゼーロとロザーロ、羊飼のミーロや、顔の赤いこどもたち、
地主のテーモ、山猫博士のボーガント・デストゥパーゴなど、いまこの暗い巨きな石の建物のなかで考えていると、みんなむかし風のなつかしい青い幻燈のように思われます。
では、わたくしはいつかの小さなみだしをつけながら、しずかにあの年のイーハトーヴォの五月から十月までを書きつけましょう。
EOF
```

ファイル出力なしに、ヒアドキュメントだけをシンプルに書くとこうなります。
実行してみましょう。

```sh
<<HOGE
創業
令和元年
HOGE
```

`<<HOGE`から次に`HOGE`だけの行が出現するまでの文字列を**標準入力**として扱ってねという意味になりますが、この例ではその標準入力を何にも使ってないので
いきなり終わる感じになります。
最初の例では`EOF`のところが、次の例では`HOGE`なことに気づきましたか？  
これはどこまでがヒアドキュメントの終端かを示す目印でしかなく、どんな文字列でも良いのです。
catに標準入力を渡すと標準出力にそのまま出力することを利用して、それをファイルにリダイレクトすると改行ありでファイルに出力される寸法です。

応用で変数に突っ込むことも可能です。いちいちファイルを用意するのが面倒な場合に使ったりします。

```sh
# $()はバッククォートで囲ってコマンドを実行するのと同じ意味ですが、範囲がわかりやすいのでこちらの方がお勧め
MSG=$(cat<<EOS
hello
world
EOS
)
```

ただ、この例はzshだと改行になりましたが、bashだとhelloとworldの間が半角スペースになります。

より詳しい説明はこちら
https://qiita.com/take4s5i/items/e207cee4fb04385a9952
</details>

## ファイルの文字コードと改行コードをコマンドを使って調べなさい

コマンド一発でできます

<details>
<summary>答え</summary>

```sh
nkf --guess /tmp/work.txt
```

実行すると下記のような出力が得られます
```ts
UTF-8 (LF)
```

`--guess`ファイルの中のバイトの並びを見て自動で判別しようとします。
UTF-8のBOM(0xEF,0xBB,0xBF)がファイルの先頭にあったらUTF-8だな、このバイトの並びが現れたらMS932だな、とかいう感じです。

ちなみに改行コードが混在した場合もちゃんと分かります。

```sh
echo -e "aaaa\nbbbb\r\ncccc"|nkf --guess
ASCII (MIXED NL)
```

</details>

## ファイルを開かずにどんな形式のファイルなのか調べなさい

コマンド一発でできます

<details>
<summary>答え</summary>

```sh
file /tmp/work.txt
```

実行すると下記のような出力が得られます
```
/tmp/work.txt: UTF-8 Unicode text
```

[演習] 色々なファイルを調べてみよう

</details>

## ファイルの指定した範囲の行だけ取得しなさい

<details>
<summary>答え</summary>

`sed -n 開始行,終了行p`で実現出来ます。

```
sed -n 2,4p anaconda-post.log
```

</details>

## ファイルのgrepで合致した行の前後も含めて取得しなさい

<details>
<summary>答え</summary>

`grep -A 前行数 -B 後行数`で実現出来ます。

```
$ grep freetype -A 2 -B 2 anaconda-post.log
No Match for argument: ethtool
No Match for argument: file
No Match for argument: freetype
No Match for argument: gettext
No Match for argument: gettext-libs
```
</details>

## ファイルを検索した結果をgrepしたい

<details>
<summary>答え</summary>

```sh
$ find /etc -type f|xargs grep --color=auto CentOS
/etc/centos-release:CentOS Linux release 7.7.1908 (Core)
/etc/os-release:NAME="CentOS Linux"
/etc/os-release:PRETTY_NAME="CentOS Linux 7 (Core)"
/etc/os-release:CENTOS_MANTISBT_PROJECT="CentOS-7"
```

xargs自体は受け取った毛
```sh
ls | xargs grep あ
は
grep あ work.txt yum.log
のように動くイメージ
```

</details>

## コマンドの実行結果をファイルにいちいち保存せずに別のコマンドで使いたい

<details>
<summary>答え</summary>

プロセス置換を使って実現可能

```sh
$ diff -y -W 10 <(for i in {1,2,3,4}; do echo $i; done) <(for i in {1,2,4,5}; do echo $i; done)
1	1
2	2
3   <
4	4
    >	5
```

</details>
