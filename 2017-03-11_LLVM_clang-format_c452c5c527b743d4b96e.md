<!--
title:   [翻訳] LLVMコーディング標準3.9.1（４/４）：スタイルの問題
tags:    C++11,LLVM=3.9.1,clang-format,コーディング規約,翻訳
id:      c452c5c527b743d4b96e
private: false
-->
この記事は、[LLVM Coding Standards](http://releases.llvm.org/3.9.1/docs/CodingStandards.html) 翻訳の４/４です。

----
[前：機械的なソースの問題](/tenmyo/items/eb0fd21212ff0184e933) | [目次](/tenmyo/items/5d9fae50d941655350ca)

----

スタイルの問題
------------

### 高位の問題

#### 公開ヘッダファイル **は** モジュール

C++は、モジュール性部門であまりうまく行っていません。そこには真のカプセル化やデータ隠蔽は（あなたは高価なプロトコルクラスを使わない限り）存在しませんが、私たちはそこで行わなければいけません。あなたが公開ヘッダファイル（LLVMソースツリーでは、トップの「`include`」にいます）を書く場合、あなたは機能モジュールを定義しています。

理想的には、モジュールは互いに完全に独立し、そのヘッダファイルには必要最小限の`#include`のみが含まれるべきです。モジュールは、単なるクラス、関数、あるいは名前空間ではありません：それらの集合で、インタフェースを定義します。このインタフェースは、いくつかの関数、クラス、またはデータ構造であってもよいですが、重要な問題はそれらがどのように連携するかです。

一般的に、モジュールは一つ以上の`.cpp`ファイルによって実装されるべきです。これら `.cpp`の各ファイルは、最初にそのインタフェースの定義ヘッダをインクルードする必要があります。これは、モジュールヘッダの依存すべてが適切にモジュールヘッダ自体に追加されていることを保証し、明らかにします。システムヘッダは、翻訳単位のユーザヘッダの後にインクルードされるべきです。

#### `#include`は最低限に

`#include`はコンパイル時間を損ないます。どうしても必要でない場合は行わないでください、特にヘッダファイルでは。

でもちょっと待ってください！使ったり、継承したりするためにクラス定義が必要になることがあります。その場合はどうぞ`#include`してください。ですが、クラスの完全な定義が必要でない場合も多いことに注意してください。もしクラスのポインタや参照を使っている場合、ヘッダファイルは不要です。もし単に関数やメソッドの宣言で返すインスタンスをだけの場合、それは不要です。実際、ほとんどの場合でクラス定義を単に必要としません。そして`#include`するのはコンパイルを加速しません。

この勧めをやりすぎるのは簡単ですが、使用しているヘッダファイルのすべてがインクルードされなくては**なりません**--- 直接または別のヘッダーファイルを介して間接的にそれらをインクルードすることができます。あなたが誤って、モジュールヘッダ内でヘッダファイルのインクルードを忘れていないことを確認するには、（上記のように）実装ファイルの**最初で**あなたのモジュールのヘッダを含めるようにしてください。この方法により、隠れた依存関係が後々に発覚することがなくなります。

#### 「内部」ヘッダは非公開

多くのモジュールは、複数の実装（`.cpp`）ファイルを使うことで複雑な実装を持っています。多くの場合、内部通信インタフェース（ヘルパークラス、余分な機能など）を公開モジュールヘッダファイルに置くことは魅力的です。これをしないでください！

あなたは本当にこのような何かをする必要がある場合は、ソースファイルと同じディレクトリに非公開ヘッダファイルを置いて、それを内々でインクルードしてください。これは、非公開インターフェイスが他者に乱されず非公開であることを保証します。

> **注**
>
> これはpublicクラス自体に余分なメソッド実装を入れても大丈夫です。それはprivate（またはprotected）となり、順調にいきます。

#### 早期終了と`continue`でコードをシンプルに

コードを読む場合に、読む人がコードブロックの理解のためにどれだけの状態とどれくらいの分岐を忘れないようにしなければならないかを覚えておいてください。なるべくインデントを減らすことは、コードを理解することをより困難にはしません。これを行う1つの偉大な方法は、早期終了し、長いループで`continue`キーワードを利用することです。関数からの早期終了を使用する例としては、この「悪い」のコードを考えてみます：

```cpp:悪い例
Value *doSomething(Instruction *I) {
  if (!isa<TerminatorInst>(I) &&
      I->hasOneUse() && doOtherThing(I)) {
    ... some long code ....
  }

  return 0;
}
```

`'if'`の本文が大きいとすると、このコードにはいくつか問題があります。関数の先頭を見た場合、it isn't immediately clear that this *only* does interesting things with non-terminator instructions, and only applies to things with the other predicates.[^1] 第二に、`if`文はコメントには辛いレイアウトになるため、なぜそれらpredicatesが重要であるか（コメントで）記述することは割合難しくなります。第三に、コード本体の深いところでは、それは余分なレベルインデントされます。最後に、関数の先頭を見た場合、predicateが真でない場合、結果が何であるかは明らかではありません。あなたはそれがnullを返すことを知るためには、関数の最後まで読まなければなりません。
[^1]: 訳注：何をやっているのかぱっと見よく分からないということが書いてありますが、うまく訳せませんでした。

このようなコードフォーマットがより好ましいです：

```cpp:好ましい例
Value *doSomething(Instruction *I) {
  // Terminators never need 'something' done to them because ...
  if (isa<TerminatorInst>(I))
    return 0;

  // We conservatively avoid transforming instructions with multiple uses
  // because goats like cheese.
  if (!I->hasOneUse())
    return 0;

  // This is really just here for example.
  if (!doOtherThing(I))
    return 0;

  ... some long code ....
}
```

これは、これらの問題を修正します。同様の問題は`for`ループで頻繁に起きます。愚かな例は、このようなものです：

```cpp:愚かな例
for (BasicBlock::iterator II = BB->begin(), E = BB->end(); II != E; ++II) {
  if (BinaryOperator *BO = dyn_cast<BinaryOperator>(II)) {
    Value *LHS = BO->getOperand(0);
    Value *RHS = BO->getOperand(1);
    if (LHS != RHS) {
      ...
    }
  }
}
```

非常に小さなループでは、この構造の問題はありません。10〜15行を超えた場合、一目で理解することは困難になります。この種のコードの問題は、あっという間にネストされてしまうことです。それはコードの読み手は、ループ内で何が行われているか把握するために、非常に多くのコンテキストを覚えておかなくてはならないことを意味します。なぜなら、彼らは`if`条件に`else`等があるかどうかを知りません。次のようなループを構成することが非常に好ましいです：

```cpp:好ましい例
for (BasicBlock::iterator II = BB->begin(), E = BB->end(); II != E; ++II) {
  BinaryOperator *BO = dyn_cast<BinaryOperator>(II);
  if (!BO) continue;

  Value *LHS = BO->getOperand(0);
  Value *RHS = BO->getOperand(1);
  if (LHS == RHS) continue;

  ...
}
```

これには、機能について早期終了を利用することの利点がすべてあります：ループのネストを減らし、条件に該当する理由を簡単に記述でき、そして`else`を気にしなくてよいことが明らかです。ループが大きい場合は、これにより非常に分かりやすくなります。

#### `return`後に` else`を使用しない

上記と同様の理由（インデントの減少と読みやすさ）から、制御フローの中断 --- `return`、`break`、`continue`、`goto`等 --- の後に‘’else’‘や‘’else if'`を使わないでください。*悪い*例を示します：

```cpp:悪い例
case 'J': {
  if (Signed) {
    Type = Context.getsigjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_sigjmp_buf;
      return QualType();
    } else {
      break;
    }
  } else {
    Type = Context.getjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_jmp_buf;
      return QualType();
    } else {
      break;
    }
  }
}
```

次のように書く方が良いです：

```cpp::好ましい例
case 'J':
  if (Signed) {
    Type = Context.getsigjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_sigjmp_buf;
      return QualType();
    }
  } else {
    Type = Context.getjmp_bufType();
    if (Type.isNull()) {
      Error = ASTContext::GE_Missing_jmp_buf;
      return QualType();
    }
  }
  break;
```

またはいっそのこと（今回の場合は）：

```cpp:思い切った例
case 'J':
  if (Signed)
    Type = Context.getsigjmp_bufType();
  else
    Type = Context.getjmp_bufType();

  if (Type.isNull()) {
    Error = Signed ? ASTContext::GE_Missing_sigjmp_buf :
                     ASTContext::GE_Missing_jmp_buf;
    return QualType();
  }
  break;
```

この案はインデントと、コードを読み取るときに覚えておかなくてはならないコードの量を減らします。

#### Predicate[^2]はループから関数へ
[^2]: 訳注：Pythonでのany()やall()のような処理のこと。主に関数型言語で述語関数とか叙述関数とか呼ばれているようです。

成否判定だけの小さなループを書くことは非常に一般的です。これを書く様々な方法はありますが、この種のものの例は次のとおりです：

```cpp:例
bool FoundFoo = false;
for (unsigned I = 0, E = BarList.size(); I != E; ++I)
  if (BarList[I]->isFoo()) {
    FoundFoo = true;
    break;
  }

if (FoundFoo) {
  ...
}
```

この種のコードを書くのは厄介であり、ほとんど常に悪い兆候です。この種のループの代わりに、私たちはpredicateを計算するために関数（staticにできます）を使うことを、非常に好みます。次のようなコード構成が好ましいです：

```cpp:好ましい例
/// 指定されたリストにfooである要素を含む場合はtrueを返す。
static bool containsFoo(const std::vector<Bar*> &List) {
  for (unsigned I = 0, E = List.size(); I != E; ++I)
    if (List[I]->isFoo())
      return true;
  return false;
}
...

if (containsFoo(BarList)) {
  ...
}
```

これを行うには多くの理由があります：インデントを減らし、しばしば共有できる同じチェックを行う他のコードとの重複を排除します。さらに重要なのは、関数の*命名を強制*し、それにコメントを書くことを強制します。この愚かな例では、多くの値を追加しません。ですが条件が複雑な場合は、predicateクエリをより簡単に理解できるようになるでしょう。BarListがfooを含むかをどのようにチェックするのかについてインラインでの詳細に直面する代わりに、関数名を信頼してより良い局所性で読んでいけます。

### 低位の問題

#### 型、関数、変数、および列挙子への適切な命名

下手に選ばれた名前は、読者に誤解を与え、バグを引き起こす可能性があります。私たちは、*わかりやすい*名前を使うことがどれだけ重要か、とても十分に強調しきれません。常識の範囲で、要素の意味と役割に一致する名前を選んでください。よく知られていない限り略語は避けてください。良い名前を選んだ後、名前に一貫した大文字を使ってください。ブレは正確な綴りのために、利用者に対して各APIを覚えるか探すことを求めます。

一般に、名前はキャメルケース（例：`TextFileReader`と`isLValue()`）でなければなりません。種類の異なる宣言は異なるルールを持っています：

-   **型名** （クラス、構造体、列挙型、typedef等を含む）は、名詞かつ大文字（例えば`TextFileReader`）で始まる必要があります。
-   **変数名**は名詞（状態を表すように）である必要があります。名前はキャメルケースで、大文字（例えば`Leader`や`Boats`）で始まる必要があります。
-   **関数名**は動詞（アクション表すように）であるべきで、コマンドのような関数は命令である必要があります。名前はキャメルケースで、小文字（例えば`openFile()`や`isFoo()`）で始まる必要があります。
-   **列挙型宣言**（例えば `enum Foo {...}`）は型のため、型の命名規則に従ってください。列挙型の一般的な使用法は、共用体(union)の弁別や、またはサブクラスの情報提供です。列挙型は、このような何かのために使われる場合、`Kind`接尾辞（例えば`ValueKind`）を持つ必要があります。
-   **列挙子**（例えば`enum { Foo, Bar }`）と**パブリックメンバ変数**は、型と同様に大文字で始まる必要があります。列挙子は、自分の小さな名前空間またはクラス内で定義されていない限り、列挙型の宣言名に対応する接頭辞を持っている必要があります。たとえば、`enum ValueKind { ...};`は`VK_Argument`、` VK_BasicBlock`といったような列挙子を含むでしょう。便利な定数としての列挙子は、接頭辞の要件は免除されます。

```cpp:定数列挙子の例
enum {
  MaxSize = 42,
  Density = 12
};
```

例外として、STLクラスを模倣するクラスは、アンダースコアで区切られた小文字の単語（例えば`begin()`、`push_back()`と`empty()`）のSTLのスタイルでメンバ名を持てます。複数のイテレータを提供するクラスは`begin()`と`end()`に特異な接頭辞を追加する必要があります（例えば `global_begin()`と `use_begin()`）。

```cpp:良い名前と悪い名前の例
class VehicleMaker {
  ...
  Factory<Tire> F;            // Bad -- abbreviation and non-descriptive.
  Factory<Tire> Factory;      // Better.
  Factory<Tire> TireFactory;  // Even better -- if VehicleMaker has more than one
                              // kind of factories.
};

Vehicle MakeVehicle(VehicleType Type) {
  VehicleMaker M;                         // Might be OK if having a short life-span.
  Tire Tmp1 = M.makeTire();               // Bad -- 'Tmp1' provides no information.
  Light Headlight = M.makeLight("head");  // Good -- descriptive.
  ...
}
```

#### アサートたっぷり

「`assert`」マクロを最大限に使います。すべての前提条件と仮定をチェックすると、バグ（あなたのものとは限りません）がアサーションによって早く発見できるかは分かりませんが、劇的にデバッグ時間を削減します。「`<cassert>`」ヘッダファイルは、おそらくすでに含まれているので、追加のコストはかかりません。

さらに、デバッグを支援するために、アサーション文になんらかのエラーメッセージ（アサーションにかかった場合に出力される）を入れてください。これは、アサーションの発生原因とそれについて何をすべきかを、未熟なデバッガが理解する助けとなります。

```cpp:一つの完全な例
inline Value *getOperand(unsigned I) {
  assert(I < Operands.size() && "getOperand() out of range!");
  return Operands[I];
}
```

```cpp:多くの例
assert(Ty->isPointerType() && "Can't allocate a non-pointer type!");
assert((Opcode == Shl || Opcode == Shr) && "ShiftInst Opcode invalid!");
assert(idx < getNumSuccessors() && "Successor # out of range!");
assert(V1.getType() == V2.getType() && "Constant types must be identical!");
assert(isa<PHINode>(Succ->front()) && "Only works on PHId BBs!");
```

もうわかりましたね。

過去には、コードに到達すべきではないと示すためにアサートが使われました。

```cpp:典型例
assert(0 && "Invalid radix for integer literal");
```

これにはいくつかの問題があり、主なものは、いくつかのコンパイラはアサーションを理解しない可能性があり、あるいはアサーションの部分でreturnが抜けていると警告を出すことです。

今日、私たちはより良いものを持っています：`llvm_unreachable`：

```cpp:好ましい例
llvm_unreachable("Invalid radix for integer literal");
```

アサーションを有効にすると、ここに到達した場合にメッセージを表示し、プログラムを終了します。アサーションが無効になっている場合（つまりリリースビルドで）、`llvm_unreachable`はこの分岐のコード生成は省略可能だというコンパイラへのヒントとなります。コンパイラはこれをサポートしていない場合は、「abort」実装にフォールバックされます。

もう一つの問題は、アサーションが無効になっている場合に、アサーションによってのみ使用される値で「未使用値」という警告が生成されるということです。

```cpp:警告となる例
unsigned Size = V.size();
assert(Size > 42 && "Vector smaller than it should be");

bool NewToSet = Myset.insert(Value);
assert(NewToSet && "The value shouldn't be in the set yet");
```

２つの興味深い例があります。最初の例では、`V.size()`の呼び出しはアサートのためにのみ有用であり、アサーションが無効になっている場合に実行されたくありません。このようなコードは、アサート自体に呼び出しを移動する必要があります。第二の例では、呼び出しの副作用はアサートが有効かどうかに関わらず起きなければなりません。この場合、警告を無効にするには値をvoidにキャストする必要があります。具体的には、このようなコードを記述することが好ましいです：

```cpp:好ましい例
assert(V.size() > 42 && "Vector smaller than it should be");

bool NewToSet = Myset.insert(Value); (void)NewToSet;
assert(NewToSet && "The value shouldn't be in the set yet");
```

#### `using namespace std`を使わない

LLVMでは、標準名前空間のすべての識別子について、「`using namespace std;`」に頼るのではなく、「`std::`」接頭辞を明示することを好みます。

ヘッダファイルで、`'using namespace XXX'`ディレクティブを追加することはヘッダを`#include`するソースファイルの名前空間を汚染します。これは明らかに悪いことです。

実装ファイル（例えば`.cpp`ファイル）では、ルールはスタイルルールの詳細ですが、それでも重要です。基本的に、明示的な名前空間接頭辞はどこの何を使っているかがわかり、コードを**明解**にします。また、LLVMコードや他の名前空間との間で名前空間の衝突が起きないため、**よりポータブル**になります。将来のC++標準の改訂では`std`名前空間へのシンボル追加もあり、異なる標準ライブラリの実装は異なるシンボル（潜在的なものでない）を公開するため、移植性のルールは重要です。ですので、私たちはLLVMで`'using namespace std;'`を決して使いません。

一般的なルールの例外（つまり、`std`名前空間の例外ではありません）は、実装ファイルのためのものです。例えば、LLVMプロジェクト内のすべてのコードは、「llvm」名前空間内のコードを実装します。ですので、それはOK、また実際明解ですし、`.cpp`ファイルは`#include`直後の先頭に`'using namespace llvm;'`ディレクティブがあります。これは、中括弧に基づいたインデントを行うソースエディタ向けに本文のインデントを減らし、概念的なコンテキストをきれいに保ちます。この規則の一般的な形式は、任意の名前空間内のコードを実装する任意の`.cpp`ファイルは、それの（そして親の）名前空間を使用してもよいが、他のものを使用してはならない、ということです。

#### ヘッダ内クラスは仮想メソッドアンカーを提供する

クラスがヘッダファイル内で定義されvtableを持つ（仮想メソッドを持つか、仮想メソッドを持つクラスから派生した）場合、常にクラス内に少なくとも１つのout-of-line仮想メソッドを持つ必要があります。これがないと、コンパイラは、そのヘッダを`#include`した`.o`ファイルすべてにvtableとRTTIをコピーし、`.o`ファイルサイズとリンク時間を増やします。

#### 列挙型を網羅したswitchにdefaultを使わない

`-Wswitch`は、列挙型の値を網羅せず、defaultも無いswitchに警告を出します。列挙型を網羅したswitchにdefaultを書いた場合、新しい要素が列挙体に追加されても`-Wswitch`は警告しません。この種のdefaultを追加することを避けるために、Clangは`-Wcovered-switch-default`警告を持ちます。これはデフォルトで無効になっていますが、Clangの警告をサポートする版でLLVMをビルドする場合は有効になります。

このスタイル要件の影響で、列挙型を網羅したswitchの各caseでreturnしていた場合、GCCはenum句は個々の列挙子だけでなく任意の値が取れることを前提としているため、GCCでLLVMを構築するときに「コントロールが非void型関数の終わりに到達します」に関連する警告が出ます。この警告を抑止するには、switchの後に`llvm_unreachable`を使います。

####ループで毎回`end()`を評価しない

C++は標準の「`foreach`」ループを持たない（それはマクロでエミュレートすることができたり、C++0xで登場したりするかもしれませんが）[^3]ため、私たちはコンテナや様々なデータ構造の始めから終わりまで手動でイテレートするたくさんのループを書きます。よくある間違いは、このスタイルでループを書くことです：
[^3]: 訳注：LLVM3.9.1現在ではforeachを含むC++11を採用しているため、この節は当てはまりません。記載の更新漏れだと思います。

```cpp:悪い例
BasicBlock *BB = ...
for (BasicBlock::iterator I = BB->begin(); I != BB->end(); ++I)
  ... use I ...
```

この作りでの問題は、ループで毎回「`BB->end()`」を評価することです。代わりに、ループ前に一度だけ評価するようなループを書いてください。

```cpp:好ましい例
BasicBlock *BB = ...
for (BasicBlock::iterator I = BB->begin(), E = BB->end(); I != E; ++I)
  ... use I ...
```

注意深い方は２つのループが異なる意味を持つ可能性をすぐに指摘するかもしれません：コンテナ（この場合、BasicBlock）が変更された場合、「`BB->end()`」は毎回のループで値が変わり２回目のループで正しいとは限りませんあなたが実際にこの動作を頼る場合は、最初の形式でループを書いて、あなたが意図的にそれをしたことを示すコメントを追加してください。

なぜ私たちは二番目の形式をが好きなのか（いつ正しいか）？最初の形式でループを書くことは2つの問題があります。まず、ループの開始時に評価するよりも非効率です。この場合、コストはおそらくわずかです --- ループのたびに余分な負荷がかかります。しかしながら、元の式がより複雑である場合、コストは急速に上昇し得ます。次のようなend式を見たことがあります：「`SomeMap[X]->end()`」そしてmapの検索は本当に安くありません。一貫して二番目の形式でそれを書くことによって、あなたは完全に問題を排除し、それについて考える必要はありません。

（さらに大きな）第二の問題は、最初の形式でループを書くことは、ループでコンテナが変更されること（コメントは手軽に確認できるという事実！）を読み手に示唆します。二番目の形式でループを記述した場合、コードは読みやすく、何をするかを簡単に理解でき、ループの本体を見ずともコンテナが変更されていないことは明白です。

またループの2番目の形式は、いくつかの余分なキーストロークが必要となりますが、私たちは強くそれを好みます。

#### `#include <iostream>`禁止

ライブラリファイルで`#include <iostream>`を使うことは**禁止**されています。なぜなら、多くの一般的な実装では、それを含むすべての変換単位に静的コンストラクタを透過的に注入するからです。

その他のストリームヘッダ（たとえば`<sstream>`）の使用はこの点で問題ないことに注意してください。`<iostream>`のみです。しかし、`raw_ostream`の提供する様々なAPIは、ほとんどすべての用途で`std::ostream`スタイルのAPIよりも優れたパフォーマンスを発揮します。

> ** 注 **
>
> 新規コードでは常に、ファイル読み込みに`llvm::MemoryBuffer`APIを、書き込みにraw_ostreamを使ってください。

#### `raw_ostream`の使用

LLVMは軽量で、シンプルで、かつ効率的なストリーム実装を`llvm/Support/raw_ostream.h`に含み、これは`std::ostream`の共通機能をすべて提供します。すべての新規コードで`ostream`ではなく`raw_ostream`を使ってください。

`std::ostream`と異なり、`raw_ostream`はテンプレートではなく、`class raw_ostream`のように前方宣言できます。公開ヘッダは通常`raw_ostream`ヘッダを含みませんが、`raw_ostream`インスタンスへの前方宣言と定数参照を使います。

#### `std::endl`を避ける

`std::endl`修飾子は、指定の出力ストリームに改行を出力する場合`iostream`と共に使われます。それに加えて、出力ストリームをフラッシュします。言い換えると、これらは同等です：

```cpp
std::cout << std::endl;
std::cout << 'n' << std::flush;
```

ほとんどの場合、おそらく出力ストリームをフラッシュする理由はなく、`'\n'`リテラルを使うことをお勧めします。

#### クラス定義内の関数定義で`inline`を使わない

クラス定義内で定義されたメンバ関数は暗黙的にインラインであるため、`inline`キーワードを入れないでください。

```cpp:禁止
class Foo {
public:
  inline void bar() {
    // ...
  }
};
```

```cpp:推奨
class Foo {
public:
  void bar() {
    // ...
  }
};
```

### 微視的詳細

このセクションでは、好ましい低レベルのフォーマットガイドラインを、私たちが好む理由と共に説明します。

#### 括弧の前にスペース

私たちは、フロー制御文の開き括弧の前でのみスペースを入れることを好みますが、普通の関数呼び出しや関数風マクロでは違います。

```cpp:良い例
if (X) ...
for (I = 0; I != 100; ++I) ...
while (LLVMRocks) ...

somefunc(42);
assert(3 != 4 && "laws of math are failing me");

A = foo(42, 92) + bar(X);
```

```cpp:悪い例
if(X) ...
for(I = 0; I != 100; ++I) ...
while(LLVMRocks) ...

somefunc (42);
assert (3 != 4 && "laws of math are failing me");

A = foo (42, 92) + bar (X);
```

これを行う理由は、まったく恣意的ではありません。このスタイルは、制御フロー演算子がより目立ち、式の流れを良くします。関数呼び出し演算子は、後置演算子として非常に強固に結合します。関数名の後にスペースを置くこと（最後の例のように）は、関数の引数リストといっしょの二項演算子について、引数を左側に持ち名前を右側に持つ二項演算のように見えます。具体的には、例のように「`A`」を簡単に読み違えます：

```cpp:例
A = foo ((42, 92) + bar) (X);
```

コードを流し読む場合。関数でのスペースを避けることで、この誤解を避けます。

#### 前置インクリメントの選好

ハード高速ルール：前置インクリメント（`++X'）は後置インクリメント（`X++`）よりも遅くなることはなく、むしろはるかに速い可能性があります。可能な限り前置インクリメントを使って。

後置インクリメントの内容は、インクリメントされる値のコピーを作成し、それを返した後、「作業値」を前置インクリメントするということを含みます。プリミティブ型の場合、これはたいしたものではありません。しかしイテレータでは、巨大な問題になる可能性があります（例えば、いくつかのイテレータはスタックを含み、それらにオブジェクトを設定します...イテレータをコピーすると、それらのコピーコンストラクタを呼ぶことにもなります）。一般に、いつも前置インクリメントを使う習慣を身につれば、問題は起きません。

#### 名前空間のインデント

大概、私たちは可能な限りインデントを減らすよう努力しています。これはコードをひどい折り返しなしに80列に収めるためにも便利ですが、コードを簡単に分かるようにすることにも便利です。これを促進するとともに、非常に深いネストの機会を避けるために、名前空間はインデントしません。読みやすくなる場合、どの名前空間が`}`で閉じられるているか示すコメント追加することは自由です。

```cpp:例
namespace llvm {
namespace knowledge {

/// This class represents things that Smith can have an intimate
/// understanding of and contains the data associated with it.
class Grokable {
...
public:
  explicit Grokable() { ... }
  virtual ~Grokable() = 0;

