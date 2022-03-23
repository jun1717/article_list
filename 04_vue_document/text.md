## はじめに
現在Vue(v2)を使用して開発を行っているのですが、実装していくともっと効率的に書けそうと思うことが増えてきました。  
そこでVueを触ってから約半年ほど経ったので改めてVueの公式ドキュメントを読んで、もっと早く知りたかったことなどをメモ書き程度にまとめてみました。  
これからVueを触ってみようと思っている人に、少しでも役立てば幸いです。


一点注意事項として、今回読んだ公式ドキュメントは[Vue2](https://jp.vuejs.org/v2/guide/index.html)のものになりますので、Vue3では非推奨や廃止になっているものもある可能性があります。


## 学んだこと
「XXしたいとき、〇〇が使える」というような形式で、学んだことを紹介していきます。

### クラス名を動的に付け替えたいよ
プロジェクト内で保持している状態に応じて、CSSのクラス名を動的に変更したいときがあります。  
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
クラス名の中にオブジェクトを入れて、keyに適用するクラス名、valueに適用するか否かの真偽値を入れます。真偽値がtrueのとき、"color-blue"のクラスが適用されます。  
このオブジェクトの中に複数の設定をすることができるので、状態が複数あり、それらによって細かくCSS制御をしたいときも、シンプルに記述できます。　　
クラス名が一語の場合はシングルクォーテーションは必要ありません。

```html
<div :class="{ 'color-blue': isActive, hidden: isHidden }">...</div>
```

参考：[HTML クラスのバインディング](https://jp.vuejs.org/v2/guide/class-and-style.html#HTML-%E3%82%AF%E3%83%A9%E3%82%B9%E3%81%AE%E3%83%90%E3%82%A4%E3%83%B3%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0])


### 表示を切り替えたまたまた
要素の表示非表示をさせるときに、Vue.jsでは`v-if`もしくは`v-show`を頻繁に利用します。2つの方法があることは知っていましたが、違いや使い分けが曖昧なままだったので改めて理解しておこうと思いました。

```html
<div v-if="true">v-ifにtrueがセットされたら、この要素は表示される</div>
<div v-else>v-ifがfalseであれば、この要素が表示される</div>

<div v-show="true">v-showにtrueがセットされたら、この要素は表示される</div>
```
`v-if`はif文のようにelseも使用することができます。

`v-if`は非表示にされたときはその要素ごと、HTMLのDOMから削除されます。表示されたときはその要素が、新しくDOMに追加されます。
`v-if`で表示非表示を制御しているVueコンポーネントは生成と破棄が行われることになるので、createdやmountedなどのライフサイクルメソッドもその都度実行されるようになります。  
createdでWebAPI通信処理を行っていたりすると大変なことになります。   

`v-show`はCSSのdisplayプロパティを切り替えてるだけなので、常にHTMLのDOM内に存在します。そのため、`v-if`で表示非表示の制御を行うよりもパフォーマンス面では優れています。
また`v-show`で表示非表示を制御しているVueコンポーネントは、例え非表示(`v-show=false`)になっていたとしても、コンポーネントのライフサイクルメソッドは実行されるので、この点は注意が必要です。

テキストの表示非表示など、シンプルな要素の切り替えにはなるべく`v-show`を使った方が良いと思います。

また余談ですが、`v-if`は[template構文](https://jp.vuejs.org/v2/guide/syntax.html)内で使用できます。template構文は、実際に描画されているDOMには表示されないので、v-ifのために無駄にdivタグなどの要素を増やすことが防げてデバッグがしやすくなります。DOMがないので、tmplate構文にはCSSのdisplayプロパティの切替を行う`v-show`は利用できません。

```html
<!-- divタグがDOMに表示される -->
<div v-if="true"><span>v-ifにtrueがセットされたら、この要素は表示される</span></div>
<!-- DOMにはspanタグしか表示されない -->
<template v-if="true"><span>v-ifにtrueがセットされたら、この要素は表示される</span></template>
```

また個人的に、`v-if`、`v-else`を書くときは、template構文にした方が分岐箇所が分かりやすく感じるので、以下のように書くことが多いです。
```html
<!-- divタグとbuttonタグがif文で関連しているのが分かりづらい -->
<div v-if="true" class="XXX" id="XXXX" name="XXXX">
    <span>テキスト表示</span>
</div>
<button v-else @click="XXXX">
    BUTTON
</button>

<!-- templateで分けることで、if文が関連している要素がわかりやすい -->
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
コード量がこれだけだと少し冗長に感じますが、要素が増えた時などはtemplateが目印になるので読みやすく感じます。またv-ifはタグ内の属性の先頭に書くといったルールが合っても良さそうです。



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

「propsの値を直接変更してはいけない」はSPAでは当然の決まりですが、Vueでそれを回避するための方法が曖昧だったので、2つの方法だけということが知れて良かったです。


参考：[単方向のデータフロー](https://jp.vuejs.org/v2/guide/components-props.html#%E5%8D%98%E6%96%B9%E5%90%91%E3%81%AE%E3%83%87%E3%83%BC%E3%82%BF%E3%83%95%E3%83%AD%E3%83%BC)





### 子コンポーネントで入力した内容を親要素に反映したいよ
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

私は上記のような場合は、子コンポーネントでpropsを初期値とした入力情報を格納する状態を新たに定義し、その状態をv-modelにセットするようにしていました。そして入力操作が行われるたびに、$emitを実行して親コンポーネントの状態を更新する、という感じで実装しました。  
一応親から子へと単一方向のデータフローになっているので問題はないですが、冗長です。

私が実装した方法をもっとスマートに書く方法として、[sync修飾子](https://jp.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A3%BE%E5%AD%90)があります。  
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

そして子コンポーネントではpropsとして受け取った値をそのまま、inputタグのvalue属性にセットしています。さらに入力ごとに実行されるinputイベントには`$emit`がセットされています。`$emit`の第一引数には更新対象の状態の名前、第二引数には入力された値が入っています。  
これで親コンポーネントで定義した状態を`$emit`を通じて更新してくれるようになります。わざわざ子コンポーネントで新たに状態を定義する必要がなくなります。

また上記のコードでは親コンポーネントでsync修飾子をつける際に、name、ageと一つずつ定義していますが、オブジェクトごと定義することもできます。
```html
<user-input-form v-bind.sync="user" />
```

参考：[.sync修飾子](https://jp.vuejs.org/v2/guide/components-props.html#%E5%8D%98%E6%96%B9%E5%90%91%E3%81%AE%E3%83%87%E3%83%BC%E3%82%BF%E3%83%95%E3%83%AD%E3%83%BC)


### タブ切り替え後も値を保存しておきたいよ
タブのように、状態によって表示するコンポーネントを切り替えたい場合は、componentタグのis属性にコンポーネント名を入れるようにすれば切り替えられます。`v-if`や`v-show`でも表示の切り替えを行うことができますが、切り替えるコンポーネント数が多い場合は`v-if`や`v-show`を使用するとその数だけHTMLのコード量が多くなります。しかしcomponentタグを使用すれば一行で済みます。
```html
<template>
  <div>
    <!-- currentComponent変数に表示するコンポーネント動的に変更して表示を切り替える-->
    <component :is="currentComponent"></component>
  </div>
</template>
```

このcomponentタグを使用してコンポーネントの切り替えを実行すると、非表示になるコンポーネントは削除されるのでinputタグに入力した情報なども消えてしまいます。  
このとき入力情報を保持しておきたい場合は、componentタグを`keep-alive`タグで囲います。
```html
<template>
  <div>
    <keep-alive>
      <component :is="currentComponent"></component>
    </keep-alive>
  </div>  
</template>
```
`keep-alive`タグを使用すると、切り替え時にコンポーネントの状態をキャッシュに保持してくれるようになります。  
一点注意事項として、必ずkeep-aliveタグの直下にcomponentタグが置く必要があります。コメントアウトなどもダメです。

参考：[動的コンポーネントにおける keep-alive の利用](https://jp.vuejs.org/v2/guide/components-dynamic-async.html#%E5%8B%95%E7%9A%84%E3%82%B3%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%8D%E3%83%B3%E3%83%88%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B-keep-alive-%E3%81%AE%E5%88%A9%E7%94%A8)



### 複数のテキストを変換させたいよ
先頭の文字だけ大文字に変換するなど、テキストの表記を加工したい場合はフィルターを使用します。
```html
<template>
  <div>
    {{name | nameFormat}}
    <!-- John と表示される -->
  </div>  
</template>

<!-- 省略 -->
<script>
export default {
  data:{
    name: 'john'
  },
  filters:{
    // 引数valueにはdata定義したnameが入る
    nameFormat:function(value){
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
};
</script>
```

computedでもデータの変換を行うことができますが、複数のデータを変換しようとすると、そのデータの数だけcomputedで定義する必要があります。  
フィルターを使用すれば複数データの変換が簡潔に書けます。  
しかし再描画のたびに処理が実行されるので、computedに比べてパフォーマンス面は劣ります。


参考：[フィルター](https://jp.vuejs.org/v2/guide/filters.html)


### Vueコンポーネント内の処理を共通化したいよ
各コンポーネントで記述してるdata定義やmethods、ライフサイクル処理などのオプションを、Vue.jsのミックスインで共通化することができます。

```javascript
// mixin.js (ミックスインはJavaScriptファイルで書ける)
export default {
  data() {
    return {
      title: "タイトル"
    };
  },
  methods: {
    hello() {
      alert('hello')
    }
  }
};
```
上記のmixin.jsをコンポーネントから読み込むことで、data定義などをいつも通りに使用することができます。
```html
<template>
  <div>
    {{ title }}
    <!-- mixin.jsに定義したtitleが表示される -->
  </div>  
</template>
<script>
import mixin from "./mixin";
export default {
  mixins: [mixin],
  mounted() {
    this.hello(); // mixin.jsのhelloメソッドが呼ばれる
  }
};
</script>
```

注意点としてミックスインで定義したdataやメソッドと同じ名前をコンポーネント側でも定義した場合は、コンポーネント側がの定義が優先されます。
```html
<template>
  <div>
    {{ title }}
    <!-- mixin.jsに定義したtitleが表示される -->
  </div>  
</template>
<script>
import mixin from "./mixin";
export default {
  mixins: [mixin],
  methods: {
    hello() {
      alert('hello')　// 実行される
    }
  },
  mounted() {
    this.hello(); // このコンポーネントで定義したhelloメソッドが呼ばれる
  }
};
</script>
```
またライフサイクルメソッドに関しては、ミックスイン→コンポーネントの順に実行されます。どちらか片方が実行されるのではなく、両方のライフサイクルメソッドが実行されます。


参考：[ミックスイン](https://jp.vuejs.org/v2/guide/mixins.html)

しかし学んだはいいものの、Vue3ではミックスインの使用は[非推奨](https://vuejs.org/api/options-composition.html#mixins)となり、代わりにVue3からリリースされた[Composition API
](https://v3.ja.vuejs.org/api/composition-api.html)を使うことが推奨されています。  
Composition APIはdata定義やmethods関数、ライフサイクルメソッドなどをVueコンポーネント外に分離することができる、新しい記述方式です。


### ページ遷移したときに画面の一部を変更したい
ルーティング設定時に、表示するコンポーネントに名前をつけておきます。  
`/home`には名前を付けず表示するコンポーネント1つだけを定義(HomeComponent)、`/main`には表示するコンポーネントに名前を付けて複数定義(MainComponent, Content1Component, Content2Component)します。
```javascript
routes: [
    {
      path: "/home",
      component: HomeComponent,
    },
    {
      path: '/main',
      // オブジェクト形式でコンポーネントを定義。keyが名前でvalueにコンポーネントをセット
      components: {
        default: MainComponent,
        content1: Content1Component,
        content2: Content2Component,
      },
    }
  ],
```
描画用のコンポーネントに`router-view`を設置し、ルーティング設定時に付けた名前をname属性にセットします。
```html
<template>
  <div>
      <router-view/>
      <router-view name="content1" class="content"/>
      <router-view name="content2" class="content"/>
  </div>
</template>
```
`/home`のパスに遷移した場合は、name属性のない一番上の`router-view`にHomeComponentが表示されます。  
`/main`のパスに遷移した場合は、name属性のない一番上の`router-view`にdefaultとして設定したMainComponentが表示され、
name属性が「content1」となっている`router-view`にContent1Component、name属性が「content2」となっている`router-view`にContent2Componentが表示されるようになります。  

画面の構成はほとんど同じで、パスによって一部の表示内容を出し分けたいときなどに有効です。

### 全画面共通の処理をはさみたいよ
ページ遷移時に処理を挟みこみたい場合、Vue Routerのナビゲーションガードを利用することで実現できます。
例えばページ遷移のたびにユーザのログイン認証状態の確認を行い、ログイン画面にリダイレクトさせたりできます。

全画面共通でページ遷移時に処理を行いたい場合は、`router`のインスタンスに直接入れます。

```javascript
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  next();// nextを実行しないと画面遷移が行われない
})
```

画面によって処理を行いたい場合は、ルーティング設定内に記述できます。
```javascript
  routes: [
    {
      path: "/XXX",
      component: Sample,
      beforeEnter: (to, from, next) => {
        // 処理記述
      }
    },
  ],
```
今まで条件によってリダイレクトさせたい場合はコンポーネントのcreatedライフサイクルメソッドなどで行っていたので、ナビゲーションガードを有効活用したいと思いました。


参考：[ナビゲーションガード](https://router.vuejs.org/guide/advanced/navigation-guards.html)


## 終わりに
少し技術に触れてから公式ドキュメントを読み直すと、既存のプロジェクトのどこに使えそうかと考えられたり、今までの自分の実装方法と比較できたりするので、知識がより身に付きやすいような気がします。  

