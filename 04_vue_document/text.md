Vue ドキュメントや Vue Udemy 教材を見て、知った知識を紹介

### クラス名を動的に付け替えたいよ
プロジェクト内で保持している状態に応じて、CSSのクラス名を追加削除したいときがあります。  
自分は当初、下記のように関数を呼び出して返却値としてクラス名を受け取るようにしていました。  

```html
<div :class="getClassName(flg)">...</div>
```

```javascript
getClassName(flg) {
    return flg ? "color-blue" : "";
}
```

これをVue.jsの記述だと以下のようにできます。

```html
<!-- <div :class="{ クラス名: 真偽値 }">...</div>  -->
<div :class="{ 'color-blue': isActive }">...</div>
```
`:class`の中にオブジェクトを入れて、keyに適用するクラス名、valueに適用するか否かの真偽値を入れます。真偽値がtrueのとき、"color-blue"のクラスが適用されます。  
このオブジェクトの中に複数の設定をすることができるので、状態が複数あり、それらによって細かくCSS制御をしたいときも、シンプルに記述できます。  クラス名が一語の場合はシングルクォーテーションは必要ありません。

```html
<div :class="{ 'color-blue': isActive, hidden: isHidden }">...</div>
```

参考：[HTML クラスのバインディング](https://jp.vuejs.org/v2/guide/class-and-style.html#HTML-%E3%82%AF%E3%83%A9%E3%82%B9%E3%81%AE%E3%83%90%E3%82%A4%E3%83%B3%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0])

### 表示を切り替えたいよ
要素の表示非表示をさせるときに、Vue.jsでは`v-if`もしくは`v-show`を頻繁に利用します。
```html
<div v-if="true">v-ifにtrueがセットされたら、この要素は表示される</div>
<div v-else>v-ifがfalseであれば、この要素が表示される</div>

<div v-show="true">v-showにtrueがセットされたら、この要素は表示される</div>
```
`v-if`はif文のようにelseも使用することができます。

`v-if`は非表示にされたときはその要素ごと、HTMLのDOMから削除されます。表示されたときはその要素が、新しくDOMに追加されます。
`v-if`で表示非表示を制御しているVueコンポーネントは、生成と破棄が行われることになるので、createdやmountedなどのライフサイクルメソッドもその都度実行されるようになります。  
createdでWebAPI通信処理を行っていたりすると大変なことになります。   

`v-show`はCSSのdisplayプロパティを切り替えてるだけなので、常にHTMLのDOM内に存在します。そのため、`v-if`で表示非表示の制御を行うよりもパフォーマンス面では優れています。
また`v-show`で表示非表示を制御しているVueコンポーネントは、例え非表示(`v-show=false`)になっていたとしても、コンポーネントのライフサイクルメソッドは実行されるので、この点は注意が必要です。

テキストの表示非表示など、シンプルな要素の切り替えにはなるべく`v-show`を使った方が良いと思います。

また余談ですが、`v-if`は[template構文](https://jp.vuejs.org/v2/guide/syntax.html)内で使用できます。template構文は、実際に描画されているDOMには表示されないので、v-ifのために無駄にdivタグなどの要素を増やすことが防げてデバッグがしやすくなります。
```html
<!-- divタグがDOMに表示される -->
<div v-if="true"><span>v-ifにtrueがセットされたら、この要素は表示される</span></div>
<!-- DOMにはspanタグしか表示されない -->
<template v-if="true"><span>v-ifにtrueがセットされたら、この要素は表示される</span></template>
```

また個人的に、`v-if`、`v-else`を書くときは、template構文にした方が分岐箇所が分かりやすく感じるので、以下のように書くことが多いです。
```html
<!-- divタグとbuttonタグが関連しているのが分かりづらい -->
<div v-if="true" class="XXX" id="XXXX" name="XXXX">
    <span>テキスト表示</span>
</div>
<button v-else>
    BUTTON
</button>

<!-- templateで分けることで、v-ifの処理が関連している要素がわかりやすい -->
<template v-if="true">
    <div class="XXX" id="XXXX" name="XXXX">
        <span>テキスト表示</span>
    </div>
</template>
<template v-else>
    <button>
        BUTTON
    </button>
</template>
```
コード量がこれだけだと少し冗長に感じますが、要素が増えた時などはtemplateが目印になるので読みやすく感じます。



参考：[条件付きレンダリング](https://jp.vuejs.org/v2/guide/conditional.html)

















### props として受け取った値を変更させたいよ

data 定義、computed 定義を紹介

### 子コンポーネントで入力した内容を親要素に教えたいよ

自分がやっていた失敗談載せる
.sync 修飾子の話

### 親から子のスロットを操作したいよ

テーブル操作をするときなどに便利

### タブ切り替えでも値を保存しておきたいよ

keep-alive を使用する
動的コンポーネントもここで紹介
v-if,v-show との違いを少し調べる

### 初回描画を早くしたいよ

import の書き方の工夫、Udemy が詳しい
遅延ローディング

### 色情報とかは Vuex にもコンポーネントでも保持したくないよ

Provide、Inject の紹介、カラーテーマや画面サイズの他に活用方法あるか調べる。

### 処理を共通化したいよ

毎回同じような comuted 処理をしている場合
ミックスインの使用を検討する

### カスタムディレクティブの使い方

DOM 操作について共通化させたいときに使うイメージ。
公式ドキュメントでは初回描画時にフォーカルを当てたり。

### 複数のテキストを変換させたいよ

フィルターつかう。

## Vue.js 以外のこと

### 入力の最中に API 通信したいよ

lodash の Debounce 関数

### 画面のレイアウトを固定したいよ

ルーティング設定の components 内の名前と、router-view の名前を関連付ける。
実際に実装してみる。

### 全画面共通の処理をはさみたいよ

router.bedoreEach の紹介
ログインしてないユーザだったリダイレクトさせる処理を加える。

### 特定の画面における処理を共通化させたいよ

特定のページで条件によってリダイレクト処理させたい場合。
beforeRouteEnter などを使う
