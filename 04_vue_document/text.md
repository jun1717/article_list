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
親コンポーネントから子コンポーネントへ状態(props)を渡したとき、その状態を変更させたいときがあります。このとき、propsの値を子コンポーネント内で直接変更することは禁止されています。理由は子コンポーネントで親コンポーネントの値を変更してしまうと、親コンポーネントがその変更を検知できず、再描画が行われないなどのバグが発生する可能性があるからです。  

Vueのドキュメントでは、以下の2つの方法でpropsの状態を変更させるように説明されています。

1. propsの値を初期値として、子コンポーネントのdataで新たに状態を定義する
```javascript
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```
2. propsの値を加工したいだけならば、computedを使用する
例えばpropsとして受け取った値を小文字に変換したり、配列から必要な情報だけ抽出したいときなどは、computedで状態を定義します。
```javascript
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

参考：[単方向のデータフロー](https://jp.vuejs.org/v2/guide/components-props.html#%E5%8D%98%E6%96%B9%E5%90%91%E3%81%AE%E3%83%87%E3%83%BC%E3%82%BF%E3%83%95%E3%83%AD%E3%83%BC)





### 子コンポーネントで入力した内容を親要素に教えたいよ
親コンポーネントで入力データを保持しており、子コンポーネントで入力欄を実装している場合を考えます。  
```javascript
// 親コンポーネントの状態例
export default {
  data() {
    return {
      user: {name: "XXXX", age: 20}
    };
  },
};

// 子コンポーネントの例
<template>
  <form>
    <label>
      名前
      <input type="text" v-model="user.name" />
    </label>
　　<label>
      年齢
      <input type="text" v-model="user.age" />
    </label>
  </form>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      default: () => ({})
    }
  }
};
</script>
```
Vue.jsでは入力操作に関する状態の変更方法として、[v-model](https://jp.vuejs.org/v2/guide/components-custom-events.html#v-model-%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA)が用意されています。v-modelに状態をセットするだけで、入力欄に記入した内容を状態にも反映してくれます。  
しかし、上のコードのようにpropsとして受け取った値を直接v-modelにセットすると、propsの値を直接変更することになるため、コンソールに警告が表示されます。  

私は上記のような実装をしたときは、子コンポーネントでpropsを初期値とした状態を新たに定義し、その状態をv-modelにセットするようにしました。そして入力操作が行われるたびに、$emitを実行して親コンポーネントの状態を更新する、という感じで実装しました。  
一応親から子へと単方向のデータフローになっているので問題はないですが、冗長ですね。

私が実装した方法をもっとスマートに書く方法として、[sync修飾子](https://jp.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A3%BE%E5%AD%90)を使用します。  
```javascript
// 親コンポーネントでsync修飾子を付けて状態を渡す
  <div>
    <user-input-form :name.sync="user.name" :age.sync="user.age" @submit="createUser" />
  </div>

// 子コンポーネント
<template>
  <form>
    <label>
      名前
      <input type="text" :value="name" @input="$emit('update:name', $event.target.value)" />
    </label>
　　<label>
      年齢
      <input type="text" :value="age" @input="$emit('update:age', $event.target.value)" />
    </label>
  </form>
</template>

<script>
export default {
  props: {
    name: {
      type: String,
      default: ""
    },
    age: {
      type: String,
      default: ""
    }
  }
};
</script>
```
親コンポーネントから渡す値のプロパティにsync修飾子を追記します。

そして子コンポーネントではpropsとして受け取った値をそのまま、inputタグのvalue属性にセットしています。さらに入力ごとに実行されるinputイベントには$emitがセットされています。$emitの第一引数には更新対象の状態の名前、第二引数には入力された値が入っています。  
これで親コンポーネントで定義した状態userの中身を更新してくれるようになります。わざわざ子コンポーネントで新たに状態を定義する必要がなくなります。

また上記のコードでは親コンポーネントでsync修飾子をつける際に、name、ageと一つずつ定義していますが、オブジェクトごと定義することもできます。
```html
<user-input-form v-bind.sync="user" />
```


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
