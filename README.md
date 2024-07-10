# Structure-From-Motion-MVG-MVS
少し古そうに見えるが、OpenMVGとOpenMVSを使用した、画像からの三次元情報復元技術を試す。

## OpenMVGの環境構築
Dockerによる環境構築  
公式の方法を用いる。
[OpenMVG (Open Multiple View Geometry) Build instructions #useing docker](https://github.com/openMVG/openMVG/blob/master/BUILD.md#using-docker)

現状では、`openMVG\Dockerfile`を用いている。

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
- 
