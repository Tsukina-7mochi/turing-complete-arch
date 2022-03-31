# Turing Complete Architectures

## Abstract

[Turing Complete](https://turingcomplete.game)のために設計したアーキテクチャのメモです。

## Elemental Architecture

8bitベースのアーキテクチャです。レジスタサイズ・アドレスバスが8bit、命令長さが32bitです。
標準の4byte outputのRAMで利用しやすいように設計されています。

### 特徴

- MIPSに影響を受けた設計
- ロード・ストアアーキテクチャ
- 16個のレジスタ
- 一般化された外部入出力命令

