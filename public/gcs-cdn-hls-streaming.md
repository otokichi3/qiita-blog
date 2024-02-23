---
title: GCS 上の動画を Cloud CDN を経由してストリーミング配信する
tags:
  - CDN
  - HLS
  - googlecloud
private: false
updated_at: '2024-02-24T00:40:09+09:00'
id: 644541b0d5b4a674b6ac
organization_url_name: null
slide: false
ignorePublish: false
---
# GCS 上の動画を Cloud CDN を経由してストリーミング配信する

## 概要

GCS に HTML ファイル、動画ファイルを配置し、それをクライアントから観れるようにします。
一応ありがちな Cloud CDN を経由するパターンにします。GCS の組み込みキャッシュも効きますが、ここは Cloud CDN で。
ちなみに 2024/02/23 現在 Media CDN は Google にリクエストしないと使えず、[U-NEXT が本気で使っている](https://av.watch.impress.co.jp/docs/topic/1408035.html)代物で大衆化されてなさそうなので使ってません。

## 環境

- Windows 11 Pro
- Ubuntu 22.04 on WSL2
- Python 3.10.7

## やり方

Python を使うので必要な人は適当なディレクトリを掘って pyenv で local にバージョンを指定してください。以下は一例。

```bash
mkdir hls-poc && cd hls-poc
pyenv install 3.10.7
pyenv local 3.10.7
```

```
$ cat .python-version
3.10.7
```

### ffmpeg をインストールする

```bash
sudo apt -y update
sudo apt install ffmpeg
```

### 動画ファイルを取得する

ファイルが入り乱れるのでフォルダ内で作業します。

```bash
mkdir resource && cd resource
wget https://download.blender.org/peach/bigbuckbunny_movies/big_buck_bunny_720p_h264.mov
MOV=big_buck_bunny_720p_h264.mov
```

Big Buck Bunny(BBB) という動画が公開されています。感謝。ライセンスはご確認を。

https://download.blender.org/peach/bigbuckbunny_movies/

### ffmpeg でマニフェストとセグメントファイルを作成する

```bash
ffmpeg -i $MOV \
-c:v copy -c:a copy \
-f hls \
-hls_time 9 \
-hls_playlist_type vod \
-hls_segment_type fmp4 \
-hls_fmp4_init_filename "init.mp4" \
-hls_segment_filename "seg.%3d.mp4" \
media.m3u8
```

こんな感じに。元の mov ファイルは削除して大丈夫です。

```
$ ls
init.mp4     seg.003.mp4  seg.008.mp4  seg.013.mp4  seg.018.mp4  seg.023.mp4  seg.028.mp4  seg.033.mp4  seg.038.mp4  seg.043.mp4  seg.048.mp4  seg.053.mp4  seg.058.mp4  seg.063.mp4
media.m3u8   seg.004.mp4  seg.009.mp4  seg.014.mp4  seg.019.mp4  seg.024.mp4  seg.029.mp4  seg.034.mp4  seg.039.mp4  seg.044.mp4  seg.049.mp4  seg.054.mp4  seg.059.mp4  seg.064.mp4
seg.000.mp4  seg.005.mp4  seg.010.mp4  seg.015.mp4  seg.020.mp4  seg.025.mp4  seg.030.mp4  seg.035.mp4  seg.040.mp4  seg.045.mp4  seg.050.mp4  seg.055.mp4  seg.060.mp4  seg.065.mp4
seg.001.mp4  seg.006.mp4  seg.011.mp4  seg.016.mp4  seg.021.mp4  seg.026.mp4  seg.031.mp4  seg.036.mp4  seg.041.mp4  seg.046.mp4  seg.051.mp4  seg.056.mp4  seg.061.mp4  seg.066.mp4
seg.002.mp4  seg.007.mp4  seg.012.mp4  seg.017.mp4  seg.022.mp4  seg.027.mp4  seg.032.mp4  seg.037.mp4  seg.042.mp4  seg.047.mp4  seg.052.mp4  seg.057.mp4  seg.062.mp4
```

### GCS バケットを用意し、CORS ポリシーを更新する

バケット作成

```bash
PROJECT=[YOUR_OWN_PROJECT_ID]
cd ..
gsutil mb gs://$PROJECT
```

JSON ファイル作成

```cors.json
[
    {
      "origin": ["*"],
      "method": ["GET"],
      "responseHeader": ["Content-Type"],
      "maxAgeSeconds": 3600
    }
]
```

ポリシー適用

```bash
gcloud storage buckets update gs://$PROJECT --cors-file=cors.json
```

resource フォルダをアップロードする

```bash
gsutil cp -r resource gs://$PROJECT
```

### index.html をアップロードする

`videoSrc` は GCS にアップロードした際の media.m3u8 ファイルの GCS パスに変更してください。あとはそのままで OK です。
あと、GCS は外部に公開してください。これ重要です。

```html
<!DOCTYPE html>
<script src="https://cdn.jsdelivr.net/npm/hls.js@1"></script>
<div>
  <h2>HTTP Live Streaming</h2>
  <video id="video" width="640" height="360" controls></video>
</div>
<script>
  var video = document.getElementById("video");
  var videoSrc =
    "https://storage.googleapis.com/[YOUR_OWN_BUCKET_NAME]/resource/media.m3u8";
  if (Hls.isSupported()) {
    var hls = new Hls();
    hls.loadSource(videoSrc);
    hls.attachMedia(video);
  } else if (video.canPlayType("application/vnd.apple.mpegurl")) {
    video.src = videoSrc;
  }
</script>
```

```bash
gsutil cp index.html gs://$PROJECT
```

いったん、これで GCS にある index.html にアクセスすれば BBB が再生される状態です。確認してみてください。
以下が表示される Web ページです。すでに LB を用意した後の URL ですが、私の場合は `http://34.144.254.201/index.html` にアクセスしています。（すでにこの IP アドレスは解放しています。アクセスできませんので悪しからず。）
![bbb.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/236789/cccb8855-8487-5e5e-166d-45e379715a6f.png)

### Cloud CDN を有効化しつつ Cloud Load Balacing で公開する

今回は簡易に HTTP で公開します。
が、この手順についてはコンソール上でぽちぽちなり Terraform で管理するなりするための手順が公式ドキュメントにしっかり記載されていますので、こちらをご覧ください。

https://cloud.google.com/load-balancing/docs/https/ext-load-balancer-backend-buckets?hl=ja

### 念のため確認

まずは GCS の index.html に直アクセスした場合の様子です。
200 OK が返ってきています。
![devtool1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/236789/e5ced1c0-f2dd-4ee4-7d61-757d1e034100.png)

次に、LB 経由で index.html にアクセスした場合の様子です。
一回目は Cloud CDN にキャッシュが保存されていないので、開いたらまず画面更新してください。
すると、以下の結果となるはずです。
![devtool2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/236789/bb3f46a6-a1d7-9fbc-6711-3f9b21072aed.png)

[304 Not Modified](https://developer.mozilla.org/ja/docs/Web/HTTP/Status/304) が返ってきています。

> これはキャッシュされたリソースへの暗黙のリダイレクトです。

つまり、一回目のアクセスによって静的コンテンツは Cloud CDN（お近くのエッジキャッシュ）にキャッシュされ、二回目のアクセスは Cloud CDN のキャッシュにアクセスしています。
おめでとうございます 🎉

### 余談

実は、index.html はキャッシュヒットしていますが、その他の動画コンテンツは下図の通り `from disk cache`、つまりお使いのブラウザキャッシュから読み込まれています。
![devtool3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/236789/d6097698-d36d-9437-1a55-aa57b7d19bb8.png)

Cloud CDN と言えどそれを支えるのは単なるサーバです。できることならサーバの負荷は減らしたいですし、動画を見る側にとってもなるべく早く観たいはずです。
なので、毎回 Cloud CDN のキャッシュサーバにアクセスするのではなく、可能な範囲でブラウザのキャッシュから読み込むようになっています。
キャッシュに興味のある方は、以下 URL にアクセスしてキャッシュが保存されている場所を確認するなり、Chrome の設定画面よりキャッシュを削除するなりしてください。
chrome://version/

キャッシュなしで試したい場合は開発者ツールの Network タブより `Disable cache` を選択しましょう。毎回 200 OK で返ってくるようになります。
なぜこのチェックボックスをオンにしたらキャッシュヒットしなくなるの？と思った方、[キャッシュの世界](https://cloud.google.com/cdn/docs/caching)へようこそ。

![devtool4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/236789/f4e291dc-cb32-bb3e-b7fb-4271243285dd.png)

## 参考にしたサイト

HLS.js 公式 GitHub リポジトリ

https://github.com/video-dev/hls.js


localhost で簡単に HLS 配信を Python でしてみる

https://qiita.com/zeek0x/items/2e9b79c32617a74e9acd
