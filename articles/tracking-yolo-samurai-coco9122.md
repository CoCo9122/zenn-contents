---
title: "最新のTracking技術を比較！YOLO-Tracking と SAMURAI の違い"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python"]
published: true
publication_name: "polarisai_blog"
---

# はじめに

Polaris.AIにて携わっているプロジェクトにおいて、Tracking技術について調査を実施しましたので、今回はその”学び”について説明したいと思います。

## Tracking技術の重要性

Tracking技術とは、コンピュータビジョンのタスクの一つで、動画や画像の中から特定の物体を追跡する技術になります。

他のタスクである物体検出（Object Detection）との違いは、静止画像に対して物体を検出するのに対し、Tracking技術は動画や連続した画像に対して物体を追跡するという点になり、他のタスクに比べて難易度が高いとされています。

実際に使用されている例としては、セキュリティカメラの映像から特定の人物を追跡する、自動運転車のカメラ映像から他の車を追跡する、などが挙げられます。

Tracking技術は、今後のデータ収集や分析においても重要な技術になっていくと考えられます。

## 今回比較する2つの技術の概要

その中でも、今回は以下の2つのTracking技術を比較しました。比較的に簡単に実装できるYoLo-Trackingと、最新の技術であるSAMURAIを今回の対象にしました。

### YOLO-Tracking

YOLO-Trackingは、ultralyticsが提供する効率的なマルチオブジェクトトラッキングソリューションです。YOLOの物体検出能力をベースにした追跡技術で、現在ではYOLOv11が最新バージョンとして提供されています。

以下が公式サイトになります
https://docs.ultralytics.com/ja/modes/track/

:::: details そもそもYOLOって何？

### YOLO（You Only Look Once）とは

YOLO（You Only Look Once）は、物体検出のためのニューラルネットワークになっています。YOLOは、画像全体を一度だけスキャンして、物体の境界ボックスとクラス確率を直接出力します。

従来の物体検出手法が複数の処理ステップを必要としていたのに対し、YOLOは単一のニューラルネットワークで物体検出を完結させる画期的なアプローチです。この「一度だけ見る」という特性により、リアルタイム処理が可能になり、高速かつ効率的な物体検出が実現しました。

### YOLOの主な特徴:

- 高速処理（リアルタイム物体検出が可能）
- 画像全体のコンテキストを考慮できる
- 物体の一般化された表現を学習する

現在ではultralyticsによってYOLOv11までバージョンアップしており、検出精度と処理速度の両面で進化を続けています。

以下がYOLOの論文になります： https://arxiv.org/abs/1506.02640

::::

### YOLO-Trackingの主な特徴
- バウンディングボックスベース追跡：対象物体を矩形の境界ボックスで表現し、フレーム間で追跡します。
- 高速処理：YOLOの高速物体検出能力を活かし、リアルタイムで複数オブジェクトの追跡が可能です。
- 事前訓練済みモデル：COCOデータセットなど大規模データで事前訓練されたモデルを使用できるため、すぐに高性能な追跡を実現できます。
- 複数トラッカー対応：BoT-SORT、ByteTrackなど複数のトラッキングアルゴリズムを選択できます。

### アーキテクチャ
YOLO-Trackingは主に以下のコンポーネントで構成されています：

- YOLO検出器：各フレームで物体検出を行います
- アソシエーションモジュール：フレーム間で検出された物体の対応付けを行います
- ID管理：追跡対象に一貫したIDを割り当てます

このアーキテクチャにより、動画内の複数物体をリアルタイムで効率的に追跡できます。

### SAMURAI

SAMURAIは、SAM（Segment Anything Model）を活用した最新のトラッキングモデルです。

SAMURAI（Segment Anything Model for Universal Real-time Adaptive Instance tracking）は従来のトラッキングモデルとは異なるアプローチを採用しています。特に注目すべき点は、事前学習されたセグメンテーションモデル（SAM）を活用して、あらゆる物体カテゴリに対応できる汎用性の高いトラッキング能力です。これは、Zero-shotトラッキングとしても知られています。

以下が公式リポジトリになります： https://github.com/yangchris11/samurai

:::: details SAMって何？

### SAM（Segment Anything Model）とは

SAM（Segment Anything Model）は、Meta AIが開発した画期的なセグメンテーションモデルです。SAMは「プロンプト」と呼ばれる入力（点、ボックス、テキストなど）に基づいて画像内の任意のオブジェクトをセグメント化できます。

### SAMの主な特徴

