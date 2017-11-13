#演習1.9  実験レポート : 【VRAM直接書き込み】

-実験目的-

本実験の目的は、パーティションを用いて、MBRの容量である512[byte]を遥かに超えた"VRAM書き換えプログラム"の動作を確認することである。
これに加え、背景色指定や使用文字色(カラーパレット)の指定、読み込みセクター指定等のBIOS割り込みルーチン機能の動作についても確認する。



-実験方法-

  【1. パーティションの使用】
MBR(マスターブートレコード)とは、ブートストラップローダ(446[byte])、パーティションテーブル(64[byte])、ブートシグニチャ([2byte])の
3つから構成されるものであり、ハードディスク等の補助記憶装置の先頭部分に書き込まれている[1]。
通常、コンピュータを起動した際には、BIOSの起動した後、MBRが最初に読み出される。
パーティションとは、「仕切り」という意味を持ち、コンピュータでは補助記憶装置の記憶領域の「ある区切られた領域」のことを指す[2]。

MBRの1つの要素であるパーティションテーブルには、MBR実行後にパーティションの読み込みを行うかどうか等の情報を持っている。
1つのパーティションに関する情報は16[byte]であるため、MBRには全てで4つのパーティションに関する情報を持っている。
今回の実験では1つだけパーティションを作成し、そこにVRAMを書き換えるプログラムを記述した。

パーティションテーブルには、
・ パーティションを使用するかどうかのフラグである『ブートフラグ』  (1[byte])
・ パーティションの開始位置をハードディスクのシリンダー、ヘッド、セクタで表す(CHS方式)、『開始位置指定』  (3[byte])
・ 『パーティションのタイプ』(1[byte])
・ パーティションの終了位置をCHS方式で表す、『終了位置指定』  (3[byte])
・ パーティションの開始位置を各セクタに割り振られた番号で指定する(LBA方式)、『開始位置指定』  (4[byte])
・ LBA方式で表される『総セクター数』 (4[byte])
の情報を持っている。
使用するパーティションテーブルには適当な値を挿入し、使用しない残りのパーティションテーブルは全てのビットを0で埋めた。
今回用いたパーティションの大きさは 512*32[byte] (=16[KB]) に設定し、十分な領域を確保した。

以下の図は、今回の実験で使用する記憶領域を簡潔に視覚化したものである。

  【2. VRAMの書き換え】
VRAMに描画する方法は基本的に、対応する番地に"色"と"文字"を代入する方法を取った。しかし、各番地に即値を代入してしまうと「キャラクタを少し左に動かしたい」等といった微調整に時間を要するため、ある程度の大きさの部品ごとに基準点を設け、その基準点からの距離で描画内容を指定することによりメンテナンス性を向上させた。
ところが、背景のように"同じ色、文字"が複数続く場合には上記のような表記方法は非常に非効率である。そのため、”色、文字、連続する個数”を指定することで自動的に描画する背景用のサブルーチンを作成した。

  【3. BIOS割り込みルーチン】
BIOS割り込みルーチンとは、コンピュータ本体に組み込まれているプログラムである。講義中で扱った"INT"命令はこれを用いて呼び出したものである。
今回の実験プログラムで用いた、ビデオ処理関係を行う"INT 10H"はAHレジスタとBHレジスタをパラメーターとして持つ。具体例を挙げると、AHレジスタの値が0x0b、BHレジスタの値が0x00である時にINT 0x10を呼び出した場合にはBLレジスタで指定された色で背景/罫線に色が付く[3]。
他には、補助記憶装置からセクターを読み込む"INT 13H"も使用した。これは、AH、AL、CH、CL、DH、DL、ES、BXの計8個のレジスタがパラメータとなっており、各々に適切な値を代入し使用した。
  
-実験結果-
以下の図は、実行結果のスクリーンショット画像である。

-考察-
実験結果から、512バイトを遥かに超えるプログラムであってもパーティションを用いることでVRAMを書き換えるができる。また、BIOS割り込みルーチンによる機能も実装することができる。

  
-参考文献-
[1] Incept Inc., "MBR（マスターブートレコード）とは - IT用語辞典", "http://e-words.jp/w/MBR.html", 2017年11月13日.
[2] NTT Reasonant Inc., "partition（パーティション）の意味", "https://dictionary.goo.ne.jp/jn/173026/meaning/m0u/", 2017年11月13日.
[3] Wikipedia, "INT 10H - Wikipedia", "https://en.wikipedia.org/wiki/INT_10H", 2017年11月13日.
