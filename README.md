# shell-hands-on
日常の業務で使えるシェルのTipsを覚えるハンズオンです

## 準備

<details>
<summary>Dockerコンテナの設定方法</summary>

dockerでCentOS7のコンテナを立ち上げましょう。`-it`オプション指定でコンテナの中にログインします。

```sh
# 「centos:7.7.1908」イメージからコンテナを起動してbashを実行する
# --rmオプションを指定しているので実行が終わったらコンテナは削除され残りません。
docker run -it --rm centos:7.7.1908 /bin/bash
```

ログインしたらまず必要なコマンドをインストールしてください

```sh
yum install -y epel-release && yum install -y nkf && yum install -y git && yum install -y file
```

このリポジトリをcloneしておいてください

```sh
git clone https://github.com/bookreadking/shell-hands-on.git
```

準備が出来たのでコマンドを試しましょう。
</details>

## 1. 改行ありのテキストをターミナル上でコピー＆ペーストで/tmp/work.txtファイルに保存しなさい

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

## 2. ファイルの文字コードと改行コードをコマンドを使って調べなさい

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

## 3. ファイルを開かずにどんな形式のファイルなのか調べなさい

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

```sh
# file /dev/tty
/dev/tty: character special

# file /bin/sh
/bin/sh: symbolic link to `bash'

# file /bin/ls
/bin/ls: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=aaf05615b6c91d3cbb076af81aeff531c5d7dfd9, stripped
```

</details>

## 4. ファイルの指定した範囲の行だけ取得しなさい

<details>
<summary>答え</summary>

`sed -n 開始行,終了行p`で実現出来ます。

```
# sed -n 2,3p price.txt
banana 120円
strawberry 650円
```

</details>

## 5. ファイルのgrepで合致した行の前後も含めて取得しなさい

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

## 6. ファイルを検索した結果をgrepしたい

<details>
<summary>答え</summary>

```sh
$ find /etc -type f|xargs grep --color=auto CentOS
/etc/centos-release:CentOS Linux release 7.7.1908 (Core)
/etc/os-release:NAME="CentOS Linux"
/etc/os-release:PRETTY_NAME="CentOS Linux 7 (Core)"
/etc/os-release:CENTOS_MANTISBT_PROJECT="CentOS-7"
```

xargs自体は受け取った物を引数にコマンドを実行する

`ls | xargs grep あ` は `grep あ work.txt yum.log`のように動くイメージ

</details>

## 7. コマンドの実行結果をファイルにいちいち保存せずに別のコマンドで使いたい

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


## 8. 2つのファイルをjoinして一つにまとめる

```
# cat name.txt
apple りんご
banana バナナ
melon メロン
pineapple パイナップル

# cat price.txt
apple 100円
banana 120円
strawberry 650円
melon 900円
```

上記の2つのファイルから下記のアウトプットを得たい

```
apple 100円 りんご
banana 120円 バナナ
melon 900円 メロン
```

<details>
<summary>答え</summary>

デフォルトは１列目の値が合致する行をまとめてくれます。

```sh
# join <(sort price.txt) <(sort name.txt)
apple 100円 りんご
banana 120円 バナナ
melon 900円 メロン
```

</details>

## 9. ファイルからパターンマッチで抽出する
下記のファイルから、ASU2JS、LCJB...の部分を抽出しなさい

```sh
$ cat user-agent.txt
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; ASU2JS; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; LCJB; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; MAARJS; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; MAFSJS; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; MALNJS; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; MAMIJS; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; MASPJS; rv:11.0) like Gecko
Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; Touch; rv:11.0) like Gecko
```

<details>
<summary>答え</summary>

awkのマッチで正規表現で取得する


```sh
# awk 'match($0, /Mozilla\/5.0 \(Windows NT 6\.3; WOW64; Trident\/7\.0; (.+); rv:11\.0) like Gecko/,a){print a[1]}' user-agent.txt
ASU2JS
LCJB
MAARJS
MAFSJS
MALNJS
MAMIJS
MASPJS
Touch
```
</details>
