# Docker チュートリアル(坂野研究室編)

Author: 村田

___

##  前書き

この資料は, 主に私や他数名が普段散々口走っている`Docker` というものがなんなのかについて**なんとなく**理解して**とりあえず**使えるようになってもらうためのものです.

詳細な設定などについては, 今後のメンバーに期待して今回は省きます.

重要なことは, "なんで `Docker` を使うのか", "どうやって使うのか" を理解することにあります.

___

なお, この資料は読み物として書いているため、一部伝わりにくい内容があるかと思います。
この資料をもとにスライドを作成しゼミで解説していくので、とりあえずざっと読んでもらえればと思います。

また、理解のためにあえて間違った記述をしている箇所があります。
ざっと理解できた方は疑問点を調べてみてください。
___

## 目次

本資料の構成は以下のとおりです.

1. 仮想環境, コンテナについて
   - 仮想環境とは?
   - なぜ仮想環境が必要なのか？("ローカルが汚れる"とは一体なんなのか？)
   - コンテナとは?
   - `Docker`って何者？
   - WSLって結局なんなの？
2. 研究室で`Docker`が必要な理由
   - 研究室の現状
   - どの端末でも実行できるプログラムの作成を目指して
   - 研究室サーバのススメ(つよつよPCを汚すな)
3. 誰でも(理論的には)できるDockerの使い方
   - インストール(vscode)
   - .devcontainer の扱い方
   - Dockerfile には何を書けばいいの？
   - コンテナの中で作業してみよう
4. 簡単なサンプル一覧

___

## 1. 仮想環境とは

windowsなどのOSが動いている状態で動かす別の環境のことを仮想環境と言います(もっと上手い言い方).

例：windows内で`wsl`を使う(ubuntuを使う), windows内でVM(Virtual Machine)を使ってwindowsXPを使うなど...

AWSやGoogleColaboratoryのクラウドサービスなんかも仮想環境と言えます。授業でcolabにアクセスしてjupyterを実行したことがあると思いますが、あれ別に一人一人に専用の物理マシンが用意されてはいませんよね。んなこと天下のGoogleもできないでしょうし...
あれらは１つの巨大なサーバーからそれぞれのアカウントの環境を都度作成して動作させているわけで、実体は持ちません。

実体を持たない環境という意味で仮想環境という言葉はしっくりきますね。

### なぜ仮想環境が必要なのか？ ("ローカルが汚れる"とは一体なんなのか)

仮想環境の利点は「別のPCを用意せずに、今ある環境とは別の環境が使える」ことです。

皆さんも普段PCを使っていく上で幾つものアプリケーションやゲームなどをインストールするかと思います。
しかし、一部のアプリケーションは特定の環境やバージョンでしか使用できないものがあります。例えば「東方旧作」などの古いゲームはそのゲームが発売された年代のOSでしかプレイできません。ただそれだけを実行するだけのために古いPCをおいておくのも面倒くさいですよね。それを解決するために仮想的に古いOSを起動し、任意のアプリケーションを実行できるようにすることが重要です。

また、
開発に携わる方々や坂野研究室の一部の人々は「ローカルが汚れる」ことを嫌い、これを避けるために仮想環境を必要とします。これはどういった意味なのでしょうか。~~単にイキっている訳ではありません。~~

例えば、皆さんのPCでコマンドプロンプト(macの人はterminal)を起動して

```sh:bash
pip list
```

を実行してみてください。皆さんのローカルのpythonに入っているモジュールの一覧が出てくると思います。

さて、ではその中で

```py:python
import numpy as np
print(np.pi)
```

を実行するのに必要なモジュールはどれでしょうか？当然`numpy`だけですよね。授業でtensorflowとかインストールしたけど結局今はnumpyとかscikit-learnくらいしか使ってないような人にはそれ以外のモジュールは不要とも言えます。使わないものを残しておいてもあまり良いとは言えません。

