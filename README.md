# Structure-From-Motion-MVG-MVS
少し古そうに見えるが、OpenMVGとOpenMVSの

## OpenMVGの環境構築
Dockerによる環境構築  
公式の方法を用いる。
[OpenMVG (Open Multiple View Geometry) Build instructions #useing docker](https://github.com/openMVG/openMVG/blob/master/BUILD.md#using-docker)

### 作業手順
1. このレポジトリをクローン
1. OpenMVGをクローン
   - ```bash
    git clone --recursive https://github.com/openMVG/openMVG.git
   ```
2. OpenMVSをクローン
   - ```bash
   git clone --recurse-submodules https://github.com/cdcseacave/openMVS.git
   ```
3. 起動
   - ```bash
   docker-compose up -d
   ```
4. `./dataset`がコンテナ内からも見れるのでそこにデータを置いて各種コマンドで利用する
