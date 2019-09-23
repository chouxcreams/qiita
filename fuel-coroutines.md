fuel-coroutinesを使った非同期HTTP通信がとても簡単だったのでメモ。

Fuel自体はKotlin全般で利用できますが、今回はAndroidアプリ開発を想定しています。

## Fuelとは
KotlinでHTTP通信できるライブラリです。[こちら](https://oldbigbuddha.dev/post/android-about-fluel/)にわかりやすく使い方がまとまっています。

[FuelのGitHubリポジトリ](https://github.com/kittinunf/fuel)を見るとわかると思うのですが、いろんなパッケージの集合体という形をとっています。今回利用するfuel-coroutinesはその中の一つです。

### なぜfuel-coroutinesなのか
Fuelの特徴として「同期でも非同期でも書ける」というものがありますが、fuelのみだと非同期まわりの融通があまり効きません。結局「コルーチンを自分で用意してそこに自前で同期処理を書く」ということになります。

fuel-coroutinesはKotlinのコルーチンライブラリであるKotlinX Coroutinesを利用しているため、コルーチンが扱いやすいです。

また、KotlinX Coroutinesと同じ書き方(というか同じものですが)でコルーチン制御できるため、検索しやすいです。(Kotlinのコルーチンについては[こちら](https://qiita.com/kawmra/items/ee4acb7db61f70dec9a8)がわかりやすいです。)

AndroidはメインスレッドでWebリクエストが送れないので、素直にfuel-coroutinesを使ったほうが色々楽です。

## 依存関係を書く
Build.gradleに依存関係を書きます

```Build.gradle
dependencies {
    implementation "com.github.kittinunf.fuel:fuel:2.2.1"
    implementation "com.github.kittinunf.fuel:fuel-android:2.2.1"
    implementation "com.github.kittinunf.fuel:fuel-coroutines:2.2.1"
}
```
fuelは様々なパッケージの集合体ですが、今回使うのは`fuel` `fuel-android` `fuel-coroutines`です。3つともgradleに書きましょう。

## 使ってみる
まずは[サンプル](https://github.com/kittinunf/fuel/tree/master/fuel-coroutines)を少し変えたものを。

```sample.kt
val url = "https://hoge.con"
val tag = "hoge"
runBlocking {
    val (request, response, result) = url.httpGet().awaitStringResponseResult()

    result.fold(
        { data -> Log.d(tag, data)},
        { error -> Log.e(tag, "An error of type ${error.exception} happened: ${error.message}") }
    )
}
```
見ての通りWebリクエストに成功したときは`result.fold`の第一引数の関数が、失敗のときは第二引数の関数が実行されます。

リクエストパラメータを追加したいときは`httpGet`の引数に`listOf(key to value)`を受け渡しましょう。

### 画像をダウンロードする
```image_download.kt
val image = runBlocking {
            val (_, _, result) = url.httpGet().awaitByteArrayResponseResult()
            val data = result.fold(
                {data-> data},
                {error->
                    Log.e(jsonDownloaderName, "An error of type ${error.exception} happened: ${error.message}")
                    null
                }
            )
            BitmapFactory.decodeByteArray(data, 0, data!!.size)!! //ここでダウンロードした画像をBitmap形式に変換する。
        }
```
敢えてさっきとは色々違う書き方をしましたが、基本的な違いは`awaitStringResponseResult`が`awaitByteArrayResponseResult`に変わっただけです。

## 参考
- https://oldbigbuddha.dev/post/android-about-fluel/
- https://qiita.com/kawmra/items/ee4acb7db61f70dec9a8
- https://qiita.com/pljp/items/a1f3e8d1d13c88a94907