- 大規模なデータセット（1,100万枚の画像、11億個のマスク）で訓練されている
- Zero-shotで新しいオブジェクトにも対応できる高い汎用性
- プロンプトベースの柔軟なインターフェース
SAMは単独ではトラッキング機能を持っていませんが、その高精度なセグメンテーション能力がSAMURAIの基盤となっています。

::::

### SAMURAIの主な特徴
- Zero-shot能力：事前学習によって獲得した知識を活用し、トレーニングデータにない新しいカテゴリの物体も追跡できます。
- 適応型トラッキング：動画内で対象物の外観が変化しても追跡を維持するためのテンプレート更新メカニズムを備えています。

### アーキテクチャ
SAMURAIは主に以下のコンポーネントで構成されています：

- SAMセグメンテーション：ベースとなるSAMモデルを使用して高精度なマスク生成を行います
- テンプレート管理機構：対象物の外観テンプレートを動的に更新します
- トラッキングモジュール：フレーム間でオブジェクトの連続性を維持します

このアーキテクチャにより、SAMURAIは物体の形状変化や部分的なオクルージョン（隠れ）が発生する複雑なシーンでも高いパフォーマンスを発揮します。

### 従来技術との違い
従来の多くのトラッキングモデルは特定の物体カテゴリに特化していたり、バウンディングボックスベースの粗い追跡を行うのに対し、SAMURAIはより汎用的かつ高精度なピクセルレベルの追跡を実現しています。

特にYOLO-Trackingと比較すると、SAMURAIは以下のような特徴があります

- より詳細なセグメンテーションマスクを提供
- カテゴリに依存しないZero-shot能力が高い
- 複雑な形状変化や部分的な隠れに強い

論文：SAMURAI: Shape-Aware Multi-Object Tracking with Pixel-Wise Segmentation Masks
https://arxiv.org/abs/2411.11922

# 各種モデルの実装方法

Pythonを使用して実装します。

今回は私はWindowsで実装しました。以下がWindows環境になります

| 項目 | 仕様　|
| ---- | ---- |
|Python| 3.12.0 |
|OS|	Windows10 home
|CPU|	Intel core i7-10700
|GPU|	NAVDIA GeForce RTX 4070

