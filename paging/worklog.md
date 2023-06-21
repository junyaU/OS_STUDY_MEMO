# 作業録

## CallAppを実行するとOSが再起動する （解決済み）
mikanosのosbook_day20bにおいてfar returnで、OSからアプリを実行すると、CPU例外が発生しOSが再起動してしまう。

```
RAX=0000000000fead60 RBX=ffffffff00000000 RCX=0000000000000023 RDX=000000000000001b
RSI=fffffffffffff000 RDI=0000000000000004 RBP=ffffffffffffeff0 RSP=ffffffffffffeff0
R8 =ffff800000001060 R9 =ffffffffffffeff8 R10=0000000000000001 R11=0000000000001000
R12=0000000000fee05e R13=0000009900000000 R14=0000000000000000 R15=00000000010ddd70
RIP=ffff800000001064 RFL=00000246 [---Z-P-] CPL=3 II=0 A20=1 SMM=0 HLT=0
```
アプリ実行時のCPLはちゃんと3になっており、RSPやRIPもアプリの仮想アドレスになっている。


```
0x10cec4 <CallApp>:	push   rbp
0x10cec5 <CallApp+1>:	mov    rbp,rsp
0x10cec8 <CallApp+4>:	push   rcx
0x10cec9 <CallApp+5>:	push   r9
0x10cecb <CallApp+7>:	push   rdx
0x10cecc <CallApp+8>:	push   r8
0x10cece <CallApp+10>:	retfq
```
とりあえず、GDBでブレークポイントを設定して、CallApp付近の命令を逆アセンブルしてみた。\
RIP、CS、RSP、SSの値をスタックに保存してfar returnの実行はできている。


```
0xffff800000001060:	push   rbp
0xffff800000001061:	mov    rbp,rsp
0xffff800000001064:	add    BYTE PTR [rax],al
→ IntHandlerPF (frame=0xfffffffffffff104, error_code=18446744073709547776)
```
far returnにより、アプリの仮想アドレスに飛ぶところまではきている。\
ファンクションプロローグは正しく実行されているが、その次の命令でページフォルトが発生している模様。\
add BYTE PTR [rax], al　は、raxが指すアドレスにalを足す命令なので、raxに格納されているアドレスがおかしい？

この時点でのRAXの値は0000000000fead60となっていた。\
このアドレスはカーネル空間のアドレスなので、CPL=3の状態のアプリからアクセスできないから？ \
far return 前（カーネルモードの時）にRAXに格納されていたアドレスが、far return 後（ユーザモードの時）にもそのまま残っているからだめなのかも \
取りあえずユーザーモードになる前に、RAXに対して仮想アドレスを設定してみてからfar returnしてみる。
```
CallApp: 
    push rbp
    mov rbp, rsp
    mov rax ,r9
    push rcx ; SS
    add r9, -8
    push r9   ; RSP
    push rdx ; CS
    push r8  ; RIP
    o64 retf
```
一旦、RSPから8足したアドレスを設定してみる。\
これによって、それぞれのレジスタの値は以下のように設定されたので、さきほどの部分でのページフォールとは回避できるはず。
- RSP: 0xffffffffffffeff0
- RAX: 0xffffffffffffeff8


RAXの値変更後の実行結果
```
0xffff800000001060:	push   rbp
0xffff800000001061:	mov    rbp,rsp
0xffff800000001064:	add    BYTE PTR [rax],al
0xffff800000001066:	add    BYTE PTR [rax],al
0xffff800000001068:	add    BYTE PTR [rax],al
                     ・
                     ・
                     ・
0xffff800000002ffe:	add    BYTE PTR [rax],al
0xffff800000003000:	<error: Cannot access memory at address 0xffff800000003000>
IntHandlerPF (frame=0xfffffffffffff100, error_code=144)
```

またしてもページフォールトが発生したが、今度は0xffff800000003000で発生した。　\
おそらくそのページの終端に達して、ページが割り当てられていないアドレスにまで到達したから。\
到達するまでは、add    BYTE PTR [rax],al　が繰り返し実行されている。\
となると、そもそもこの命令が出力されること自体おかしいのではないかと思い、ChatGPTに聞いてみると、

> add BYTE PTR [rax],alという命令が繰り返し出力されているというのは一般的ではありません。そして、特にwhileループのコードから生じるものではありません。
この命令は、RAXレジスタが示すメモリアドレスのバイトにALレジスタの値を加算します。そしてこの命令が連続しているということは、何らかの無限ループやバグを示唆している可能性があります。\
特に、メモリアドレスやレジスタ値が不適切な場合、このような命令が生成される可能性があります。

とのことだったので、なぜこの命令が出力されてしまっているのかを調べていくことにする。

アプリ側のコードとしては、下記の内容になっている
```cpp
extern "C" int main(int argc, char **argv) {
    while (true) {
    };
}
```
ただ単純に無限ループを走らせるだけのこーどである。\
このアプリの実行ファイルを逆アセンブルしてみると、
```
ffff800000001060 <main>:
ffff800000001060:	55                   	push   %rbp
ffff800000001061:	48 89 e5             	mov    %rsp,%rbp
```
こうなる。ただただ、プロローグのみが出力されている。\
そういえば、OSでアプリを実行する時も、プロローグと、add    BYTE PTR [rax],alのみが出力されていた。\
このwhileループは中に何の記述もしていないため、ループが終わらないことを、コンパイラが検知し最適化してしまったのか… \
最適化させないために、-O0を付与してコンパイルしてみる。
```
ffff800000001070 <main>:
ffff800000001070:	55                   	push   %rbp
ffff800000001071:	48 89 e5             	mov    %rsp,%rbp
ffff800000001074:	89 7d fc             	mov    %edi,-0x4(%rbp)
ffff800000001077:	48 89 75 f0          	mov    %rsi,-0x10(%rbp)
ffff80000000107b:	e9 fb ff ff ff       	jmp    ffff80000000107b <main+0xb>
```
ループの部分がちゃんと出力されている！\
これでアプリ実行後もページフォールトが発生せずにループが回るようになった。

