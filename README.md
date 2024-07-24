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

`openmvs/openmvs-ubuntu:latest`では`openMVG_main_openMVG2openMVS`のバイナリが見当たらなかったため、`openMVS/docker/Dockerfile`をビルドすることにした。

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
- [openMVG/wiki/OpenMVG-on-your-image-dataset](https://github.com/openMVG/openMVG/wiki/OpenMVG-on-your-image-dataset)
- [openMVS/wiki/Usage](https://github.com/cdcseacave/openMVS/wiki/Usage#viewer-usage)


## 開発中のメモ
### 20240710昼
dockerによる環境構築が終わったため実際にOpenMVGによる三次元再構成を試す。
[コマンドはここから取得](https://github.com/openMVG/openMVG/wiki/OpenMVG-on-your-image-dataset)

#### OpenMVGのコンテナ内で実行
```bash
cd /opt/openMVG_Build/software/SfM/
python3 SfM_SequentialPipeline.py /dataset/ImageDataset_SceauxCastle-master/images/ /dataset/ImageDataset_SceauxCastle-master/test_reconstruct
/opt/openMVG_Build/install/bin/openMVG_main_openMVG2openMVS -i /dataset/ImageDataset_SceauxCastle-master/test_reconstruct/reconstruction_sequential/sfm_data.bin -o /dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene.mvs -d /dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene_undistorted_images

find /opt/openMVG_Build -iname openMVG_main_openMVG2openMVS
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
/openMVS_build/bin/DensifyPointCloud /dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene.mvs
```
```
root@ad2dc9b95679:/openMVS_build/bin# /openMVS_build/bin/DensifyPointCloud /dataset/test/scene.mvs 
06:39:48 [App     ] Build date: Dec 10 2019, 20:59:37
06:39:48 [App     ] CPU: Intel(R) Core(TM) i7-10700 CPU @ 2.90GHz (16 cores)
06:39:48 [App     ] RAM: 24.46GB Physical Memory 128.00GB Virtual Memory
06:39:48 [App     ] OS: Linux 5.15.153.1-microsoft-standard-WSL2 (x86_64)
06:39:48 [App     ] SSE & AVX compatible CPU & OS detected
06:39:48 [App     ] Command line: /dataset/test/scene.mvs
06:39:48 [App     ] error: invalid project
```
謎のエラーで点群を密にできない。
["invalid project" error in docker image when trying to compute dense point cloud](https://github.com/cdcseacave/openMVS/issues/1056)
と同じ問題のように思えるので、自前でビルドする。

```bash
VCPKG_ROOT=/vcglib cmake ..
CMake Deprecation Warning at CMakeLists.txt:30 (cmake_policy):
  The OLD behavior for policy CMP0011 will be removed from a future version
  of CMake.

  The cmake-policies(7) manual explains that the OLD behaviors of all
  policies are deprecated and that a policy should be set to OLD only under
  specific short-term circumstances.  Projects should be ported to the NEW
  behavior and not rely on setting a policy to OLD.


-- cotire 1.8.0 loaded.
-- Detected version of GNU GCC: 74 (704)
Compiling with C++14
CUDA_TOOLKIT_ROOT_DIR not found or specified
-- Could NOT find CUDA (missing: CUDA_TOOLKIT_ROOT_DIR CUDA_NVCC_EXECUTABLE CUDA_INCLUDE_DIRS CUDA_CUDART_LIBRARY) 
-- Can't find CUDA. Continuing without it.
-- WARNING: BREAKPAD was not found: Please specify BREAKPAD directory using BREAKPAD_ROOT env. variable
-- Can't find BreakPad. Continuing without it.
-- Boost version: 1.65.1
-- Found the following Boost libraries:
--   iostreams
--   program_options
--   system
--   serialization
--   regex
-- Eigen 3.2.10 found (include: /usr/local/include/eigen3)
-- OpenCV 3.2.0 found (include: /usr/include;/usr/include/opencv)
-- CGAL 4.11 found (include: /usr//include)
CMake Error at build/Utils.cmake:223 (message):
  VCG required, but not found: Please specify VCG directory using VCG_ROOT
  env.  variable
Call Stack (most recent call first):
  build/Modules/FindVCG.cmake:23 (package_report_not_found)
  libs/MVS/CMakeLists.txt:9 (FIND_PACKAGE)


-- Configuring incomplete, errors occurred!
See also "/openMVS/make/CMakeFiles/CMakeOutput.log".
```

cmake できない。以下のように環境変数を追加したらできた。

```bash
VCG_ROOT=/vcglib cmake ..
cmake --build .
```
でバイナリーができた。

実行したが、結果は変わらず'error: invalid project'となった。
```bash
openMVS/make/bin/DensifyPointCloud /dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene.mvs
02:23:59 [App     ] Build date: Jul 10 2024, 08:22:41
02:23:59 [App     ] CPU: Intel(R) Core(TM) i7-10700 CPU @ 2.90GHz (16 cores)
02:23:59 [App     ] RAM: 24.46GB Physical Memory 128.00GB Virtual Memory
02:23:59 [App     ] OS: Linux 5.15.153.1-microsoft-standard-WSL2 (x86_64)
02:23:59 [App     ] SSE & AVX compatible CPU & OS detected
02:23:59 [App     ] Command line: /dataset/ImageDataset_SceauxCastle-master/test_reconstruct/scene.mvs
02:23:59 [App     ] error: invalid project
```

### 20240724
OpenMVSについて調査する。

- [OpenMVGとOpenMVSのdockerを作る](https://zenn.dev/kenfuku/scraps/d5730e37444cbb)
- [Build latest OpenMVG+OpenMVS+COLMAP in docker](https://gist.github.com/cdcseacave/ef290eacefa44db67dadaed01a6fe319)
- []()


[Build latest OpenMVG+OpenMVS+COLMAP in docker](https://gist.github.com/cdcseacave/ef290eacefa44db67dadaed01a6fe319)が見た感じ動きそうなので、試してみる。


docker-compose-unified.ymlを作って構成を用意した。

ビルド後のファイルが削除されているように見えたのでその処理を消した。

```bash
python3 /openMVS/scripts/python/MvgMvsPipeline.py /dataset/ImageDataset_SceauxCastle-master/images /dataset/test20240724
```
動かない。

また、githubに書いてあることとdockercompose後の中身が一致しないため、一行にまとまっていたコマンドを分割して、複数のLayerに分けて実行してみたところ、git cloneでエラーがでていた。

```
 => ERROR [unified  7/23] RUN git clone https://github.com/colmap/colmap.git --branch dev                                                                                                                                         2.5s 
------
 > [unified  7/23] RUN git clone https://github.com/colmap/colmap.git --branch dev:
1.205 Cloning into 'colmap'...
1.797 fatal: Remote branch dev not found in upstream origin
------
failed to solve: process "/bin/sh -c git clone https://github.com/colmap/colmap.git --branch dev" did not complete successfully: exit code: 128
```

ブランチの指定をやめることで、クローンできた。
ビルドはできなかったため、ビルド直前のコンテナに入って、ビルドコマンドを手打ちして結果をみることにした。

```bash
cd /comap_build
cmake . ../colmap -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="/opt"
make -j4 && make install
```
cmake でエラーログができた。
```
root@338e1da66554:/comap_build# cmake . ../colmap -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="/opt"
-- Enabling LSD support
-- Found FreeImage
--   Includes : /usr/include
--   Libraries : /usr/lib/x86_64-linux-gnu/libfreeimage.so
CMake Error at cmake/FindFLANN.cmake:89 (message):
  Could not find FLANN
Call Stack (most recent call first):
  cmake/FindDependencies.cmake:17 (find_package)
  CMakeLists.txt:105 (include)


-- Configuring incomplete, errors occurred!
See also "/comap_build/CMakeFiles/CMakeOutput.log".
```

これの追加でビルド成功した。
```bash
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install \
  libflann-dev \
  libsqlite3-dev \
  libmetis-dev
```

次はOpenMVGのビルドで躓いた。
```bash
cd openMVG_build
cmake -DCMAKE_BUILD_TYPE=RELEASE \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    -DOpenMVG_BUILD_TESTS=OFF \
    -DOpenMVG_BUILD_EXAMPLES=OFF \
    -DOpenMVG_BUILD_DOC=OFF \
    -DOpenMVG_BUILD_OPENGL_EXAMPLES=ON \
    -DOpenMVG_USE_OPENCV=ON \
    -DOpenMVG_USE_OCVSIFT=OFF \
    -DCOINUTILS_INCLUDE_DIR_HINTS=/usr/include \
    -DCLP_INCLUDE_DIR_HINTS=/usr/include \
    -DOSI_INCLUDE_DIR_HINTS=/usr/include \
    -DEIGEN_INCLUDE_DIR_HINTS=/usr/include/eigen3 \
    ../openMVG/src
```

Eigenを利用しているようだったが、Dockerfile上ではインストール順が前後していたため入れ替えた。また、バージョンも3.4のほうが良いよいという報告（[recipe for target 'all' failed](https://github.com/openMVG/openMVG/issues/2143)）があったので、先に変えておいた。

`-DEIGEN_INCLUDE_DIR_HINTS=/usr/include/eigen3`を
`-DEIGEN_INCLUDE_DIR_HINTS=/usr/local/include/eigen34/include/eigen3/`に
変更した。

```bash
cmake -DCMAKE_BUILD_TYPE=RELEASE \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    -DOpenMVG_BUILD_TESTS=OFF \
    -DOpenMVG_BUILD_EXAMPLES=OFF \
    -DOpenMVG_BUILD_DOC=OFF \
    -DOpenMVG_BUILD_OPENGL_EXAMPLES=ON \
    -DOpenMVG_USE_OPENCV=ON \
    -DOpenMVG_USE_OCVSIFT=OFF \
    -DCOINUTILS_INCLUDE_DIR_HINTS=/usr/include \
    -DCLP_INCLUDE_DIR_HINTS=/usr/include \
    -DOSI_INCLUDE_DIR_HINTS=/usr/include \
    -DEIGEN_INCLUDE_DIR_HINTS=/usr/local/include/eigen34/include/eigen3/ \
    -DEIGEN3_INCLUDE_DIR=/usr/local/include/eigen34/include/eigen3 \
    ../openMVG/src
make -j4 && make install
```

```Dockerfile
RUN cd /openMVG_build && cmake \
    -DCMAKE_BUILD_TYPE=RELEASE \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    -DOpenMVG_BUILD_TESTS=OFF \
    -DOpenMVG_BUILD_EXAMPLES=OFF \
    -DOpenMVG_BUILD_DOC=OFF \
    -DOpenMVG_BUILD_OPENGL_EXAMPLES=ON \
    -DOpenMVG_USE_OPENCV=ON \
    -DOpenMVG_USE_OCVSIFT=OFF \
    -DCOINUTILS_INCLUDE_DIR_HINTS=/usr/include \
    -DCLP_INCLUDE_DIR_HINTS=/usr/include \
    -DOSI_INCLUDE_DIR_HINTS=/usr/include \
    ../openMVG/src && \
    make -j 4 && make install && \
    cp ../openMVG/src/openMVG/exif/sensor_width_database/sensor_width_camera_database.txt /opt/bin/

RUN cd /openMVS_build && cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DVCG_ROOT=/vcglib \
    -DCMAKE_TOOLCHAIN_FILE=/vcglib/scripts/buildsystems/vcpkg.cmake \
    -DEIGEN3_INCLUDE_DIR=/usr/local/include/eigen34/include/eigen3 \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    ../openMVS && \
    make -j4 && make install && \
    cp ../openMVS/MvgMvsPipeline.py /opt/bin/

RUN cd /openMVS_build && cmake \
    -DCMAKE_BUILD_TYPE=Release \
    -DVCG_ROOT=/vcglib \
    -DEIGEN3_INCLUDE_DIR=/usr/local/include/eigen34/include/eigen3 \
    -DCMAKE_INSTALL_PREFIX="/opt" \
    ../openMVS && \
    make -j4 && make install && \
    cp ../openMVS/MvgMvsPipeline.py /opt/bin/
```

あとはこの２レイヤーを動かせばいいだけ。



`pkg-config --modversion eigen3`

eigen3の古いもの（3.3.7）が使われているためエラーが出ると考えた。3.4をインストールしているのに3.3.4が使われていた。
`rm -rf /usr/lib/cmake/eigen3/`で削除した。
`ln -s /usr/local/include/eigen34/include/eigen3 /usr/lib/cmake/eigen3`をしてみた。
中身の構成が違うため、動かなかった。
```
root@f98f46d96fbf:/openMVG_build# ls /usr/lib/cmake/eigen3
Eigen3Config.cmake  Eigen3ConfigVersion.cmake  Eigen3Targets.cmake  UseEigen3.cmake

root@f98f46d96fbf:/openMVG_build# ls 
include  share

root@f98f46d96fbf:/openMVG_build# ls /usr/local/include/eigen34/include/eigen3
Eigen  signature_of_eigen3_matrix_library  unsupported

root@f98f46d96fbf:/openMVG_build# ls /usr/include/eigen3/
Eigen  eigen3  signature_of_eigen3_matrix_library  unsupported
```


[WWWサイトから、Stable Release (2022/10/17現在、version 3.4.0) のファイルを取ってきて、 インストールするのは簡単である。](http://nalab.mind.meiji.ac.jp/~mk/labo/text/nantoka-c++/node13.html)