今回使用する動画は以下の動画になります。[pixabay](https://pixabay.com/ja/videos/%E4%BA%BA-%E5%95%86%E6%A5%AD-%E5%BA%97-%E5%BF%99%E3%81%97%E3%81%84-%E3%83%A2%E3%83%BC%E3%83%AB-6387/)と[pixabay](https://pixabay.com/ja/videos/%E3%82%AD%E3%83%A3%E3%83%8B%E3%82%AF%E3%83%AD%E3%82%B9-%E7%8A%AC-%E8%B5%B0%E3%82%8B-216020/)からダウンロードしたものになります。

![](/images/tracking-yolo-samurai-coco9122/img-0001.gif)
*1.1 サンプル画像1*

![](/images/tracking-yolo-samurai-coco9122/img-0004.gif)
*1.2 サンプル画像2*

:::message alert
Zennには動画の貼り付けはできないため、gif画像で代用しています。カクカクなのはご容赦ください
:::

## YOLO-Tracking

YOLO-Trackingの実装方法は以下の通りです。

```sh
# YOLO-Trackingのインストール
pip install ultralytics
```

次に使いやすくなるようにyolo-trackingのモデルの実装します。
```python:models/yolo-tracking.py
import os

from ultralytics import YOLO, settings

class YoloTracking:
    def __init__(self, model="yolo11l.pt"):
        self.directory_name = os.path.dirname(__file__)
        self.model_path = self.directory_name + "/model/" + model

        self.__set_config()
        self.classes = [0]

        print(f"Loading YOLO model from {self.model_path}")

        self.model = YOLO(self.model_path)

    def __name__(self):
        return "YoloModel"
    
    def __set_config(self,):
        settings.update({"sync": False})
    
    def track_video(self, video_path, stream=True, show=True):
        return self.model.track(video_path, show=show, stream=stream)
    
    def track_img(self, video_path, persist=True):
        return self.model.track(video_path, persist=persist, classes=self.classes)
```

実際に動画をトラッキングするは以下のコードになります。
今回はGPUを使用しているため、GPUの確認を行います。それと時間計測を行うコードが入っています。
```python:main.py
import torch
import time
import cv2
import numpy as np
from collections import defaultdict

from models import YoloTracking

# GPUの確認
print("GPU Available: ", torch.cuda.is_available())
if torch.cuda.is_available():
    print("Device Count: ", torch.cuda.device_count())
    print("Device Name: ", torch.cuda.get_device_name(0))
    torch.cuda.set_device(0)
    print("Current Device: ", torch.cuda.current_device())

def yolo_tracking():
    
    yolo = YoloTracking()
    
    video_path = "walking_persons.mp4"
    track_history = defaultdict(lambda: [])

    # cv2で動画を読み込むための設定
    cap = cv2.VideoCapture(video_path)
    if not cap.isOpened():
        print("Error: Video not found")
        return
    

    # cv2で動画を書き込むための設定
    fourcc = cv2.VideoWriter_fourcc(*'mp4v')
    out = cv2.VideoWriter('output.mp4', 
        fourcc, 
        cap.get(cv2.CAP_PROP_FPS), 
        (int(cap.get(cv2.CAP_PROP_FRAME_WIDTH)), 
        int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT)))
    )

    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break

        results = yolo.track_img(frame)
        boxes = results[0].boxes.xywh.cpu()
        track_ids = results[0].boxes.id.int().cpu().tolist()
        annotated_frame = results[0].plot()

        for box, track_id in zip(boxes, track_ids):
            x, y, w, h = box
            track = track_history[track_id]
            track.append((float(x), float(y)))  
            if len(track) > 30:  
                track.pop(0)

            points = np.hstack(track).astype(np.int32).reshape((-1, 1, 2))
            cv2.polylines(
                annotated_frame,
                [points],
                isClosed=False,
                color=(230, 230, 230),
                thickness=10,
            )
        
        out.write(annotated_frame)

    cap.release()
    out.release()

if __name__ == "__main__":

    start_time = time.time()
    yolo_tracking()
    print("Time taken: ", time.time() - start_time)
```

実際に動画をトラッキングした結果が以下になります。
![](/images/tracking-yolo-samurai-coco9122/img-0002.gif)
*1.3 YOLO-Trackingの結果1*

![](/images/tracking-yolo-samurai-coco9122/img-0005.gif)
*1.4 YOLO-Trackingの結果2*

一つ目の結果からマルチオブジェクトトラッキングができていることが確認できます。ただ、一部のオブジェクトがトラッキングされていないですね。

二つ目の結果から、同様にトラッキングできていますが、木に隠れた場合などはトラッキングに外れてしまうことが確認できます。画像では見にくいですが、木にかぶさった場合の番号は再連番されていますね。それに犬も人と検知していますね。
YOLO-TrackingにおいてRe Identificationは行われていないため、このような結果になります。

推論速度は、一つ目は約24秒、二つ目は約78秒となっております。

:::message alert
速度は、GPUやCPUの性能によって変わりますので参考程度に
:::


## SAMURAI

次にSAMURAIの実装方法は以下の通りです。
公式のリポジトリにあるREADMEに従って実装します。

https://github.com/yangchris11/samurai/blob/master/README.md

```sh
# リポジトリのクローン
git clone git@github.com:yangchris11/samurai.git

# ディレクトリの移動
cd samurai

# ライブラリのインストール
cd sam2
pip install -e .
pip install -e ".[notebooks]"

pip install matplotlib tikzplotlib jpeg4py opencv-python lmdb pandas scipy loguru
```

URLにアクセスして[ptファイル](https://dl.fbaipublicfiles.com/segment_anything_2/092824/sam2.1_hiera_base_plus.pt)をダウンロードします。

ダウンロードしたファイル`sam2.1_hiera_base_plus.pt`を`samurai/sam2/checkpoints`に移動します。

:::message alert
本来であれば、download_ckpts.shを実行してモデルをダウンロードできますが、Windows環境では実行できないため、手動でダウンロードしました。
:::

次に初期プロンプトを設定します。SAMURAINIはSAM2同様に初期プロンプトを設定するためのファイルが必要です。
初期プロンプトにはtxtファイルを使用し、以下の内容で記述します。x,y,w,hはトラッキングしたい物体の座標を表しています。
```txt: first_prompt_bbox.txt
# x,y,w,h
```
このことからSAMURAIだけではトラッキングはできないため、YOLOで物体検出を行い、その結果を初期プロンプトを作成し、SAMURAIに渡す形で実装します。

今回は`scripts/demo.py`を修正する形で実装します。
YOLOで物体検出を行い、その結果を初期プロンプトに書き込むコードの追加と時間計測を行うコードを追加します。

```python:scripts/demo.py
def create_init_prompt(args):
    frames_or_path = prepare_frames_or_path(args.video_path)

    yolo = YoloTracking()

    cap = cv2.VideoCapture(frames_or_path)
    if not cap.isOpened():
        print("Error: Video not found")
        return
    
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break

        results = yolo.predict(frame)
        boxes = results[0].boxes.xywh.cpu()
        clses = results[0].boxes.cls.cpu()

        for box, cls in zip(boxes, clses):
            if cls == 0:
                x_center, y_center, w, h = box
                x = x_center - w // 2
                y = y_center - h // 2
                with open("first_prompt_bbox.txt", "a") as f:
                    f.write(f"{x},{y},{w},{h}\n")
        break


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--video_path", required=True, help="Input video path or directory of frames.")
    parser.add_argument("--txt_path", required=True, help="Path to ground truth text file.")
    parser.add_argument("--model_path", default="sam2/checkpoints/sam2.1_hiera_base_plus.pt", help="Path to the model checkpoint.")
    parser.add_argument("--video_output_path", default="demo.mp4", help="Path to save the output video.")
    parser.add_argument("--save_to_video", default=True, help="Save results to a video.")
    args = parser.parse_args()
    start_time = time.time()
    create_init_prompt(args)
    main(args)
    print("Time taken: ", time.time() - start_time)
```

実際に動画をトラッキングした結果が以下になります。
![](/images/tracking-yolo-samurai-coco9122/img-0003.gif)
*1.5 SAMURAIの結果1*

![](/images/tracking-yolo-samurai-coco9122/img-0006.gif)
*1.6 SAMURAIの結果2*

一つ目の結果から、問題なくトラッキングができていることが確認できます。
二つ目の結果から、同様にトラッキングができていることが確認できます。またRe Identificationも行われていますね。

推論速度は、一つ目は約35秒、二つ目は約117秒でした。

:::message alert
同様に速度は、GPUやCPUの性能によって変わりますので参考程度に
:::

# 性能比較

YOLO-TrackingとSAMURAIの性能を比較します。

## 速度

以下にYOLO-TrackingとSAMURAIの速度を比較した結果を示します。
| モデル |　動画1 | 動画2 |
| ---- | ---- | ---- |
| YOLO-Tracking | 24秒 | 78秒 |
| SAMURAI | 35秒 | 117秒 |

シンプルなモデルであるYOLO-Trackingの方が速度が速いことが確認できますね。

## 精度

今回の結果だけでは一概には言えませんが、SAMURAIの方が精度が高いと感じました。
ただ、処理時間のわりにYOLO-Trackingの精度も高いので、プロジェクトの要件や目的によって変わるのかなといった印象です。

## 使いやすさ

実際にそれぞれのモデルを実装してみて、圧倒時にYOLO-Trackingの方が使いやすいと感じました。
pip installだけ簡潔し、ドキュメントも豊富にあるので、すぐに実装できることができました。
一方、SAMURAIはリポジトリをクローンして、モデルをダウンロードして、初期プロンプトを設定する必要があるため、少し手間がかかります。

# どちらを使うべきか

YOLO-TrackingとSAMURAIのどちらを使うべきかについてですが、。一概には言えませんが、以下のような特徴を加味して判断する必要がありそうですね。

- YOLO-Tracking
  - 高速処理
  - 事前訓練済みモデル
  - 複数トラッカー対応
- SAMURAI
  - Zero-shot能力
  - 適応型トラッキング
  - Re Identification
  
したがって、「どちらを使うべきか」は、プロジェクトの要件や目的によって変える必要があると思います。

# Zero shotを人以外にもやってみる

せっかくなのでZero shotを人以外にもやってみます。
今回は馬をトラッキングしてみます。これも[pixabay](https://pixabay.com/ja/videos/%E9%A6%AC-%E5%8B%95%E7%89%A9-%E8%B5%B0%E3%82%8B-%E9%87%8E%E7%94%9F%E5%8B%95%E7%89%A9-206140/)からダウンロードしたものになります。

![](/images/tracking-yolo-samurai-coco9122/img-0007.gif)
*1.7 馬の動画*

同様にSAMURAIでトラッキングしてみます。
![](/images/tracking-yolo-samurai-coco9122/img-0008.gif)
*1.8 馬の結果*

さすがにSAMURAIでも難しいですね。画面外に出ても、途中までトラッキングが続いていることが確認できますが、別の馬の個体が画面内に入ってくると間違ってトラッキングしていますね。

# まとめ

今回はYOLO-TrackingとSAMURAIの性能を比較しました。
YOLO-Trackingは高速処理が可能で、事前訓練済みモデルを使用できるため、簡単にトラッキングを実装できます。
一方、SAMURAIはZero-shot能力が高く、適応型トラッキングが可能で、Re Identificationも行われています。

プロジェクトの要件や目的によって、どちらを使うべきかは異なりますが、今後のデータ収集や分析においても、Tracking技術は重要な技術になっていくと考えられます。