<!--
title:   [翻訳] LLVMコーディング標準3.9.1（２/４）：言語、ライブラリ、および標準
tags:    C++11,LLVM=3.9.1,clang-format,コーディング規約,翻訳
id:      e1194d8093b7a79abd5f
private: false
-->
この記事は、[LLVM Coding Standards](http://releases.llvm.org/3.9.1/docs/CodingStandards.html) 翻訳の２/４です。

----
[前：はじめに](/tenmyo/items/5d9fae50d941655350ca#はじめに) | [目次](/tenmyo/items/5d9fae50d941655350ca) | [次：機械的なソースの問題](/tenmyo/items/eb0fd21212ff0184e933)

----

言語、ライブラリ、および標準
-----------------------------------

コーディング標準を用いる、LLVM及びその他LLVMプロジェクトに含まれるソースコードの大半は、C++コードです。いくつかの部位では、環境の制約、歴史的な制限、またはサードパーティ製のソースコード利用に由来して、Cコードが用いられています。全体としては、規格に準拠しモダンでポータブルなC++コードを、実装言語とします。

### C++標準のバージョン

LLVM、Clang、そしてLLDは現在C++11に適合して書かれていますが、ホストコンパイラとしてサポートされる主要なツールチェーンで利用可能な機能に限定しています。LLDBプロジェクトはサポートされるホストコンパイラのセットもより積極的であり、したがって、さらに多くの機能を使用しています。利用できる機能に関わらず、コードは（合理的な範囲で）標準に則りポータブルでモダンなC++11コードであることが期待されます。不要なベンダー固有の拡張等は避けます。

### C++標準ライブラリ

C++標準ライブラリを活用してください。LLVMや関連するプロジェクトは、標準ライブラリ機能を重視し可能な限り用いています。標準インタフェース（策定中含む）から見て標準ライブラリに欠けている機能については、共通サポートライブラリとして、LLVM名前空間内に期待されるインタフェースに沿い実装されます。

標準I/Oストリーム非使用等の、いくつかの例外があります。これらについては、[プログラマーズマニュアル](http://releases.llvm.org/3.9.1/docs/ProgrammersManual.html)により詳細な情報があります。

### 利用するC++11言語とライブラリの機能

LLVM、Clang、およびLLDではC++11を使用していますが、ツールチェーンで利用可能な機能すべてを用いるわけではありません。LLVM内で用いられる機能セットは、MSVC 2013、GCC 4.7、およびClang 3.1でサポートされているものです[^1]。このセットの最終的な定義は、各ツールチェーンによるビルドボットが受け入れるものです。ビルドボットと議論しないでください。しかし、何を想定してよいかの助けとして、下記のような手引きがあります。
[^1]: 訳注：次リリースでは、MSVC2015(Update3)、GCC 4.8、Clang 3.1にアップデートされるようです

各ツールチェーンは、それが何を受け入れるかの良いリファレンスを提供します。

-   Clang: <http://clang.llvm.org/cxx_status.html>
-   GCC: <http://gcc.gnu.org/projects/cxx0x.html>
-   MSVC: <http://msdn.microsoft.com/en-us/library/hh567368.aspx>

ほとんどの場合、MSVCのリストが支配的要因となります。ここに動作することが期待される機能をざっと示します。このリストにない機能は、私たちのホストコンパイラではサポートされそうにありません。

-   右辺値参照: [N2118](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2118.html)
    -   ただし、 `*this` やメンバ関数への修飾については、右辺値参照 *禁止*
-   Static assert: [N1720](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2004/n1720.html)
-   `auto` 型推論: [N1984](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1984.pdf), [N1737](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2004/n1737.pdf)
-   後置戻り型: [N2541](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2541.htm)
-   ラムダ: [N2927](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2927.pdf)
    -   ただし、デフォルト引数と合わせての利用は *禁止* 。
-   `decltype`: [N2343](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2343.pdf)
-   連続する閉じ山かっこ: [N1757](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1757.html)
-   Extern templates: [N1987](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1987.htm)
-   `nullptr`: [N2431](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2431.pdf)
-   厳密に型付けされ、前方宣言可能な列挙型: [N2347](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2347.pdf), [N2764](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2764.pdf)
-   ローカル型や無名型のテンプレート引数: [N2657](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2657.htm)
-   範囲によるforループ: [N2930](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2930.html)
    -   ただし、 `do {} while()` ループの周りは `{}` が必要です。そのため、関数マクロ内の範囲によるforループでも `{}` が要求されます。
-   `override` および `final`: [N2928](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2928.htm), [N3206](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2010/n3206.htm), [N3272](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2011/n3272.htm)
-   アトミック操作とC++11メモリモデル: [N2429](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2429.htm)
-   可変個引数テンプレート: [N2242](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2242.pdf)
-   明示的な変換演算子: [N2437](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2437.pdf)
-   関数のdefault&delete宣言: [N2346](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2346.htm)
    * But not defaulted move constructors or move assignment operators, MSVC 2013
    :   cannot synthesize them.

-   初期化子リスト: [N2627](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2672.htm)
-   委譲コンストラクタ: [N1986](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1986.pdf)
-   デフォルトのメンバ初期化子（非静的データメンバ初期化子）: [N2756](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2756.htm)
    -   明示的に初期化されないスカラメンバでのみ使用してください。非スカラのメンバは、一般により適切なデフォルトコンストラクタを持っており、そしてMSVC 2013では波かっこの初期化子リストによる問題を抱えています。

C++11標準ライブラリでサポートされた機能は、十分に追跡されていないうえ、とても大きすぎます。ほとんどの標準ライブラリは、C++11ライブラリのほぼ全てを実装しています。あるいはLinuxでのサポートが共通項となるでしょう。libc++では、テストや文書化は十分ではありませんが、ほぼ完全であることが見込めます。場合にもよりますが。libstdc++の場合は、[the libstdc++ manual](http://gcc.gnu.org/onlinedocs/gcc-4.7.3/libstdc++/manual/manual/status.html#status.iso.2011)で詳細な文書化がなされています。一般には問題ありませんが、注意を要するわずかな不足が数点あります。

-   type traitsの一部が未実装です
-   正規表現ライブラリはありません。
-   アトミックライブラリの大部分は十分に実装されていますが、フェンスがありません。幸いなことに、それらはほとんど必要ありません。
-   ロケールのサポートは不完全です。

これら以外については、ビルドボットが知らせるまでは標準ライブラリが使えると想定して進めてよいでしょう。これら不確実な点について進めており、Linuxシステム上でテストすることができない場合は、そういった機能の使用を最小限に抑え、Linuxビルドボットがバグを検出しないか注視することが最善のアプローチとなるでしょう。例として、type traitの未実装を踏んでしまった場合は、LLVMのtraitsヘッダにそれのエミュレートを追加します。

### その他の言語

Go言語で記述されたコードは、以下の書式ルールの対象にはなりません。その代わりに、 [gofmt](https://golang.org/cmd/gofmt/) ツールによる整形を採用しています。

Goコードは慣習に倣うよう努力すべきです。[Effective Go](https://golang.org/doc/effective_go.html) および [Go Code Review Comments](https://code.google.com/p/go-wiki/wiki/CodeReviewComments) [^2]の２つがこれの良いガイドラインとなります。

[^2]: 翻訳記事：[#golang CodeReviewComments 日本語翻訳](http://qiita.com/knsh14/items/8b73b31822c109d4c497)

----
[前：はじめに](/tenmyo/items/5d9fae50d941655350ca#はじめに) | [目次](/tenmyo/items/5d9fae50d941655350ca) | [次：機械的なソースの問題](/tenmyo/items/eb0fd21212ff0184e933)

----