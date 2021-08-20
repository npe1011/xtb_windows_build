# GrimmeのXTBをWindowsでビルドしたメモ書き
GrimmeらのXTB [https://github.com/grimme-lab/xtb] について、Windows用のバイナリを公開してた人がいたのですが、いつの間にか消えてるので自前でビルドした手順メモです。
MSYS2/MinGW-w64 を使っています。本当はIntelコンパイラ+MKLとかのほうが速いんでしょうけど、まあWindows上で使いたい場面ではそうそう速度に拘ることもないでしょう。2021/8/20時点で最新の6.4.1をビルドしました。crestはめんどくさそうなのとWindowsでやることもないのでスルー。

## 1. MSYS2のインストール
https://www.msys2.org/ からx86_64用のインストーラを落として実行。とりあえず全部デフォルト。
`C:\msys64` 以下にインストール。コマンドラインが立ち上がったら
```
pacman -Syu
```
として諸々をアップデートしたあと、コマンドラインを閉じる。

## 2. 必要なライブラリ等のセットアップ
MSYS2 MinGW 64-bit を立ち上げて以下の通りにして必要なものをインストール（余分なのもあるかもしれないけど気にしない）。確認は全部yで。
ビルドに必要なコンパイラやmeson、ninjaなどのツールと、線形代数ライブラリのBLAS/LAPACKをインストールする。

```
pacman -S base-devel
pacman -S mingw-w64-x86_64-toolchain
pacman -S mingw-w64-x86_64-openblas
pacman -S mingw-w64-x86_64-lapack
pacman -S mingw-w64-x86_64-meson
pacman -S mingw-w64-x86_64-ninja
```

## 3. ビルド
最終的にはMSYS2中でなくて、Windowsのコマンドラインから直接呼ぶ想定なので、インストール先は適当に `home/NAME/opt/xtb-6.4.1` とした。ビルドは `home/NAME/src` 以下でやった。ちょっと詰まった点として、この環境だとPythonのデフォルトの文字コードがWindows用（cp932）になっていて、そのままだとninjaがcodec errorでコケました。なので環境変数でutf-8にしています。
```
export PYTHONIOENCODING=utf-8
export PYTHONUTF8=1
cd ~
mkdir src && cd src
wget https://github.com/grimme-lab/xtb/archive/refs/tags/v6.4.1.tar.gz
tar -xvf v6.4.1.tar.gz
cd xtb-6.4.1
meson setup _build_gcc -Dla_backend=openblas --prefix=$HOME/opt/xtb-6.4.1
ninja -C _build_gcc test
ninja -C _build_gcc install
```
これでビルドは完了。

## 4. Windowsからの実行
諸々のshared library (DLL) はMSYS2/MinGW-w64環境にあって、それがないと実行できない。実行時に必要なDLLは `C:\msys64\mingw64\bin` にあるので、ここに `PATH` を通せばいいが、Windows環境をわざわざ汚染するのはやりたくない。それと `XTBPATH` という環境変数も設定しないとダメ。以上のことから実行用のバッチファイルを作って、それを経由で実行するようにした。
ビルドしたファイルは、`C:\msys64\home\NAME\opt\xtb-6.4.1` にある。フォルダごと適当な場所でコピーして使うことにした。`xtb-6.4.1\bin\xtb.exe` が実行ファイルで、`xtb-6.4.1\share\xtb` にパラメータファイルが入っているのでここを `XTBPATH` に設定する必要がある。 

以下の内容を、`xtb-6.4.1\xtb.bat` として保存。 `CHCP 65001` はターミナルの文字コードをutf-8にしている。これをやらないと一部文字化けする。
 
```xtb.bat
@echo off
CHCP 65001
set PATH=%PATH%;C:\msys64\mingw64\bin
set XTBPATH=%~dp0%share\xtb
%~dp0bin\xtb.exe %*
```

実行するときは、xtb.bat に引数を渡せばそれで実行できる。移動したフォルダに `PATH` を通しても良いと思う。


