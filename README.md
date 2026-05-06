# write_blog

このリポジトリは、[michikusa-log](https://www.michikusa-log.jp) の技術ブログ記事で使用した検証コードを置くための公開リポジトリです。

記事本文では説明を読みやすくするために結果や考え方を中心にまとめ、ここでは再現用の notebook、入力画像、生成した比較画像、簡易指標のCSVを管理します。

## Contents

| Directory | Description |
|---|---|
| `low_light_icd_comparison_notebook/` | 低照度画像補正とICDの考え方を比較する notebook |

## Low-Light ICD Comparison Notebook

`low_light_icd_comparison_notebook/low_light_icd_comparison_notebook.ipynb` は、記事「暗い画像をただ明るくしない: ICDで色とノイズの扱いはどう変わるのか試す」で使用した実験 notebook です。

この notebook では、同じ低照度画像に対して次の方法を比較します。

- Gamma / Exposure補正
- CLAHE
- 簡易Multi-Scale Retinex
- ICD-inspiredな簡易実装
- 公式ICDNet推論（任意）

`ICD-inspired` は論文モデルの再実装ではなく、Intensity と Chromaticity に分ける考え方を説明するための簡易実装です。公式モデルの推論は、notebook内の任意セルで [mubaisam/ICD](https://github.com/mubaisam/ICD) を使用します。

## Notes

- 公式ICDリポジトリのclone結果、checkpoint、学習済み重みファイルはこのリポジトリには含めません。
- 入力画像は [Pexels Photo 3185041](https://www.pexels.com/photo/black-metal-window-frame-3185041/) を使用しています。利用条件は [Pexels License](https://www.pexels.com/license/) を確認してください。
- 論文は [Rethinking Low-Light Image Enhancement: A Log-Domain Intensity--Chromaticity Decoupling Perspective](https://arxiv.org/abs/2605.02627) を参照しています。
