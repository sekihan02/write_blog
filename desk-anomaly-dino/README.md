# desk-anomaly-dino

DINO系の画像埋め込みを使って、机の上に追加された物体や状態変化を検出するためのNotebook実験です。

やっていることは embedding-based anomaly detection の最小構成です。

1. 正常状態の机画像からパッチ特徴量バンクを作る
2. 検査画像からパッチ特徴量を取り出す
3. 検査画像の各パッチを正常特徴量バンクと比較する
4. 正常画像のどのパッチにも似ていない場所を異常として扱う
5. `heatmap` / `overlay` / `mask` / `summary.csv` を保存する

## ディレクトリ構成

```text
desk-anomaly-dino/
+- notebooks/
|  +- desk_anomaly_dino.ipynb
+- data/
|  +- reference_normal/
|  |  +- ref_001_clean.jpg
|  +- observed/
|     +- obs_001_observed.jpg
+- outputs/
|  +- desk_anomaly_dino/
|     +- heatmap/
|     +- overlay/
|     +- mask/
|     +- annotated/
|     +- comparison/
|     +- components/
|     +- summary.csv
+- cache/
|  +- reference_features/
+- README.md
+- requirements.txt
```

`cache/reference_features/` と `outputs/` の中身は実行時に生成されるため、公開用の管理対象には含めません。

`data/` のサンプル画像は再現用に残せますが、公開前にEXIFなどのメタデータを除去しておく前提です。

## 画像名

正常基準画像は `data/reference_normal/` に入れます。

```text
ref_001_clean.jpg
ref_002_clean_shadow.jpg
ref_003_clean_bright.jpg
```

検査したい画像は `data/observed/` に入れます。

```text
obs_001_observed.jpg
obs_002_bottle.jpg
obs_003_paper.jpg
obs_004_parts.jpg
```

最初に置かれていた画像は、次のようにリネーム済みです。

```text
base.jpg -> data/reference_normal/ref_001_clean.jpg
now.JPG  -> data/observed/obs_001_observed.jpg
```

## Notebook名について

今回はNotebookが1本だけなので、連番の `01_` は外して `notebooks/desk_anomaly_dino.ipynb` にしました。

あとで「撮影整理」「特徴量比較」「カメラ入力」などを別Notebookに分けたくなったら、その時点で `01_`, `02_` のように戻す方が自然です。

## 実行方法

Notebookを開きます。

```text
notebooks/desk_anomaly_dino.ipynb
```

上から順にセルを実行してください。CUDAが使える環境では自動でGPUを使います。

## Hugging Faceのgated repoエラーについて

DINOv3の `facebook/dinov3-vits16-pretrain-lvd1689m` はgated repoです。

ブラウザでHugging Faceにログイン済み、またはモデルページでアクセス権を持っていても、Notebookを実行しているPython環境にトークンが無い場合は401になります。

特にDockerや別のJupyterカーネルで動かしている場合、Windows側やブラウザ側のログイン状態は自動では引き継がれません。

Notebook内にトークン入力セルを入れています。Hugging Face上で作成したトークンの名前、たとえば `hugging_token` は管理用の表示名なので、Notebookには入力しません。

入力するのはトークンの値です。Notebook上では次のような入力欄が出ます。

```text
Hugging Face token（入力は表示されません）:
```

入力した値は、このNotebookカーネル内の `HF_TOKEN` として使います。Notebookファイルには保存しません。

毎回入力したくない場合は、同じPython環境のターミナルでログインしておく方法もあります。

```bash
huggingface-cli login
```

細かく管理するなら、環境変数で渡す方法もあります。

```bash
HF_TOKEN=your_token
```

トークンはHugging FaceのRead権限を持ち、対象のgated modelにアクセスできるアカウントのものを使ってください。

## 出力

成功すると次に保存されます。

```text
outputs/desk_anomaly_dino/heatmap/
outputs/desk_anomaly_dino/overlay/
outputs/desk_anomaly_dino/mask/
outputs/desk_anomaly_dino/annotated/
outputs/desk_anomaly_dino/comparison/
outputs/desk_anomaly_dino/components/
outputs/desk_anomaly_dino/summary.csv
```

この本線出力はDINOv3用です。

