# Structure-From-Motion-MVG-MVS
少し古そうに見えるが、OpenMVGとOpenMVSを使用した、画像からの三次元情報復元技術を試す。

## OpenMVGの環境構築
Dockerによる環境構築  
公式の方法を用いる。
[OpenMVG (Open Multiple View Geometry) Build instructions #useing docker](https://github.com/openMVG/openMVG/blob/master/BUILD.md#using-docker)

現状では、`openMVG\Dockerfile`を用いている。

ビルド後データは`/opt/openMVG_Build`に配置される。


## OpenMVSの環境構築
Dockerによる環境構築  
`openmvs/openmvs-ubuntu:latest`があったためこれを用いる。

## MeshLabについて
MeshLabは[公式サイト](https://www.meshlab.net/)にて各プラットフォーム向けのビルドがダウンロードできるのでそれに従ってインストールをする。
特にWindows環境ではMicrosoft Storeからインストールするのが妥当だろう。


## 環境構築の作業手順
1. このレポジトリをクローン
1. OpenMVGをクローン
   - `git clone --recursive https://github.com/openMVG/openMVG.git`
2. OpenMVSをクローン
   - `git clone --recurse-submodules https://github.com/cdcseacave/openMVS.git`
3. 起動
   - `docker-compose up -d`
4. `./dataset`がコンテナ内からも見れるのでそこにデータを置いて各種コマンドで利用する


## 主要なコマンド


## 参考文献
- [Structure From Motion (SfM)を試す 〜 OpenMVG編 (Ubuntu 16.04) 〜](https://qiita.com/fujin/items/d816a7e9b8c2577a7e37)
- [MeshLab](https://www.meshlab.net/)
- [openMVG/wiki](https://github.com/openMVG/openMVG/wiki)
- []()


## 開発中のメモ
### 20240710昼
dockerによる環境構築が終わったため実際にOpenMVGによる三次元再構成を試す。
[コマンドはここから取得](https://github.com/openMVG/openMVG/wiki/OpenMVG-on-your-image-dataset)

#### OpenMVGのコンテナ内で実行
```bash
cd /opt/openMVG_Build/software/SfM/
python3 SfM_SequentialPipeline.py /dataset/ImageDataset_SceauxCastle-master/images/ /dataset/ImageDataset_SceauxCastle-master/test_reconstruct
```

手法的にカメラパラメータが必要である。実際にデータセットを見つけた。試しにiPhoneがあるかを調べた。
```bash
cat /openMVG/src/openMVG/exif/sensor_width_database/sensor_width_camera_database.txt | grep iPhone
"iPhone 3;1";4.74
iPhone 3;1;4.7
iPhone 3G;3.56
iPhone 3GS;3.6
iPhone 4;4.54
iPhone 4S;4.54
iPhone 5;4.54
iPhone 5c;4.54
iPhone 5s;4.89
iPhone 6 Plus;4.89
iPhone 6;4.89
iPhone 6s Plus;4.89
iPhone 6s;4.89
iPhone 7 Plus;4.89
iPhone 7;4.89
iPhone 8 Plus;4.89
iPhone 8;4.89
iPhone SE;4.89
iPhone X;4.89
iPhone XR;5.6
iPhone XS Max;5.6
iPhone XS;5.6
"iPhone3;1";4.74
iPhone3;1;4.7
"iPhone4;1";4.57
iPhone4;1;4.5
iPhone4S;4.57
"iPhone5;1";4.57
"iPhone5;2";4.57
"iPhone5;3";4.57
"iPhone5;4";4.57
iPhone5;1;4.5
"iPhone6;1";4.89
"iPhone6;2";4.89
iPhone6;1;4.8
Prestigio MultiPhone 5550 Duo;4.54
```
最新のiPhoneがデータとしてないことを確認した。

自前のデータで再構成する場合はexifのカメラ名にあわせて、センサーサイズ的な情報を記載しなければ動かないと思われる。

#### OpenMVSのコンテナ内で実行
再構成結果である`/dataset/ImageDataset_SceauxCastle-master/test_reconstruct`を用いて、MVSを実行したい。

```bash
openMVG_main_openMVG2openMVS -i dataset/ImageDataset_SceauxCastle-master/test_reconstruct/reconstruction_sequential/sfm_data.bin -o dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene.mvs -d dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene_undistorted_images
```