マイクラのModを使ったことがある人はModが競合を起こしてクラッシュしたり描画がおかしくなった経験がないでしょうか。(`影MOD`と"`黄昏の森`の相性がゴミなのは結構知られていますよね.)複数のModが1つのMinecraftというゲームのリソースに干渉した結果、整合性が取れなくなってシステムが動作できなくなるのです。
この問題を解消しマイクラを楽しむにはどうすればいいでしょうか？競合するModを取り除けばいいですよね。

これと同じような現象が今後研究していく上で発生するかもしれません。今は複数のモジュールが競合せずに使えていますが、もしPyTorchなどの複数のバージョンのモジュールに依存したモジュールを使うことになった際に、単にインストールしても使えなかったり、そもそもインストールに失敗してしまう可能性があります。この原因は以下のストーリーが考えられます。

あるモジュールAはversion1.8のモジュールBを必要としているとします。しかしローカルに入っているのはversion2.5のモジュールBでした。当然モジュールAを使うためにモジュールBをversion1.8に下げようとします。ここで厄介なことにversion1.8のモジュールBはversion2.0のモジュールCが必要でした。そして残念なことにローカルのpythonのバージョンではモジュールCはversion1.0までしか対応していませんでした。

こんな感じでローカルに何も考えずインストールしまくると、お互いに必要なリソースが絡み合ってしまい処理がうまくいかなくなります。自分でソフトウェアのアンインストールやレジストリファイルの削除ができなくなるような状態のことを指して「ローカルが汚れる」と呼んだりします。

### そんな汚れた環境にいられるか！俺は綺麗な部屋に戻らせてもらう

それができたらどれほど良かったでしょうか。既にあなたのローカル環境は面倒臭いほど色々入り混じってしまっているかもしれません。 ひどい方はOSを初期化したほうが早いでしょう。

汚れちまったもんは仕方がないので、せめて今後使うものは汚さずに使っていきたいですよね。
そこで用いるのが仮想環境です。仮想環境は今のローカル環境とは切り離して新しい空間の中で作業を行うことができます。これによって授業用のプログラムを実行する環境や研究のプログラムを実行する環境など、役割を明確に分けて作業環境を作ることができるため、環境の管理がずっと楽になります。

この仮想環境を作成するシステムの１つがコンテナです。

### コンテナとは

コンテナはさまざまな環境をパッケージ化して管理、実行を容易とするソフトウェアです。

コンテナ船をイメージしてもらえればわかりやすいかと思います。
坂野研というコンテナ船には村田の決定木環境のコンテナや山本くんの洪水予測環境のコンテナ、若林くんのガウス過程回帰の開発環境のコンテナが積み込まれているとします。
高木くんは山本くんのコンテナを取り出して自分のPCで実行することで、山本くんの環境を構築することができ、追実験などを容易に行うことができます。これは研究の共有や引き継ぎにも有効です。

コンテナは、作りたい環境の設定やファイルを記述することで作成し、その環境の構築はコンテナを実行するだけですみます。また、コンテナは後述する性質によって通常のOSよりも早く起動、削除ができ、複数の環境を同時に取り扱う際などに有効です。

コンテナを取り扱う人としては、サーバで複数の環境を管理するエンジニアや、ノートPC、デスクトップ、サーバ機など複数のPCで同一の環境を扱いたいプロジェクトチームなどが考えられます。

### コンテナを扱う上での注意点

コンテナは起動しているOSと同じカーネルを用いる環境のコンテナしか実行することはできません。
具体的に言うと、この後紹介するDockerのコンテナはLinuxカーネルでしか動かないものしか紹介しません。
なんのこっちゃという話ですが、コンテナというシステムの本質的な部分なので順番に説明していきます。

まずカーネルについてです。
カーネルは CPUやメモリなどのハードウェアとOSとの橋渡しを行うコンピュータにおける中心的な役割を担うシステムです。
コンピュータの構成要素としてOSがあることは授業で教わったかと思いますが、OSが各ソフトウェア、プログラムの命令を処理する際に消費するメモリの管理やCPUへの命令の順序を管理するものがカーネルです。カーネルがあるおかげで私たちは複数のソフトを同時に作業することができますし、メモリの消費量を気にしなくて良いのです。
また、カーネルを同じくするOSについては、ソフトウェアを共有できたり、ある程度の互換性を持ち得ます。
例えば同じWindows NTカーネルを基とするWindowsは10から11にバージョンアップしても大半のゲームやアプリケーションをそのまま使えますよね。
要は、カーネルを同じくするOSであればプログラムやシステムに互換性があるということです。

例1. 新幹線の線路だったら「のぞみ」も「こだま」も「さくら」も同じ線路で走れるじゃないですか。あれと似たような感じです。

例2. EU加盟国だったらユーロで支払いできますよね。

しかし、基盤となるカーネルが異なるOSではこれができません。Windowsで使えるソフトがmacOSやUbuntuでは使えないということはザラですし、UbuntuとWindowsを切り替えるにはいちいちシャットダウンが必要です。

例1. 新幹線は在来線の線路を走れませんし、その逆も然りといった感じです。(厳密に言えばこの例は間違っているのは承知しているので悪しからず。なんかイメージではできそうになくないですか？)

例2. EUから離脱したイギリスではユーロ使えませんよね。

これらの理由からUbuntu環境のコンテナをWindowsのPCで実行することは通常できないのです。

では「通常」ではなければどうやるのでしょうか。
今まで使われてきていた技術は、仮想マシンで既存の環境から切り離して新規にカーネルごと起動するという方法でした。Windows内でVMを起動しVMの中でUbuntuを起動するという手段を取ることで、Windowsを起動しながらUbuntuを使うことができるようになります。ただしこの方法ではVMの立ち上げに一般的なOSと同じ時間がかかりますし、WindowsとUbuntuとのファイルのやり取りが非常に面倒臭いという欠点があります。

では今はどうしているのでしょうか？おそらく察しがついていると思いますが、これを実現するのが「WSL2」です

### WSLって結局なんなの？

今皆さんが使っている仮想環境の代表例が「WSL」です。

坂野研に入ってまず「WSL」を使えるようにしたかと思います。その際には$\LaTeX$を使うためと説明し詳細な説明をしていなかったため、WSLが一体なんなのかあまり理解せずに使っていたかと思います。正直、$\LaTeX$はWSLがなくても使えますし、わざわざ使う理由がわからないのも無理がありません。結局「WSL」とは一体何なんでしょう？

WSLとは「Windows Subsystem for Linux」の略です。つまりWindowsでLinuxのOS環境を使うためにあるシステムです。
それだけって思うかもしれませんが、実は結構気持ち悪いことを実現しています。

そもそもWindowsとLinux OSは単にOSが異なるというだけではなく、カーネルと呼ばれる部分から全く異なるシステムです。先にも述べましたが、カーネルがないとそのOSは動きませんしOS上で実行されるソフトウェアももちろん使うことができません。理想を言えばカーネルを世界共通規格にして基本的にどんなPCでも同じプログラムが使えれば最高です。

しかし残念なことに、カーネルは世界共通規格とはなりませんでした。Windowsは商業用のWindows NTカーネルで、オープンソースのLinuxはLinuxカーネルで発展していきました。
そのため、Windows NTカーネルで動いているWindows上でLinuxカーネルが必要なLinux OSは本来実行することは不可能なはずです。はずでした。

しかしWSLはそれを可能にしました。詳しい説明は省きますが、WSL(特にWSL2)は特殊な仮想化技術を使ってWindowsNTカーネルとLinuxカーネルを共存させることを実現しました。

このシステムのおかげで私たちはWindows内でデュアルブートやVMを起動せずにLinuxカーネルを使うことができ、皆さんはWSLを立ち上げるだけでLinuxOSの１つであるUbuntu環境を操作することができるのです。しかもWindowsと相互にファイルのやり取りもできます。

ややこしいのでまとめると、全く異なるOSのwindowsとUbuntuを、windows内で一緒に使えるようにしたものがWSLです。

![イメージ](https://roy-n-roy.nyan-co.page/imgs/windows_wsl_wsl2.png "Windowsでのコンテナ技術とWSL")


<!-- <p>
  <img src="https://roy-n-roy.nyan-co.page/imgs/windows_wsl_wsl2.png" alt="Windowsでのコンテナ技術とWSL" title="Windowsでのコンテナ技術とWSL>
  <span class="caption">Windowsでのコンテナ技術とWSL</span>
</p> -->

> [!NOTE]
> (わかる人にとっては面白いので少し詳細を話すと、WSL2は仮想化支援ソフトのHyper-Vを使っているのですが、hyper-vを起動するとホストOSのWindowsがHyper-Vの仮想環境の１つとしてHyper-Vの管理下に入ります。そしてHyper-VによってWindowsカーネルとLinuxカーネルをそれぞれ動かしています。ざっくり言うとWindowsかLinuxのどっちかしか動かんカーネルなんか使わずに両方動かせるHyper-Vカーネル作ってWindowsとLinux動かせばええやんといった感じです。気が狂ってて面白くないですか？さらに言えば、WSL上のUbuntuはコンテナです。我々はコンテナ上でコンテナを立ち上げていたわけです。)

___

## 2. 研究室でDockerが必要な理由

ぶっちゃけないよ！

と終われたらそもそもこんなもの書かなくて良かったのですが、実際必要になるシーンが出てくる方がいると思いますし、使えた方が便利なのでつべこべ言わず覚えて使ってみましょう。

### 研究室の現状

まず初めに研究室の現状について話していきます。

皆さんお察しの通り、当研究室には十分な量の(高性能な)PCがありません。しかしながら研究には多くのリソースが必要となってきます。例えば私の行った実験ではメモリを256GBフルで使いましたし、それ以外の方でも、実行が１日〜1週間以上かかるような実験がよく見られました。そのため、普段使っているPCとは別で長時間使用できるリソースが必要となってくることが予想されます。

### 研究室サーバのススメ(つよつよPCを汚すな)

この問題を解消するために、研究用のプログラムはサーバ機で実行していただこうと考えています。(現在サーバ機は準備中です。４月下旬あたりには皆さんがアクセスして使えるように必ずします。)

しかし、現在皆さんがノートPCなどで書いたコードをそのままサーバ機で実行しても、ノートPCの環境がサーバ機には存在しないので実行することはできません。

サーバ機に直接環境をインストールしても良いですが、コマンド操作しかできないためミスをした時のリスクが大きいです。個人個人のアカウントを用意するので、たとえ個人のアカウント内で環境がぶっ壊れても特に問題はないですし、そもそもぶっ壊れるほどの強い権限を付与する予定はないので最悪インストールしても構いませんが、スマートではないことは念頭においてください。

それよりもコンテナを使った方が明らかに個人での管理が楽なので、コンテナを使って楽に作業環境を維持していきましょう。

### どの端末でも実行できるプログラムの作成を目指して

今後皆さんにはプログラムとそれを実行する環境の２つを１つのセットとして開発を行なっていただきたいです。つまりDockerを使って開発環境をコンテナ化して管理して欲しいです。コンテナ化について難しいことは特にないです。プログラムが入ったフォルダー内　に後述するコンテナの設定ファイルを作成するだけです。これによって、ノートPCで作ったコードとコンテナの設定ファイルが入ったフォルダーのみをサーバ機に送るだけで、サーバ機ではコンテナを立ち上げることでノートPCと同じ環境を構築でき、コード実行も容易に行うことができます。

今回書いているDockerについてもサーバ機にインストールしたものを用意するので、皆さんが行うのは

1. サーバ機にアクセス
2. Dockerコンテナを立ち上げる
3. プログラムの実行、編集

この３点になります。

サーバ機にアクセスすることを難しく考える必要はありません。単純なSSHを使うだけですし、VSCodeならボタンをポチポチするだけで使うことができます。上記３点の操作はVSCodeなら上記操作を最小４回のクリックで行えます。

___

## 3. 誰でも(理論的には)できるDockerの使い方

さてそれではDockerの使い方について紹介していきます。

前提として、今資料ではVSCodeを用いた方法を紹介します。VSCode以外の方はコマンドラインで完結する方法も今後紹介するかもしれませんが今回はスルーします。

同士椎木が大体まとめてくれているのでその[資料](https://github.com/momijiyuki/intro_docker)もパクって(参考にして)進めていきます.

あと、wslは導入済みの前提で行きます.
まだ入れていない, 別のPCにWSLをインストールしたいという方はやはり同士椎木のWSLインストール方法を参考にしてみてください.
一応powershellスクリプトも作ったので面倒臭い人は言ってください.

偉大なるパクり元: <https://github.com/momijiyuki/intro_docker>

### インストール(WSL, LinuxOS)

とりあえずpowershellでwslをupdateしましょう

```ps1:powershell
wsl --update
```

そうしたらwslを起動しシェルを立ち上げて以下のコードを順番に実行します.

```bash:wsl
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
sudo sh -c 'echo "[boot]\nsystemd=true" > /etc/wsl.conf'
```

ここまでできたらwslを再起動しましょう.

```ps1:powershell
wsl --shutdown
```

試しに以下のコマンドをwslで実行してみましょう。helloの表示が出ればインストールは成功です。

```sh:wsl
docker run --rm hello-world
```

### インストール(Mac)

Macユーザーの方は超簡単です。
HomeBrewで

```zsh:zsh
brew install --cask docker
```

を実行すればインストールできます.

もしHomebrewを使わないという場合は,

- [intelチップ](https://desktop.docker.com/mac/main/amd64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-amd64)

- [appleチップ](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-arm64)

のどちらかをダウンロードすればインストーラがダウンロードされるので、自分のCPUに合わせてダウンロードしてください。
あとは起動してインストールするだけです。

>[!WARNING]
>DockerDesktopがバックグラウンドで実行されている必要があります。デーモンが動いていませんとエラーが出たら起動し直してください。

### インストール(それ以外, 頑なにwindowsしか使わない方)

知らん

### Dockerコンテナの構成

vscodeのRemote Containersという拡張機能を使用することで、コンテナの起動や管理が非常に楽になります。そのため、 VSCodeありきで話していきます。というかコマンドラインでの方法は覚えてないです。

Remote Containersでは、1つのコンテナ環境は`.devcontainer`というフォルダで管理します。このフォルダの中に`devcontainer.json`と`Dockerfile`という２つの設定ファイルを配置することで,vscode内のボタン操作1つでコンテナを起動することができます。図示すると

```tree
.
├── .devcontainer/
│   ├── devcontainer.json # vscodeの設定
│   ├── Dockerfile
│   └── requirements.txt
├── 実行したいプログラムが入ったフォルダなど/
│   ├── lib/
│   └── main.py
```

のような構成になります.

### devcontainer.json

`devcontainer.json`は VSCodeでコンテナを作成する際に使用する設定を記述するファイルです.
どのDockerfileを使って作成するかやコンテナ内での VSCodeの設定や使用する拡張機能などを記述します。
基本的には以下の内容を使いまわしていけば問題ないです．

```json:devcontainer.json
{
	"name": "python 3.11",
	"build": { "dockerfile": "Dockerfile" },
	"runArgs": ["--init", "--name", "py_workbench"],
	"customizations": {
		"vscode": {
			"settings": {
                "diffEditor.ignoreTrimWhitespace": false,
				"explorer.openEditors.visible": 0,
                "files.insertFinalNewline": true,
                "files.trimTrailingWhitespace": true,
				"markdown-preview-enhanced.scrollSync": false
				// "notebook.lineNumbers": "on"
			},
			"extensions": [
				"oderwat.indent-rainbow",
				"yzhang.markdown-all-in-one",
				"yzane.markdown-pdf",
				"shd101wyy.markdown-preview-enhanced",
				"donjayamanne.python-extension-pack"
				// "ms-toolsai.jupyter",
			]
	  	}
	}
}
```

余計なものはいらないという方には以下の内容をお勧めします．

```json:devcontainer.json
{
    "name": "実験環境",
    "build": {
        "dockerfile": "Dockerfile"
    }
}
```

まあ，特別な理由がなければ上のほうが使いやすいです．基本的な拡張機能が大体入っていますし

### Dockerfile

Dockerfileは作成するコンテナ環境の設定や作成時に実行するコマンドなどをまとめた設計書のような存在です。

以下のサンプルを例に見ていきましょう。

```Dockerfile:Dockerfile
FROM ubuntu:22.04
USER root

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3.11-dev \
    python3-pip \
    python3-tk && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade pip
RUN python -m pip install numpy scikit-learn matplotlib
```

1行目

```Dockerfile
FROM ubuntu:22.04
```

では使用するDockerイメージを定義しています。DockerイメージとはUbuntuやWindowsなどのOSやそこに使われているコマンド、アプリケーションといった実行環境をまとめたテンプレートです。ubuntuなどの代表的なOSに限らず、pythonを実行するための`python:3.10`環境のように特定の用途に適した環境も存在します.これらはdockerHubと呼ばれるサイトにまとめられているので、自分の欲しい環境があれば検索してみると誰かが作ってくれているかもしれません。Dockerfileでは`FROM ~~`で使用するイメージを定義します。

2行目

```Dockerfile
USER root
```

ではコンテナ内でのログインユーザーを設定します。ここではroot:管理者ユーザーでログインすることを定義しています。そのため、このコンテナ内ではコードの実行に際して`sudo`をつける必要がなくなったり色々できます。

これ以外にも、一般ユーザーの`hoge`などを定義することももちろんできます。まあ自分で使う環境だったらrootでいいと思います。

3行目

```Dockerfile
ENV DEBIAN_FRONTEND=noninteractive
```

はコンテナの環境変数の設定をしています。
具体的には

4行目以降

```Dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3.11-dev \
    python3-pip \
    python3-tk && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*
```

ここはコンテナ環境が作成されてから実行されるコマンド類になります。
実行されているコマンドは見ての通り、LinuxOSをインストールしてから最初に実行するべきコマンドです。aptパッケージをアップデートして必要なライブラリをインストールしています。ここではpython3.11をインストールしていますね。

`RUN ~`で実行したいコマンドを選択しています。この環境はUbuntuなのでubuntuのコマンドですが、CentOSなどの別の環境などではそれぞれのコマンドを実行します。

最後2行

```Dockerfile
RUN python -m pip install --upgrade pip
RUN python -m pip install numpy scikit-learn matplotlib
```

最後も単純で、`RUN`によってpythonのpipをupdateし、pythonに必要なライブラリをインストールしています。
これらのコマンドを事前にDockerfileに記述しておくことによって、コンテナを起動するだけで諸々の設定がすでに完了した環境が用意できます。

### コンテナの中で作業してみよう

ではコンテナを作成、起動してみましょう。

基本的にはWSLの中で作業するか，サーバに接続して作業してもらうかの２択に今後なっていくことを期待しています．
ついでに言えば，gitで管理してgithubでグループ内共有とかできるようになるともっといいですね．
まあそこらへんは焦らずやってもらって大丈夫なのでまずはDockerに慣れていきましょう．

サンプルファイルは[ここ](www.example.com)においているのでフォルダごとダウンロードしてください。(できる方はクローンした方が早いです)


### コンテナの作り方

先ほどまではコンテナを立ち上げることをやってもらいましたが，実際は自分の作業環境は自分で作成して管理することが基本です．
目標としては，皆さんが実験を行っている環境をコンテナとして作成できるようになれば上出来です．

しかしいきなり全部設定して作成するのは難しいと感じると思います．
そのため，簡単にコンテナを作成する方法について紹介していきます．
(ここで紹介する方法はあくまで最小用件で作成する方法です．もっと凝った設定を行いたい方は自分で調べるか私たちに聞いていただければと思います．)

まず，`devcontainer.json`についてはいじる必要はありません．これはvscode側での設定のため，サンプルの内容をそのままコピーしてもらえれば大丈夫です．

問題となるのが`Dockerfile`ですが，これも難しいことはありません．

pythonユーザーの方はとりあえず以下を記述してください．

```Dockerfile
FROM python:3.11-bullseye
USER root
RUN mkdir -p /root/workspace
# COPY requirements.txt /root/workspace
WORKDIR /root/workspace

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3-tk && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade pip
# RUN pip install -r requirements.txt
RUN python -m pip install numpy scikit-learn matplotlib
```

この内容でpythonのコンテナを作るうえでほぼ確実に必要な操作がまとまっています．

ここまで書けたら，一度コンテナを作成してコンテナ内に移ってください．

コンテナに入ったら，実行したいプログラムに必要なライブラリをpipでインストールしてください．

```sh
pip install ~
```

インストールし，プログラムが実行できることを確認したら,

```sh
pip freeze > .devcontainer/requirements.txt
```

を実行してpipの設定を`requirements.txt`に保存します．

その後，

```Dockerfile:Dockerfile
RUN python -m pip install numpy scikit-learn matplotlib
```

の行を削除し，コメントアウトしていた行を解除してください．
これによって必要なライブラリが`requirements.txt`で一括でインストールされるようになります．

ここまで出来たらvscodeの`コンテナをリビルド`を実行し，エラーが出ずにコンテナが再起動されれば成功です．

ここまで実行した後のフォルダ構成，`devcontainer.json`, `Dockerfile`は以下の通りです．

```tree
.
├── .devcontainer/
│   ├── devcontainer.json
│   ├── Dockerfile
│   └── requirements.txt
├── main.py
```

```json:devcontainer.json
{
	"name": "python 3.11",
	"build": { "dockerfile": "Dockerfile" },
	"runArgs": ["--init", "--name", "py_workbench"],
	"customizations": {
		"vscode": {
			"settings": {
                "diffEditor.ignoreTrimWhitespace": false,
				"explorer.openEditors.visible": 0,
                "files.insertFinalNewline": true,
                "files.trimTrailingWhitespace": true,
				"markdown-preview-enhanced.scrollSync": false
				// "notebook.lineNumbers": "on"
			},
			"extensions": [
				"oderwat.indent-rainbow",
				"yzhang.markdown-all-in-one",
				"yzane.markdown-pdf",
				"shd101wyy.markdown-preview-enhanced",
				"donjayamanne.python-extension-pack"
				// "ms-toolsai.jupyter",
			]
	  	}
	}
}
```

```Dockerfile:Dockerfile
FROM python:3.11-bullseye
USER root
RUN mkdir -p /root/workspace
COPY requirements.txt /root/workspace
WORKDIR /root/workspace

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3-tk && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

RUN python -m pip install --upgrade pip
RUN pip install -r requirements.txt
```

### Dockerfileのサンプル集

ここでは`Dockerfile`のサンプルを少しだけ紹介します．

> [!TIP]
> Optional information to help a user be more successful.
