# Grabette 実演データの SO-101 への変換と模倣学習

人が手持ちグリッパー（Grabette）で物体を掴んだ実演データを、小型ロボットアーム SO-101 の関節角に変換し、
模倣学習用のデータセットを作成して、Diffusion Policy（DP）と ACT を学習し、シミュレーション上で動作を確かめるまでの手順・結果考察をまとめたものである。

## ノートブックの構成

番号順に実行。

| ノートブック | 内容 |
|---|---|
| `00_data_acquisition.ipynb` | 外部データ（Grabette データセット、SO-101 モデル）の取得と記録 | 
| `01_grabette_data_and_coordinates.ipynb` | Grabette データの中身の理解、軌跡の復元、カメラ座標系 → ロボット座標系の対応の決定 |
| `02_so101_initial_pose_and_ik.ipynb` | SO-101 の関節・グリッパーの向き、初期姿勢、FK の確認、IK の成功と失敗 |
| `03_retargeting_visual_check.ipynb` | 1 エピソードのリターゲット、実データでの IK の失敗、変換結果の動画による目視確認 | 
| `04_dataset_build.ipynb` | 全エピソードの変換、選別（IK 成功率・手首カメラの向き）、LeRobot v3.0 形式での保存、学習時データセットとの照合 | 
| `05_policy_training_results.ipynb` | DP・ACT の学習の記録（学習用 PC での実行の転記）と、学習ログ・保存モデルの確認、学習済みモデルによる予測 | 
| `06_policy_simulation.ipynb` | 学習済みの DP・ACT を、缶を置いた MuJoCo シミュレーションで動かす。失敗の原因の診断と考察 |

05 の学習は GPU を持つ別の PC（学習用 PC）で行った。
05 では、学習用 PC でのコードと出力を転記したうえで、学習で保存されたファイルをこの PC で読み込んで確認している。

## データの入手元

| データ | 入手元 | バージョン |
|---|---|---|
| Grabette 実演データセット | <https://huggingface.co/datasets/SteveNguyen/grabette_pick3_cartesian_480> | リビジョン `d46306f6a58e7c83c144ee41af4e893b788d6f9f` |
| SO-101 MuJoCo モデル | <https://github.com/google-deepmind/mujoco_menagerie>（`robotstudio_so101/`） | コミット `ac6b2b09983786f3036cab1000221017fa2193b4` |

取得は `00_data_acquisition.ipynb` が行い、`data/` に保存。

## 作成したデータセットと学習した方策

| データセット（`outputs/`） | エピソード | フレーム | 学習した方策 | 
|---|---|---|---|
| 基本版 | 457 | 69,555 | ACT（150 エポック） |
| 手首カメラ選別版 | 437 | 66,467 | DP（150 エポック） | 