  ...

};

} // end namespace knowledge
} // end namespace llvm
```

閉じられている名前空間が何らかの理由で明らかであるときは終了コメントをスキップするのも自由です。例えば、ヘッダファイル内の最も外側の名前空間はほとんど混乱の原因となりません。しかし、ソースファイルの途中で閉じた名前空間（名前の有無を問わず）は、おそらく説明を使えます。

#### 無名名前空間

一般的に名前空間の話をした後は、特に無名名前空間について疑問に思うかもしれません。偉大な言語機能である無名名前空間は、名前空間の内容が現在の翻訳単位でのみでしか見えないことをC++コンパイラに伝え、より積極的な最適化を可能にし、シンボル名の衝突の可能性を排除します。無名名前空間は、C++では、Cの関数とグローバル変数にある「static」のようにあります。C++でも「`static`」は使えますが、無名名前空間はより一般的です：これらはファイルに対してクラス全体を非公開にできます。

匿名の名前空間の問題は、本来的に本文のインデントを求めることと、参照の局所性を減らすことです：C++ファイルのrandom関数の定義を見る場合、それがstaticかどうか簡単に分かりますが、それが無名名前空間にあるかどうかを確認するにはファイルの大きなまとまりをスキャンする必要があります。

このため、私たちには単純なガイドラインがあります：無名名前空間は可能な限り小さくし、クラス定義のためだけに使います。

```cpp:OK
namespace {
class StringSort {
...
public:
  StringSort(...)
  bool operator<(const char *RHS) const;
};
} // end anonymous namespace

