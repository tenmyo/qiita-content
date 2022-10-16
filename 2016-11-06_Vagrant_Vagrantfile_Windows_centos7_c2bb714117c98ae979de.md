<!--
title:   Windows+VagrantでCentOS7の公式boxが起動できない("rsync" could not be found on your PATH)
tags:    Vagrant,Vagrantfile,Windows,centos7
id:      c2bb714117c98ae979de
private: false
-->
# TL;DR
[vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest)をインストール(`vagrant plugin install vagrant-vbguest`)して、Vagrantfileを`config.vm.synced_folder ".", "/vagrant", type: "virtualbox"`に書き換える。

# 背景
　WindowsのVagrantでCentOS7のbox(centos/7)を入れようとしたが、`vagrant up`に失敗する。

```:コマンドプロンプト
C:\Users\tenmyo\vm\test>vagrant init centos/7
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

C:\Users\tenmyo\vm\test>vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'centos/7'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'centos/7' is up to date...
==> default: Setting the name of the VM: test_default_1478419157855_72559
==> default: Fixed port collision for 22 => 2222. Now on port 2200.
"rsync" could not be found on your PATH. Make sure that rsync
is properly installed on your system and available on the PATH.
```


# 調査
## 関連Qiita記事
[Windows 7 + Vagrant 1.4.2 でCentOS7のboxが起動できなかったので調べた - Qiita](http://qiita.com/ryozi_tn/items/1edf32f9c5ef174ddd53)
[Windows環境のVagrantでCentOS Atomic Hostを動かす - Qiita](http://qiita.com/tiibun/items/7f17a2c1198f5614bf4f)

### 2016/11/15 こちらの記事でばっちり解説されていました。
[vagrant において、明示的に VirtualBox の機能(vboxsf)を使いフォルダを同期するには、`config.vm.synced_folder ".", "/vagrant", type: "virtualbox"` のように Vagrantfile に設定するとよい - Qiita](http://qiita.com/toby_net/items/6eb74471871ab9fba087)


## 原因
　boxのVagrantfileで `config.vm.synced_folder ".", "/vagrant", type: "rsync"` と共有フォルダ種類を rsync 決め打ちにされていて、 Windows は rsync が入っていないためエラーとなっている。
CentOS公式BoxはVirtualBoxの共有フォルダ(Guest Additions)をあえて無効にしているみたい。[公式リリースノート](https://seven.centos.org/2017/09/updated-centos-vagrant-images-available-v1708-01/)

# 対処
　下記の方法がある。自分は手軽で便利な３にした。

1. 共有フォルダを無効にする
2. rsync をインストールする
3. 共有フォルダ種類を Vagrant デフォルトに再設定する


## 共有フォルダを無効にする
　関連Qiita記事で上がっている方法。 Vagrant ファイルを以下のように`disabled: true`する。

```ruby:Vagrantfile
Vagrant.configure(2) do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true
end
```


## rsync をインストールする
　まっとうな方法


## 共有フォルダ種類を Vagrant デフォルトに再設定する
Vagrant ファイルを以下のように`type: nil`する。

2016/11/15追記：`type:"virtualbox"`でもよいようです。

```ruby:Vagrantfile
Vagrant.configure(2) do |config|
  config.vm.synced_folder ".", "/vagrant", type:nil
end
```