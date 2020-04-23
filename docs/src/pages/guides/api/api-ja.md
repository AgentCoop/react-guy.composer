# APIの設計アプローチ

<p class="description">Material-UIの使用方法については多くのことを学び、v1のリライトによってコンポーネントAPIを完全に再考することができました。</p>

> API設計が難しいのは、単純に見えるようにしても実際にはかなり複雑に見えるようにしたり、単純だが複雑に見えるようにしたりできるからです。

[@sebmarkbage](https://twitter.com/sebmarkbage/status/728433349337841665)

Api設計が難しいのは、単純に見えるようにしても実際にはかなり複雑に見えるようにしたり、単純だが複雑に見えるようにしたりできるからです。 組版機能を最大限に活用するため、低レベルのコンポーネントを提供しています。

## コンポジション

コンポーネントの構成に関してAPIに矛盾があることに気付いたかもしれません。 透過性を提供するために、APIを設計する際に次のルールを使用しています。

1. `children` プロパティの利用は、Reactで構成を行う慣用的な方法です。
2. 場合によっては、子どもの順序の入れ替えを許可する必要がない場合など、子どもの構成が限定されることもあります。 この場合、明示的なプロパティーを指定すると、実装がより単純になり、よりパフォーマンスが向上します。; たとえば、`Tab`は`アイコン`および`ラベル`プロパティを例に取ります。
3. APIの一貫性が重要です。

## ルール

上記の構成のトレードオフとは別に、次のルールを実施します。

### スプレッド

提供されたドキュメント化されていないプロパティはルート要素に広がります; たとえば、`className`プロパティはルートに適用されます。

ここで、`MenuItem`のリプルを無効にするとします。 スプレッド動作を利用できます。

```jsx
<MenuItem disableRipple />
```

` disableRipple` プロパティは次のように流れます：[` MenuItem `](/api/menu-item/) > [` ListItem `](/api/list-item/) > [` ButtonBase `](/api/button-base/) 。

### ネイティブプロパティ

提供されたドキュメント化されていないプロパティはルート要素に広がります; たとえば、[`className`](/customization/components/#overriding-styles-with-class-names)プロパティはルートに適用されます。

### CSS クラス

すべてのコンポーネントで、[`クラス`](/customization/components/#overriding-styles-with-classes)プロパティを使用してスタイルをカスタマイズできます。 クラス設計は、次の2つの制約に答えます: Material Design仕様を実装するのに十分なだけで、可能な限りクラス構造を単純にします。

- ルート要素に適用されるクラスは、常に`root`と呼ばれます。
- 既定のスタイルはすべて1つのクラスにグループ化されます。
- 非ルート要素に適用されるクラスには、要素の名前の接頭辞が付きます（例：ダイアログコンポーネントの` paperWidthXs`） 。
- boolean型のプロパティ**で適用される変数に接頭辞は付きません**。例えば、`rounded`プロパティで適用される`rounded`クラスのようになります。
- Enumプロパティ**によって適用されるバリアントはprifixされます**、(例：`color="primary"`プロパティによって適用される`colorPrimary`クラス)。
- バリアントには** 1レベルの特異性があります** 。 `color`および`variant`プロパティは、variantと見なされます。 スタイルの特殊性が低いほど、オーバーライドが簡単になります。
- バリアント修飾子の特異性を高めます。 私たちは既に疑似クラス(`:hover`, `:focus`など。)のためにそれをしなければなりません</strong>。 より多くの定型的なコストで、より多くの制御を可能にします。 もっと直感的になればいいのですが。

```js
const styles = {
  root: {
    color: green[600],
    '&$checked': {
      color: green[500],
    },
  },
  checked: {},
};
```

### ネストされたコンポーネント

コンポーネント内のネストされたコンポーネントには、次のものがあります。

- 最上位レベルのコンポーネント抽象化の鍵となる独自のフラット化されたプロパティー たとえば、`Input`コンポーネントの場合は`id`プロパティです。
- ユーザが内部レンダリングメソッドのサブコンポーネントを微調整する必要がある場合は、独自の`xxxProps`プロパティを使用します。 たとえば、`Input`を内部的に使用するコンポーネントの`inputProps`プロパティと`InputProps`プロパティを公開します。
- 独自の` xxxComponent `コンポーネントインジェクションを実行するためのプロパティ。
- ユーザーが命令型アクションを実行する必要がある場合は、独自の`xxxRef`プロパティ たとえば、`inputRef`プロパティを公開して、`Input`コンポーネントのネイティブ`入力`にアクセスします。 これは、[「DOM要素にアクセスするにはどうすればいいですか。」](/getting-started/faq/#how-can-i-access-the-dom-element)という質問に答えるのに役立ちます。

### プロパティの命名

ブーリアン型のプロパティの名前は、**のデフォルト値**に基づいて選択する必要があります。 たとえば、入力エレメントの`disabled`属性を指定すると、デフォルトで`true`になります。 このオプションを選択すると、次のような省略表記が可能になります。

```diff
-<Input enabled={false} />
+<Input disabled />
```

### 制御されたコンポーネント

ほとんどの制御対象コンポーネントは、`値`および`onChange`プロパティによって制御されます。 ただし、ディスプレイ関連の状態には、`open`/`onClose`/`onOpen`の組み合わせが使用されます。

### boolean vs enum

コンポーネントのバリエーションのためのAPIを設計するには、次の二つのオプションがあります。*boolean*; または*enum*を使用します。 たとえば、異なるタイプのボタンを選択します。 各オプションには長所と短所があります。

- Option 1 *boolean*:
    
    ```tsx
    type Props = {
    contained: boolean;
    fab: boolean;
    };
    ```
    
    このAPIは、簡略表記法を有効にしました： `<Button>`、` <2 /> ` 、` <3 /> ` 。

- Option 2 *enum*:
    
    ```tsx
    type Props = {
      variant: 'text' | 'contained' | 'fab';
    }
    ```
    
    このAPIはより冗長です： `<Button>`、`<Button variant="contained">`、`<Button variant="fab">`。
    
    ただし、無効な組み合わせの使用を防ぎ、 は公開されるプロパティの数を制限し、 は将来新しい値を簡単にサポートできます。

Material-UIコンポーネントは、次の規則に従って2つのアプローチの組み合わせを使用します。

- *ブーリアン*は、 **2**つの自由度が必要な場合に使用します。
- *enum*は、**2以上** の自由度が必要な場合、または将来さらに自由度が必要になる可能性がある場合に使用します。

前のボタンの例に戻ります。 3自由度が必要なので、* enumを使用します* 。

### Ref

` ref `はルート要素に転送されます。 つまり、レンダリングされたルート要素を変更せずに、 `component`<0>コンポーネント</0>プロパティを介して、コンポーネントがレンダリングする最も外側のDOM要素に転送されます。 `コンポーネント`プロパティを介して別のコンポーネントを渡すと、代わりに参照がそのコンポーネントにアタッチされます。

## 用語集

- **host component**:`react-dom`のコンテキストにおけるDOMノードタイプ、例えば`'div'`。 [ React Implementation Notesも参照してください](https://reactjs.org/docs/implementation-notes.html#mounting-host-elements) 。
- **host element** ：` react-domのコンテキストのDOMノード`たとえば、` window.HTMLDivElementのインスタンス` 。
- **outermost**:コンポーネントツリーを上から下に読み込むときの最初のコンポーネントです。つまり、幅優先の検索です。
- **root component** ：ホストコンポーネントをレンダリングする最も外側のコンポーネント。
- **root element**：ホストコンポーネントをレンダリングする最も外側の要素。