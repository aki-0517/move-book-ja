# Move - デジタルアセットのための言語

スマートコントラクトプログラミング言語は、歴史的にデジタルアセットの定義と管理に焦点を当ててきました。例えば、EthereumのERC-20標準は、デジタル通貨トークンとやり取りするための標準セットの先駆けとなり、ブロックチェーン上でデジタル通貨を作成・管理するための設計図を確立しました。その後、ERC-721標準の導入は重要な進歩を示し、非代替性トークン（NFT）の概念を普及させました。NFTは一意で分割不可能なアセットを表現します。これらの標準は、今日私たちが目にする複雑なデジタルアセットの基盤を築きました。

<!-- ## Move and Digital Assets -->

<!-- note: consider "native" -> "fine-grained" -->

しかし、Ethereumのプログラミングモデルには、アセットのネイティブ表現が欠けていました。言い換えると、外部的にはスマートコントラクトはアセットのように振る舞いましたが、言語自体には固有にアセットを表現する方法がありませんでした。Moveは最初から、アセットのファーストクラス抽象化を提供することを目的とし、アセットについて考え、プログラムするための新しい道筋を開きました。

<!-- Move was initially created in 2018 as part of the Libra project. The language was designed to address shortcomings in existing smart contract languages, especially in handling assets and access control. The Move language aims to provide first-class abstractions for these concepts, improving the safety and productivity of smart contract programming. -->

アセットにとって不可欠な特性を強調することが重要です：

- **所有権：** すべてのアセットは所有者に関連付けられており、これは物理世界における所有権の直接的な概念を反映しています。車を所有するのと同じように、デジタルアセットも所有できます。Moveは所有権を強制し、アセットが_移動_されると、以前の所有者は完全にその制御を失います。このメカニズムにより、明確で安全な所有権の変更が保証されます。

- **複製不可能性：** 現実世界では、固有のアイテムを簡単に複製することはできません。Moveはこの原則をデジタルアセットに適用し、プログラム内で任意に複製されることを防ぎます。この特性は、デジタルアセットの希少性と独自性を維持するために重要であり、物理的アセットの内在的価値を反映しています。

- **破棄不可能性：** 家や車を跡形もなく誤って失うことができないのと同様に、Moveはプログラム内でアセットが破棄または紛失されることを防ぎます。代わりに、アセットは明示的に転送または破棄される必要があります。この特性により、デジタルアセットの意図的な取り扱いが保証され、偶発的な損失を防ぎ、アセット管理における説明責任を確保します。

Moveはこれらの特性をその設計に組み込むことに成功し、デジタルアセットにとって理想的な言語となりました。

## まとめ

- Moveは、デジタルアセットのファーストクラス抽象化を提供するよう設計され、開発者がアセットをネイティブに作成・管理できるようにします。
- デジタルアセットの不可欠な特性には、所有権、複製不可能性、破棄不可能性が含まれ、Moveはこれらを設計に組み込んでいます。
- Moveのアセットモデルは現実世界のアセット管理を反映し、安全で責任のあるアセットの所有権と転送を保証します。

## 参考文献

- [Move: A Language With Programmable Resources (pdf)](https://developers.diem.com/papers/diem-move-a-language-with-programmable-resources/2019-06-18.pdf)
  by Sam Blackshear, Evan Cheng, David L. Dill, Victor Gao, Ben Maurer, Todd Nowacki, Alistair Pott,
  Shaz Qadeer, Rain, Dario Russi, Stephane Sezer, Tim Zakian, Runtian Zhou\*