`annotated/` には異常候補のbboxとラベル付き画像を保存します。
`components/` には画像ごとの異常候補一覧をCSVで保存します。

まず見るべきなのは `overlay/` です。ロボット連携や座標化を考える場合は、`mask/` と `summary.csv` の `center_x`, `center_y`, `area_ratio` を見ます。

Notebook上では、最初に `reference_normal` と `observed` の入力画像を横並びで確認します。

1枚試すセルでは、まず左から次の順に横並びで表示します。

```text
Reference normal | Observed | Anomaly heatmap | Overlay | Annotated | Mask
```

この並びで、正常状態と現在画像の違い、DINO埋め込みが異常として見ている場所、bbox付きの異常候補、しきい値後のマスクを同時に確認できます。

同じ横並び画像は `outputs/desk_anomaly_dino/comparison/` にも保存します。

さらにNotebook上では、同じ6種類を1枚ずつ大きく表示します。細部を見たい場合は、横並びではなく個別表示の `Annotated` と `Mask` を確認してください。

## 異常候補の見方

Notebookでは、maskをconnected componentsで分割し、異常候補ごとに次を出します。

```text
decision
rank
center_x, center_y
x_min, y_min, x_max, y_max
area
area_ratio
mean_score
max_score
```

`decision=accepted_component` が、現在のしきい値と面積条件を満たして「異常候補として採用された領域」です。Notebookの `Annotated` 画像では赤いbboxで表示します。

候補が0件の場合でも、`decision=top_score_point_not_component` としてraw scoreが高い地点を表示します。これは「異常として採用された領域」ではなく、「今の設定では採用されなかったが、モデルが最も怪しいと見ている地点」です。Annotated画像では青丸で表示します。

`rank=1` が最も強い候補または最大スコア地点です。ロボットやカメラ制御に使う場合は、まず `decision=accepted_component` の `rank=1` を使ってください。

`summary.csv` には各画像の最大候補を `top_*` 列として入れています。全候補は `components/` のCSVを見てください。

## スコアについて

論文寄りに、定量値とmaskにはraw anomaly scoreを使います。

```text
raw anomaly score = (1 - cosine_similarity) / 2
```

`heatmap` には見やすさのため、画像内でrobust normalizeした表示用スコアを使います。

つまり、

```text
anomaly_raw     = mask / summary / components 用
anomaly_visual  = heatmap / overlay 用
```

です。

画像端やキーボード境界の反応を抑えるため、初期状態ではROIを有効にしています。Notebookの設定セルで調整できます。

## 論文そのものではない追加処理

Notebookの中心部分は、DINOv3特徴、single reference、patch token、L2正規化、最近傍cos類似度、anomaly score、upsampling、threshold maskです。

一方で、机上実験用に次の後処理を追加しています。これらは論文そのままの最小手法ではありません。

```text
ROI処理
connected componentsによる候補分割
bbox / center / area / max_score の出力
今回の画像セット向けのthreshold調整
```

ROIはキーボードや画像端の誤反応を抑えるための実験用設定です。

connected componentsは、ロボットやカメラ制御で「どこを見るべきか」を扱いやすくするための追加処理です。

また、`max_score = 0.2139` のような値は確率ではありません。21.39%の確率で異常という意味ではなく、DINOv3特徴空間で正常referenceのpatchに似ていない度合いです。

初期値では次の設定です。

```python
ANOMALY_SCORE_THRESHOLD = 0.10
MIN_COMPONENT_AREA = 120
```

候補が0件になる場合は、`ANOMALY_SCORE_THRESHOLD` または `MIN_COMPONENT_AREA` を下げてください。Notebookの「raw scoreしきい値確認」セルでは、各しきい値で候補数も表示します。

## 付録: DINOv2比較

Notebookの最後にDINOv2比較用の付録セルを入れています。

現在は比較しやすいよう、最初から実行する設定にしています。

```python
RUN_DINOV2_APPENDIX = True
```

DINOv2の出力先は本線とは分けています。

```text
outputs/desk_anomaly_dino_appendix_dinov2/
```

以前フォールバックで作られたDINOv2結果も、この付録フォルダへ移動済みです。

DINOv2付録を実行した場合も、先頭のobserved画像についてDINOv3本線と同じ形式で可視化・候補表を表示します。
