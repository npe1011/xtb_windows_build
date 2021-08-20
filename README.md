# GrimmeのXTBをWindowsでビルドしたメモ書き
GrimmeらのXTB [https://github.com/grimme-lab/xtb] について、Windows用のバイナリを公開してた人がいたのですが、いつの間にか消えてるので自前でビルドした手順メモです。
MSYS2/MinGW-w64 を使っています。本当はIntelコンパイラ+MKLとかのほうが速いんでしょうけど、まあWindows上で使いたい場面ではそうそう速度に拘ることもないでしょう。

## 1. MSYS2のインストール
https://www.msys2.org/ からx86_64用のインストーラを落として実行。とりあえず全部デフォルト。
`C:\msys64` 以下にインストール。コマンドラインが立ち上がったら
```
pacman -Syu
```
として諸々をアップデートコマンドラインを閉じる。

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