static void runHelper() {
  ...
}

bool StringSort::operator<(const char *RHS) const {
  ...
}
```

```cpp:NG
namespace {

class StringSort {
...
public:
  StringSort(...)
  bool operator<(const char *RHS) const;
};

void runHelper() {
  ...
}

bool StringSort::operator<(const char *RHS) const {
  ...
}

} // end anonymous namespace
```

これは特に悪く、なぜなら大きなC++ファイルの途中の「`runHelper`」を見た場合、すぐにファイルローカルかどうかを知るすべがないからです。明示的にstaticにされている場合、これはすぐに明らかです。また、それが宣言されたというだけでは、名前空間に「`operator<`」の定義を含める理由になりません。

関連項目
--------

これらのコメントや勧告の多くは他の情報源から抜粋されています。私たちの仕事のための二つの特に重要な書籍は、次のとおりです。

1.  [Effective C++](http://www.amazon.com/Effective-Specific-Addison-Wesley-Professional-Computing/dp/0321334876) by Scott Meyers.[^4] 同じ著者による「More Effective C++」「Effective STL」もまた、興味深く有用です。
2.  [Large-Scale C++ Software Design](http://www.amazon.com/Large-Scale-Software-Design-John-Lakos/dp/0201633620/ref=sr_1_1) by John Lakos

[^4]: 訳注：日本語版は[Effective C++ 第3版](https://www.amazon.co.jp/Effective-%E7%AC%AC3%E7%89%88-ADDISON-WESLEY-PROFESSIONAL-COMPUTI/dp/4621066099)


もし空き時間がありこれらを読んだことがないのであれば：そう、あなたが何かを学ぶことがあります。

----
[前：機械的なソースの問題](/tenmyo/items/eb0fd21212ff0184e933) | [目次](/tenmyo/items/5d9fae50d941655350ca)

----