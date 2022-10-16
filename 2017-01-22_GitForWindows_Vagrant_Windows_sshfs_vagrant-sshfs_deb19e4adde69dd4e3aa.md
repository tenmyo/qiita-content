<!--
title:   Windowsホストからvagrant-sshfsを使ってフォルダ共有してみた
tags:    GitForWindows,Vagrant,Windows,sshfs,vagrant-sshfs
id:      deb19e4adde69dd4e3aa
private: false
-->
# やってみた
[CentOSのブログで紹介](https://seven.centos.org/2016/12/updated-centos-vagrant-images-available-v1611-01/)されていたので、[vagrant-sshfs](https://github.com/dustymabe/vagrant-sshfs)を入れてみました。
CentOSさん的にはVirtualBoxの共有フォルダよりも、NFSやsshfsを推奨しているようです。

サイトではsftp-serverコマンドが要件となっていますが、[Git for Windows](https://git-for-windows.github.io/) 付属の Git Bash からであれば、`/usr/lib/ssh`にパスを通しておくだけでOKでした。（標準で同梱されていたみたいです）

MSYS2の方は、pacmanでopenssh入れて同様にパスを通せばOKではないかと思います。
Cygwinの方はサイトの記載通り、openssh入れるだけでOKぽいです。


# 感想
VirtualBox Guest Additionsで共有していたころは、 `clang-tidy -fix`が上書きに失敗する（[VirtualBox#4890](https://www.virtualbox.org/ticket/4890)）ので困っていたのですが、sshfsでの共有にしたらうまくいくようになりました。
~~今のところパフォーマンスの違いはあまり感じません。~~

# 積み残し
- VirtualBox Guest Additions との性能比較
- IssueやPR（MSYS2等でsftp-serverが入っていてもvagrant-sshfsが自動検出してくれない）*1 *2

*1: `VagrantPlugins::SyncedFolderSSHFS::SyncedFolder::find_executable`で、ある程度環境に合わせてパスを登録してくれているのだけれど、Cygwinでインストールされる`/usr/sbin`だけで、`/usr/lib/ssh`は登録してくれない。
*2: 環境分岐に使っている `Vagrant::Util::Platform` で、MSYS2として検出するか、Cygwinと同じとして検出してほしいが、そうなってない。（コマンドプロンプトと同じくくりになってしまう）

# 追記：2017/01/24
取り急ぎ普段の使い方で時間を計ってみました。
ホストはSSDで、/vagrant以下のソースコードを、out-of-sourceでtmpfsにビルド。（≒Readのみ）
※rsyncはvagrant up時にエラーが出て、まれにいくつかのファイルがコピーされないことがありました。

|        | real        | user       | sys       |
|:------:|:-----------:|:----------:|:---------:|
| sshfs  | 11m38.785s  | 1m15.201s  | 0m26.677s |
| vboxsf |  1m59.664s  | 1m20.970s  | 1m29.213s |
| rsync  |  0m38.236s  | 1m 3.831s  | 0m 7.747s |