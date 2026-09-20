# TEDxJP-5K-C

A Japanese ASR evaluation set for context. 5,000 segments of 2 to 30 seconds, arranged into 698 stretches of consecutive speech.

[日本語はこちら](#日本語)

## Overview

Built from Japanese TEDx talks on YouTube. The audio is the talk itself; the reference text is the human-written Japanese caption track published with it.

Changes from the sources:

- Adjacent caption cues are merged so that segment lengths spread uniformly over 2 to 30 seconds.
- Cuts are placed only where the captions leave a pause, with a random margin of 0 to 200 ms at each end to imitate a different voice-activity detector.
- Caption text is normalised: full-width digits and Latin letters to half-width, non-speech markers such as laughter and applause removed, whitespace stripped.

All of that is identical to [TEDxJP-5K-V](https://github.com/Columba1198/TEDxJP-5K-V). What differs is how the 5,000 segments are chosen.
This dataset ([TEDxJP-5K-C](https://github.com/Columba1198/TEDxJP-5K-C)) groups consecutive speech, so that ASR models able to make use of context can be measured on it.

## Why this exists

Japanese ASR benchmarks that can be obtained without an application process are scarce, and the ones that exist cover a narrow range.

- Common Voice is read speech averaging around four seconds. Long-form behaviour is invisible, and some references do not match their audio.
- TEDxJP-10K has reliable references, but one utterance is one caption cue, so nothing runs longer than about 11 seconds. Its references are close to verbatim: fillers were added by hand and Arabic numerals rewritten as kanji.

TEDxJP-5K-V picks its 5,000 segments spread thinly over the talks, which is what gives it its variety. The cost is that its segments are independent even within one talk: the context between them is not preserved.

That makes it unsuitable for measuring a model that refers to what was said before — Whisper's `--condition-on-previous-text`, a model that takes acoustic features from several segments, a streaming system carrying state between utterances.

This dataset takes a starting point picked at random in each talk and cuts segments consecutively from it, as sequences. The preceding segment is in the dataset, so models of that kind can be measured too.

## The TEDxJP-5K sets

The source talks and the caption format are the same across the three datasets. What changes is the audio and the choice of segments.

| | [TEDxJP-5K-V](https://github.com/Columba1198/TEDxJP-5K-V) | [TEDxJP-5K-N](https://github.com/Columba1198/TEDxJP-5K-N) | TEDxJP-5K-C |
|---|---|---|---|
| Measures | accuracy across varying audio lengths | robustness to noise | use of preceding context |
| Audio | unprocessed (clean) | perturbed (noise added, speed perturbation, etc.) | unprocessed (clean) |
| Choice of segments | spread thinly over the talks | identical to -V | consecutive (698 sequences) |
| Is the preceding segment in the dataset? | rarely | rarely | usually |
| Segment length | uniform 2.0 to 30.0 s, mean 16.0 s | same cuts as -V; 1.4 to 30.0 s, mean 15.1 s after the speed changes | uniform 2.0 to 30.0 s, mean 16.0 s |
| Segments / talks | 5,000 / 257 | 5,000 / 257 | 5,000 / 257 |
| Total audio | 22.2 h | 20.9 h | 22.2 h |
| Reference characters | 409,033 | 409,033 | 410,838 |
| Utterance ids | all 5,000 shared with -N | all 5,000 shared with -V | 67 coincide with -V and -N |

-V and -N cut their segments at the same points, so their scores can be compared directly and the difference between them is a measure of robustness.
-C uses almost entirely different segments, so its score cannot be compared directly against either.

## How the sequences are built

| Property | Setting |
|---|---|
| Segment length | Uniform over 2 to 30 seconds, in 1-second bins |
| Merging | Adjacent caption cues joined; cuts only at pauses of 0.5 s or longer |
| Edge margin | 0 to 200 ms at each end, drawn independently |
| Speakers | Tuned so that a wide range of talks is represented |
| Starting point | Picked at random within the talk |
| Sequence length | 4 segments or more where the material allows, 20 at most; mean 7.2, median 5 |
| Pause a sequence may carry | Up to 5 seconds between one segment and the next |

A sequence ends where the talk stops being continuous:

- The captions leave a gap longer than 5 seconds. A break that long may mean the topic has moved on, or that the captions are missing a stretch of speech.
- A stretch of speech cannot be cut into a segment at all: a single caption longer than 30 seconds, or a short isolated caption with long pauses on both sides. That stretch is left out of the dataset and the next sequence begins after it.
- The sequence reaches 20 segments, so that one talk cannot dominate the dataset.

Talks contribute 19.5 segments on average, and a sequence averages 7.2, so a talk usually holds two or three of them.

## Statistics

| | |
|---|---|
| Segments | 5,000 |
| Sequences | 698 |
| Segments per sequence | 1 to 20, mean 7.2, median 5 |
| Segments with a preceding segment in the dataset | 4,302 (86.0%) |
| Preceding segments available, per segment | mean 5.5, at most 19 |
| Total audio | 22.2 hours |
| Length | 2.0 to 30.0 s, mean 16.0 s |
| Talks | 257 |
| Reference characters | 410,838 |
| Format | FLAC, 16 kHz, mono |

## Using the sequences

`manifest.jsonl` carries three fields the other datasets do not:

| Field | Meaning |
|---|---|
| `seq_id` | The sequence this segment belongs to |
| `seq_index` | Its position in that sequence, from 0 |
| `seq_len` | How many segments the sequence has |

The file is written in sequence order, so reading it top to bottom gives each sequence in the order it was spoken. `seq_index` 0 means there is no history for that segment.

`prev_context.json` holds, for each segment, the captions of up to eight segments that come before it in the same talk, oldest first. It is here to match the other two datasets.
When measuring a feature that conditions on preceding context, such as Whisper's `--condition-on-previous-text`, we recommend feeding it the model's own output rather than `prev_context.json`, because in real use an error in the output can propagate.

## Notes

- The captions favour readability over verbatim accuracy, so fillers are not transcribed. A system that writes out every hesitation is charged for insertions.
- Numbers are written with half-width Arabic digits.
- Punctuation and symbols are kept in the references. Strip them, along with whitespace, from both reference and hypothesis before computing CER.
- Audio is not distributed. TEDx talks are licensed CC BY-NC-ND 4.0, so this repository ships the segmentation and the references. You can rebuild the dataset by running `rebuild.py`.
- If a talk becomes private or is removed, its segments cannot be rebuilt. `rebuild.py` skips them and lists which ones. A sequence that is missing a segment is broken, so drop the whole sequence rather than that segment alone.
- 67 of the 5,000 clips are identical to a TEDxJP-5K-V clip, reference and all. The random edge margins are drawn from the same seed in both datasets, so a segment that happens to cover the same captions in both gets the same cut. Nothing else is shared: the remaining 4,933 are cut differently.
- The source talks are the ones TEDxJP-10K uses, but the segmentation differs, so no clip matches a TEDxJP-10K utterance.

## Layout

```
prev_context.json  up to 8 preceding segments per utterance, read off this dataset's own rows
plan.json          segmentation: source talk, cut range, reference text, sequence
manifest.jsonl     NeMo-style manifest, one line per segment, in sequence order
text               utterance id and reference, Kaldi style
segments utt2spk spk2utt   Kaldi-style metadata
rebuild.py         downloads the talks and regenerates clips/
clips/ source/     produced by rebuild.py, not tracked
```

## Rebuilding

```sh
pip install numpy soundfile "yt-dlp[default]"
python rebuild.py
```

ffmpeg must be on PATH, along with a JavaScript runtime such as Node.js for the YouTube extractor. Keep yt-dlp current: YouTube-side changes break older versions, and a nightly build is occasionally needed before the fix reaches a release.

## Licence

This repository is licensed under Apache-2.0, covering the segmentation, the metadata and the scripts.

The audio produced by `rebuild.py` is not covered. The source talks remain under CC BY-NC-ND 4.0 and belong to their speakers and TEDx organisers.

---

<a name="日本語"></a>

# TEDxJP-5K-C

文脈を測るための日本語ASR評価セット。2〜30秒のセグメント5,000本を、連続する発話のまとまり698本として収録しています。

## 概要

YouTubeで公開されている、日本語のTEDxトークから作成しました。音声はトーク本体、字幕は各トークに付随する手入力の日本語字幕です。

元データからの変更点:

- 隣接する字幕を連結し、セグメント長を2〜30秒に均一分布させています。
- 切れ目は字幕の間が空いている箇所にのみ置き、VADの違いを模して前後に0〜200msのランダムなマージンを付けています。
- 字幕を正規化しています。全角の数字と英字は半角にし、音声に含まれない字幕（『（笑い）』や『（拍手）』など）と空白は除去しています。

ここまでは [TEDxJP-5K-V](https://github.com/Columba1198/TEDxJP-5K-V) と同一です。違うのは5,000本の選び方です。
本セット（ [TEDxJP-5K-C](https://github.com/Columba1198/TEDxJP-5K-C) ）では、連続した発話をグループ化しており、文脈情報を処理できるASRモデルに対応しています。

## 作成理由

申請なしで入手できる日本語ASRベンチマークは少なく、既存のものは測れる範囲が限られています。

- Common Voice は読み上げ音声で平均4秒程度です。長尺での精度を計測できないうえ、音声と一致しない字幕も含まれます。
- TEDxJP-10K は字幕の質が高い一方、1発話が字幕1キューなので最長でも約11秒です。字幕は逐語寄りで、フィラーが手作業で追加され、アラビア数字が漢数字に書き換えられています。

TEDxJP-5K-V は257本のトークから薄く広く5,000本を選んでおり、それが多様性の源になっています。その代償として、トーク内でも各セグメントが独立しており、音声間の文脈が維持されていません。
そのため、直前の発話を参照するモデルの性能測定には向きません。Whisperの `--condition-on-previous-text`、複数セグメントの音声特徴量を使用するモデル、発話間で状態を持ち越すストリーミング認識などです。
このデータセットでは、各トークでランダムに選んだ地点を起点にして、そこから続けてセグメントを切り出したシーケンスを収録しています。直前のセグメントがデータセット内にあるため、前述のモデルの性能も測定できます。

## TEDxJP-5K の3セット

元トークと字幕のフォーマットは3つのデータセットで共通です。違うのは、音声とセグメントの選び方です。

| | [TEDxJP-5K-V](https://github.com/Columba1198/TEDxJP-5K-V) | [TEDxJP-5K-N](https://github.com/Columba1198/TEDxJP-5K-N) | TEDxJP-5K-C |
|---|---|---|---|
| 測るもの | 多様な音声長での精度 | 雑音への耐性 | 直前の文脈の活用 |
| 音声 | 無加工 | ノイズを追加（雑音追加、速度変更など） | 無加工 |
| セグメントの選び方 | トーク全体から薄く広く | -V と同一 | 連続（698シーケンス） |
| 直前のセグメントがデータセット内にあるか | ほぼ無い | ほぼ無い | ほぼ有る |
| セグメント長 | 2.0〜30.0秒を均一分布、平均16.0秒 | 切れ目は -V と同一。速度変更後は1.4〜30.0秒、平均15.1秒 | 2.0〜30.0秒を均一分布、平均16.0秒 |
| セグメント数 / トーク数 | 5,000 / 257 | 5,000 / 257 | 5,000 / 257 |
| 合計 | 22.2 時間 | 20.9 時間 | 22.2 時間 |
| 字幕文字数 | 409,033 | 409,033 | 410,838 |
| 発話ID | 5,000本すべて -N と共通 | 5,000本すべて -V と共通 | 67本が -V / -N と一致 |

-Vと-Nはセグメントの分割位置が同じなので、スコアを直接比較できます。スコアの差がロバスト性の指標になります。
-Cは採用したセグメントがほぼ異なるため、スコアの直接比較はできません。

## シーケンスの作り方

| 項目 | 内容 |
|---|---|
| セグメント長 | 2〜30秒を1秒刻みのビンで均一分布 |
| 連結 | 隣接する字幕を連結。切れ目は0.5秒以上の間がある箇所のみ |
| 端のマージン | 前後に0〜200msを独立に付与 |
| 起点 | トーク内からランダムに選択 |
| シーケンス長 | 素材が許す限り4セグメント以上、上限20。平均7.2、中央値5 |
| シーケンスが跨げる無音 | セグメント間で5秒まで |

シーケンスは、トークが連続でなくなる箇所で終わります。

- 字幕が5秒以上空いている箇所。これだけ間が空くと、話題が変わっている可能性や字幕が欠落している可能性があります。
- セグメントに切り出せない発話に行き当たったとき。30秒を超える単一の字幕や、前後に長い無音がある短い孤立した字幕です。その発話はデータセットに含めず、次のシーケンスはその後ろから始まります。
- セグメントが20本に達したとき。1つのトークがデータセットを占有しないための上限です。

1トークあたりのセグメント数は平均19.5本、1シーケンスは平均7.2本なので、1トークは通常2〜3本のシーケンスを含みます。

## 統計

| | |
|---|---|
| セグメント数 | 5,000 |
| シーケンス数 | 698 |
| シーケンスあたりのセグメント数 | 1〜20、平均7.2、中央値5 |
| 直前のセグメントがデータセット内にあるもの | 4,302本（86.0%） |
| 1セグメントが遡れる履歴 | 平均5.5本、最大19本 |
| 合計 | 22.2 時間 |
| 音声長 | 2.0〜30.0 秒、平均 16.0 秒 |
| トーク数 | 257 |
| 字幕文字数 | 410,838 |
| 形式 | FLAC、16 kHz、モノラル |

## シーケンスの使い方

`manifest.jsonl` には他のデータセットに無いフィールドが3つあります。

| フィールド | 意味 |
|---|---|
| `seq_id` | そのセグメントが属するシーケンス |
| `seq_index` | シーケンス内での位置（0始まり） |
| `seq_len` | そのシーケンスのセグメント数 |

ファイルはシーケンス順に書かれているので、上から読めば各シーケンスが発話された順に並びます。`seq_index` が 0 のセグメントには履歴がありません。

`prev_context.json` は、各セグメントの直前にあたる同一トーク内の字幕を、古い順に最大8個ずつ収めたものです。他の2つのデータセットに合わせて置いています。
Whisperの `--condition-on-previous-text` のような、直前の文脈に条件付けする機能の性能測定においては、`prev_context.json` ではなく実際のモデルの出力を渡すことを推奨します。実際の運用では出力の誤りが伝播する可能性があるためです。

## 注意事項

- 字幕は逐語記録ではなく読みやすさを優先しており、フィラーは書かれていません。言い淀みまで書き起こすシステムは挿入誤りとして減点されます。
- 数字は半角アラビア数字です。
- 字幕には句読点と記号が残っています。CERの計算前に、空白とあわせて字幕とモデル出力の両方から除去してください。
- 音声は同梱していません。TEDxトークは CC BY-NC-ND 4.0 ライセンスのため、本リポジトリには分割情報と字幕のみを収録しています。`rebuild.py` を実行することで、データセットを再構築できます。
- 動画が非公開化または削除されると、そのセグメントは再構築できません。`rebuild.py` は該当分をスキップして一覧を表示します。欠けたセグメントを含むシーケンスは途切れているので、そのセグメントだけを除くのではなくシーケンス全体を除外してください。
- 5,000本のうち67本は、字幕を含めて TEDxJP-5K-V のクリップと完全に同一です。端のランダムマージンを両データセットで同じシードから引いているため、同じ字幕を覆うセグメントは同じ切り出しになります。共通なのはこれだけで、残る4,933本の切れ目は異なります。
- 元動画は TEDxJP-10K と共通ですが、分割点が異なるため、クリップが一致することはありません。

## ディレクトリ構成

```
prev_context.json  各セグメントの直前最大8セグメント分の字幕（本データセット自身の行から生成）
plan.json          分割情報（元トーク・切り出し範囲・字幕・シーケンス）
manifest.jsonl     NeMo形式マニフェスト。1行1セグメント、シーケンス順
text               発話IDと字幕（Kaldi形式）
segments utt2spk spk2utt   Kaldi形式メタデータ
rebuild.py         元トークを取得し clips/ を生成
clips/ source/     rebuild.py が生成（追跡対象外）
```

## 再構築

```sh
pip install numpy soundfile "yt-dlp[default]"
python rebuild.py
```

ffmpegがPATH上に必要です。YouTubeからのダウンロードには、Node.jsなどのJavaScriptランタイムも要ります。yt-dlpは最新版を使ってください。YouTube側の変更で古いバージョンはダウンロードに失敗することがあり、修正が正式リリースに入るまではnightlyビルドが必要な場合もあります。

## ライセンス

本リポジトリは Apache-2.0 です。分割情報・メタデータ・スクリプトが対象になります。

`rebuild.py` が生成する音声は対象外です。元トークは CC BY-NC-ND 4.0 のままで、権利は各講演者およびTEDx主催者に帰属します。
