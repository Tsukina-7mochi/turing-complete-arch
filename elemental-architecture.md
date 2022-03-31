# Elemental Architecture

## 命令形式

| タイプ | 1byte | 1byte | 1byte | 1byte |
| - | - | - | - | - |
| R | opcode | rs | rt | rd |
| I | opcode | rs | Immediate | rd |
| J | opcode | Unused | Unused | Address |

## 命令セット

| 種類 | 名称 | 略号 | 意味 | 形式 | オペコード |
| -------- | - | - | - | - | - |
| 論理演算 | Or | `or` | $rd = $rs \| $rt | R | 0x00 |
| 論理演算 | Or Immediate | `ori` | $rd = $rs \| Imm | I | 0x01 |
| 論理演算 | And | `and` | $rd = $rs & $rt | R | 0x02 |
| 論理演算 | And Immediate | `andi` | $rd = $rs & Imm | I | 0x03 |
| 論理演算 | Xor | `xor` | $rd = $rs ^ $rt | R | 0x04 |
| 論理演算 | Xor Immediate | `xori` | $rd = $rs ^ Imm | I | 0x05 |
| 論理演算 | Not Or | `nor` | $rd = $rs \| $rt | R | 0x08 |
| 論理演算 | Not Or Immediate | `nori` | $rd = $rs \| Imm | I | 0x09 |
| 論理演算 | Not And | `nand` | $rd = $rs & $rt | R | 0x0A |
| 論理演算 | Not And Immediate | `nandi` | $rd = $rs & Imm | I | 0x0B |
| 論理演算 | Not Xor | `xnor` | $rd = $rs ^ $rt | R | 0x0C |
| 論理演算 | Not Xor Immediate | `xnori` | $rd = $rs ^ Imm | I | 0x0D |
| 算術演算 | Add | 'add' | $rd = $rs + $rt | R | 0x10 |
| 算術演算 | Add Immediate | 'addi' | $rd = $rs + Imm | I | 0x11 |
| 算術演算 | Subtract | 'sub' | $rd = $rs - $rt | R | 0x12 |
| 算術演算 | Subtract Immediate | 'subi' | $rd = $rs - Imm | I | 0x13 |
| 算術演算 | Set Less That Unsigned | 'sltu' | $rd = ($rs < $rt ? 1 : 0) | R | 0x14|
| 算術演算 | Set Less Than Immediate Unsigned | 'sltiu' | $rd = ($rs < Imm ? 1 : 0) | I | 0x15 |
| 算術演算 | Set Less That | 'slt' | $rd = ($rs < $rt ? 1 : 0) | R | 0x16|
| 算術演算 | Set Less Than Immediate | 'slti' | $rd = ($rs < Imm ? 1 : 0) | I | 0x17 |
| 算術演算 | Divide | 'div' | $rd = $rs / $rt | R | 0x18 |
| 算術演算 | Modulo | 'mod' | $rd = $rs % $rt | R | 0x19 |
| 算術演算 | Divide Unsigned | 'divu' | $rd = $rs / $rt | R | 0x1A |
| 算術演算 | Modulo Unsigned | 'modu' | $rd = $rs % $rt | R | 0x1B |
| 算術演算 | Multiply | 'mul' | $rd = $rs * $rt | R | 0x1C |
| 算術演算 | Multiply Upper Byte | 'mulu' | $rd = ($rs * $rt) >> 8 | R | 0x1D |
| 算術演算 | Shift Left Logical | 'sll' | $rd = $rs << $rt | R | 0x1E |
| 算術演算 | Shift Right Logical | 'srl' | $rd = $rs >> $rt | R | 0x1F |
| 分岐命令 | Branch On Equal | 'beq' | if($rs == $rt) PC = BranchAddr | R | 0x20 |
| 分岐命令 | Branch On Not Equal | 'bne' | if($rs != $rt) PC = BranchAddr | R | 0x21 |
| 分岐命令 | Jump | 'j' | PC = BranchAddr | J | 0x22 |
| 分岐命令 | Jump And Link | 'jal' | R[15] = PC; PC = BranchAddr | J | 0x23 |
| 分岐命令 | Jump Register | 'jr' | PC = $rs | R | 0x24 |
|   入出力 | Load External | 'lx' | $rd = Ext[Imm][$rs] | I | 0x40 |
|   入出力 | Store External | 'sx' | Ext[Imm][$rs] = $rd | I | 0x41 |

- ALU内部の設計がしやすい番号割り当てになっています
- RAM, Input, Output, スタック領域, その他入出力装置へのアクセスを統一的に `lx`, `sx` 命令で行います。

## レジスタ

| 名前 | 番号 | 用途 | 呼び出し間で保存? |
| - | - | - | - |
| $zero | 0 | 定数の0 | - |
| $v0-$v5 | 1-6 | 引数・戻り値 | No |
| $t0-$t3 | 7-10 | 一時変数 | No |
| $s0-$s3 | 11-14 | 一時変数(保存) | Yes |
| $ra | 15 | 戻りアドレス | Yes |

- スタック領域はRAMと物理的に分離されているという前提のため、スタックポインタ及びフレームポインタはありません。

```assembly
const zero 0
const v0 1
const v1 2
const v2 3
const v3 4
const v4 5
const v5 6
const t0 7
const t1 8
const t2 9
const t3 10
const s0 11
const s1 12
const s2 13
const s3 14
const rp 15
```

## 手続き呼び出しの例(スタックが外部装置の2番に設定されている場合)

```assembly
SX zero 2 s0
SX zero 2 s1
SX zero 2 s2
SX zero 2 s3
SX zero 2 rp
# ...処理
LX zero 2 rp
LX zero 2 s0
LX zero 2 s1
LX zero 2 s2
LX zero 2 s3
JR rp _ _
```

## 改善点

- Jump関係の番号割当を工夫してレジスタ参照するかどうか、即値を使うかどうか、 `$rp` に現在のPCの値を格納するかを判別しやすくした方がいい
- 命令長が長い
  - プログラムメモリが少ないのでプログラムサイズが大きくなるのはきつい
  - アセンブラが貧弱なのでレジスタ番号を同じバイトに詰め込むのが難しい
  - 同様の理由でアセンブラ用の変数も用意していないし、命令を少し多めにしている
- `slli`, `srli` がほしい
  - 内部設計を簡単にするために削ってしまった
- `eval` に相当する命令が実行できない
  - ファイルから読み込んで実行ができない
- `lx`, `sx` を拡張した命令が欲しい
  - 複数の外部装置(ex. 複数のRAM) に対してシーケンシャルアクセスしたい
  - FPUみたいな外部演算装置を扱えるように `lx` と `sx` をあわせたものがほしい
  - 外部装置の番号の設定を命令に含めず、特殊レジスタで管理したほうがいいかも、頻繁に切り替えるものでもないし