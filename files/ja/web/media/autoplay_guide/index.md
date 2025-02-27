---
title: メディアおよびウェブオーディオ API の自動再生ガイド
slug: Web/Media/Autoplay_guide
---

ページが読み込まれるとすぐに音声（または音声トラックを含む動画）の再生を自動的に開始することは、ユーザーにとって歓迎されない驚きです。 メディアの自動再生は便利な目的に役立ちますが、注意して必要なときにだけ使用してください。 ユーザーがこれを制御できるようにするために、ブラウザーは多くの場合、さまざまな形式の自動再生のブロック（autoplay blocking）を提供します。 このガイドでは、さまざまなメディアおよびウェブオーディオ API（Web Audio API）の自動再生機能について説明します — 自動再生の使用方法と自動再生のブロックを適切に処理するためのブラウザーの操作方法の簡単な概要を含みます。

ソースメディアに音声トラックがない場合、または音声トラックがミュートされている場合、自動再生のブロックは {{HTMLElement("video")}} 要素には適用*されません*。 アクティブな音声トラックを持つメディアは**可聴である**と見なされ、自動再生のブロックがそれらに適用されます。 **可聴でない**メディアは自動再生のブロックの影響を受けません。

## 自動再生と自動再生のブロック

用語「**自動再生**（**autoplay**）」は、ユーザーが再生の開始を明確に要求しなくても音声の再生を開始させる機能を意味します。 これには、メディアの自動再生のための HTML 属性の使用と、ユーザー入力を処理するコンテキストの外で再生を開始するための JavaScript コードのユーザーの両方が含まれます。

つまり、次の両方が自動再生のふるまいと見なされるため、ブラウザーの自動再生のブロックのポリシーに従います。

```html
<audio src="/music.mp4" autoplay>
```

と

```js
audioElement.play();
```

次のウェブ機能および API は自動再生のブロックの影響を受ける可能性があります。

- {{Glossary("HTML")}} の {{HTMLElement("audio")}} 要素と {{HTMLElement("video")}} 要素
- [ウェブオーディオ API](/ja/docs/Web/API/Web_Audio_API)（Web Audio API）

ユーザーの観点からは、警告なしに自発的にノイズを発し始めるウェブページまたはアプリは、耳障り、不便、または気まずいものになる可能性があります。 そのため、ブラウザーでは通常、特定の状況下でのみ自動再生が正常に行われるようにすることしかできません。

### 自動再生の可用性

一般的な規則として、次の条件の*少なくとも 1 つ*が当てはまる場合にのみ、メディアの自動再生が許可されると想定できます。

- 音声がミュートになっているか、音量が 0 に設定されている。
- ユーザーがサイトを操作した（クリック、タップ、キーを押すなど）。
- サイトがホワイトリストに登録されている場合。 これは、ユーザーがメディアと頻繁に関わっているとブラウザーが判断した場合は自動的に、または設定や他のユーザーインターフェイス機能を使用して手動で行われる場合があります。
- 自動再生機能ポリシーを使用して {{HTMLElement("iframe")}} とそのドキュメントに自動再生のサポートを付与する場合。

そうでないと、再生がブロックされる可能性があります。 ブロックされる正確な状況、およびサイトがホワイトリストに登録される方法の詳細はブラウザーによって異なりますが、上記のガイドラインを参考にしてください。

> **メモ:** 別の言い方をすれば、音声が含まれているメディアの再生は、ユーザー操作がまだ行われていないタブでプログラム的に再生が開始されると、通常はブロックされます。 ブラウザーはさらに他の状況下でブロックすることを選択するかもしれません。

## メディア要素の自動再生

自動再生とは何か、自動再生が許可されないようにすることができるかについて説明したので、次に、ウェブサイトまたはアプリがページの読み込み時に自動的にメディアを再生する方法、自動再生の失敗の検出方法、および自動再生がブラウザーによって拒否されたときの対処方法について説明します。

### autoplay 属性

コンテンツを自動的に再生する最も簡単な方法は、{{HTMLElement("audio")}} 要素または {{HTMLElement("video")}} 要素に {{htmlattrxref("autoplay", "audio")}} 属性を追加することです。 これにより、要素の {{domxref("HTMLMediaElement.autoplay", "autoplay")}} プロパティが `true` に設定され、`autoplay` が `true` の場合、次のことが発生した後、メディアはできるだけ早く自動的に再生を開始します。

- ページは自動再生機能を使用することを許可されている。
- 要素はページの読み込み中に作成された。
- ネットワークのパフォーマンスや帯域幅に劇的な変化がないと仮定して、再生を開始し、中断することなくメディアの最後まで再生を続けるのに十分なメディアが受信されている。

#### 例: autoplay 属性

`autoplay` 属性を使用する {{HTMLElement("audio")}} 要素は、次のようになります。

```html
<audio id="musicplayer" autoplay>
  <source src="/music/chapter1.mp4">
</audio>
```

#### 例 2: 自動再生の失敗を検出する

重要なことを自動再生に頼っている場合、または自動再生の失敗が何らかの形でアプリに影響を与える場合は、自動再生が開始されなかったことを知りたいと思うでしょう。 残念ながら、{{htmlattrxref("autoplay", "audio")}} 属性の場合、自動再生が正常に開始されたかどうかを認識するのは難しいです。 自動再生が失敗したときにトリガーされるイベントはありません。 また、設定可能な例外やコールバック、あるいは自動再生が機能したかどうかを示すフラグもメディア要素にありません。 本当にできることは、いくつかの値を調べて、自動再生が機能したかどうかについて山を張ることだけです。

見方を変えることができるのであれば、メディアの再生がうまくいかなかったときではなく、メディアの再生がうまくいったことを知ることに頼ることをお勧めします。 メディア要素で {{event("play")}} イベントが発生するのをリスンすることで、これを簡単に行うことができます。

`play` イベントは、メディアが一時停止後に再開されたとき*と*自動再生が発生したときの両方に送信されます。 つまり、初めて `play` イベントが発生したときは、ページが開かれた後に初めてメディアの再生が開始されたことがわかります。

次の HTML をメディア要素として考えます。

```html
<video src="myvideo.mp4" autoplay onplay=handleFirstPlay(event)">
```

ここでは、{{domxref("HTMLMediaElement.onplay", "onplay")}} イベントハンドラが設定された、{{htmlattrxref("autoplay", "video")}} 属性が設定されている {{HTMLElement("video")}} 要素があります。 イベントは `handleFirstPlay()` と呼ばれる関数によって処理され、この関数は入力として `play` イベントを受け取ります。

`handleFirstPlay()` は次のようになります。

```js
function handleFirstPlay(event) {
  let vid = event.target;

  vid.onplay = null;

  // 再生が開始された後に行う必要があるものをすべて開始する。
}
```

{{domxref("Event")}} オブジェクトの {{domxref("Event.target", "target")}} から動画要素への参照を取得した後、その要素の `onplay` ハンドラは `null` に設定されます。 これにより、今後の `play` イベントがハンドラに配信されなくなります。 これは、ドキュメントがバックグラウンドタブにあるときに、動画がユーザーによって一時停止および再開された場合、またはブラウザーによって自動的に行われる場合に発生する可能性があります。

この時点で、あなたのサイトやアプリはそれがする必要があるものは何でも始めることができます。

> **メモ:** この方法では、自動再生とユーザーによる手動再生開始は区別されません。

### play() メソッド

用語「自動再生」はまた、スクリプトが、ユーザ入力イベント処理のコンテキストの外側で、音声を含んだメディアの再生を開始しようと試みるシナリオを指します。 これは、メディア要素の {{domxref("HTMLMediaElement.play", "play()")}} メソッドを呼び出すことによって行われます。

> **メモ:** 自動再生設定のサポートは、自動再生の他の手段よりも `autoplay` 属性の方が広く普及しているため、できるだけ `autoplay` 属性を使用することを強くお勧めします。 また、ブラウザーが再生開始の責任を負うようにして、再生のタイミングを最適化します。

#### 例: 動画の再生

この簡単な例では、ドキュメント内の最初の {{HTMLElement("video")}} 要素を再生します。 ドキュメントに自動的にメディアを再生するパーミッションがない限り、`play()` は再生を開始させません。

```js
document.querySelector("video").play();
```

#### 例: play() のエラー処理

{{domxref("HTMLMediaElement.play", "play()")}} メソッドを使用して開始すると、メディアの自動再生の失敗を検出するのがはるかに簡単になります。 `play()` は、メディアが正常に再生を開始すると解決され、自動再生が拒否された場合など、再生が開始しないと却下される {{jsxref("Promise")}} を返します。 自動再生が失敗した場合は、あなたはおそらくメディアを再生するためのパーミッションをユーザーが与えるようにブラウザーに手動で指示する方法をユーザーに提供したいと思うでしょう。

あなたは仕事を達成するために、このようなコードを使うかもしれません。

```js
let startPlayPromise = videoElem.play();

if (startPlayPromise !== undefined) {
  startPlayPromise.catch(error => {
    if (error.name === "NotAllowedError") {
      showPlayButton(videoElem);
    } else {
      // 読み込みエラーまたは再生エラーを処理する。
    }
  }).then(() => {
    // 再生が開始された後にのみ行う必要があるものを開始する。
  });
}
```

`play()` の結果に対して最初に行うことは、それが未定義（`undefined`）ではないことを確認することです。 これは、以前のバージョンの HTML 仕様では、`play()` が値を返さなかったためです。 操作の成功または失敗を判断できるようにという promise を返すことが最近追加されました。 未定義をチェックすることで、このコードが古いバージョンのウェブブラウザーでエラーにより失敗することを防ぎます。

次に promise に {{jsxref("Promise.catch", "catch()")}} ハンドラを追加します。 これは、エラーの名前（{{domxref("DOMException.name", "name")}}）を調べて、`NotAllowedError` かどうかを確認します。 これは、自動再生が拒否されたなど、パーミッションの問題が原因で再生が失敗したことを示します。 その場合は、ユーザーが手動で再生を開始できるようにするためのユーザーインターフェイスを表示する必要があります。 これはここでは関数 `showPlayButton()` によって処理されます。

その他のエラーは適切に処理されます。

`play()` によって返された promise がエラーなしで解決された場合、`then()` 句が実行され、自動再生が開始されたときに必要なことは何でも開始できます。

## ウェブオーディオ API を使用した自動再生

[ウェブオーディオ API](/ja/docs/Web/API/Web_Audio_API) では、ウェブサイトまたはアプリは、{{domxref("AudioContext")}} にリンクされているソースノードで `start()` メソッドを使用して音声の再生を開始できます。 ユーザ入力イベント処理のコンテキストの外側でそうすることは、自動再生規則の影響を受けます。

_より多くのコンテンツがすぐに来るでしょう。 自動再生のブロックは、Mozilla でもまだ開発中です。 他の人がすでにそれを持っているならば、彼らがこのセクションに参加することを歓迎します..._

## autoplay 機能ポリシー

上記のブラウザー側での自動再生機能の管理および制御に加えて、ウェブサーバーは自動再生が機能することを許可する意欲を表現することもできます。 {{Glossary("HTTP")}} の {{HTTPHeader("Feature-Policy")}} ヘッダーの [`autoplay`](/ja/docs/Web/HTTP/Headers/Feature-Policy/autoplay) ディレクティブは、メディアの自動再生に使用できるドメインがあれば、それを制御するために使用されます。 デフォルトでは、`autoplay` 機能ポリシー（feature policy）は `'self'`（_一重引用符を含む_）に設定されています。 これは、ドキュメントと同じドメインでホストされているときに自動再生が許可されることを示します。

また、`'none'` を指定して自動再生を完全に無効にしたり、`'*'` を指定してすべてのドメインからの自動再生を許可したり、メディアを自動的に再生できる 1 つ以上の特定のオリジンを指定できます。 これらのオリジンはスペース文字で区切ります。

> **メモ:** 指定された機能ポリシーは、そのフレームとその中にネストされているすべてのフレームに新しい機能ポリシーを設定する {{htmlattrxref("allow", "iframe")}} が含まれていない限り、ドキュメントとその中にネストされているすべての {{HTMLElement("iframe")}} に適用されます。

`<iframe>` の {{htmlattrxref("allow", "iframe")}} 属性を使用してそのフレームとそのネストされたフレームの機能ポリシーを指定するときは、値 `'src'` を指定して、フレームの {{htmlattrxref("src", "iframe")}} 属性で指定されたものと同じドメインからのメディアの自動再生のみを許可できます。

### 例: ドキュメントのドメインからの自動再生のみを許可する

{{HTTPHeader("Feature-Policy")}} ヘッダーを使用して、ドキュメントの{{Glossary("origin","オリジン")}}からのメディアの自動再生のみを許可するには次のようにします。

```
Feature-Policy: autoplay 'self'
```

{{HTMLElement("iframe")}} に対して同じことを行うには次のようにします。

```html
<iframe src="mediaplayer.html"
        allow="autoplay 'src'">
</iframe>
```

### 例: 自動再生と全画面モードの許可

前の例に[全画面 API](/ja/docs/Web/API/Fullscreen_API)（Fullscreen API）のパーミッションを追加すると、ドメインに関係なく全画面のアクセスが許可されている場合、次のような `Feature-Policy` ヘッダーになります。 必要に応じてドメイン制限を追加できます。

```
Feature-Policy: autoplay 'self'; fullscreen
```

`<iframe>` 要素の `allow` プロパティを使って同じパーミッションを設定すると、次のようになります。

```html
<iframe src="mediaplayer.html"
        allow="autoplay 'src'; fullscreen">
</iframe>
```

### 例: 特定のソースからの自動再生を許可する

ドキュメント（または `<iframe>`）の独自ドメインと `https://example.media` の両方からメディアを再生できるようにする `Feature-Policy` ヘッダーは、次のようになります。

```
Feature-Policy: autoplay 'self' https://example.media
```

次のように {{HTMLElement("iframe")}} を記述して、この自動再生ポリシーをそれ自体に適用する必要があり、すべての子フレームをこのように記述することを指定することができます。

```html
<iframe width="300" height="200"
        src="mediaplayer.html"
        allow="autoplay 'src' https://example.media">
</iframe>
```

### 例: 自動再生を無効にする

`autoplay` 機能ポリシーを `'none'` に設定すると、ドキュメントまたは `<iframe>` とすべてのネストされたフレームに対して自動再生が完全に無効になります。 HTTP ヘッダーは次のとおりです。

```
Feature-Policy: autoplay 'none'
```

`<iframe>` の `allow` 属性を使う場合は、次のようになります。

```html
<iframe src="mediaplayer.html"
        allow="autoplay 'none'">
</iframe>
```

## ベストプラクティス

ここでは、自動再生を最大限に活用するためのヒントと推奨されるベストプラクティスを紹介します。

### メディアコントロールを使用した自動再生失敗の処理

自動再生の一般的な使用例は、記事、広告、またはページの主な機能のプレビューに合わせてビデオクリップの再生を自動的に開始することです。 このような動画を自動再生するには、2 つの選択肢があります。 音声トラックを使用しない、または音声トラックを使用しますが、デフォルトで音声をミュートするように {{HTMLElement("video")}} 要素を次のように設定する方法です。

```html
<video src="/videos/awesomevid.webm" controls autoplay muted>
```

この動画要素は、ユーザーコントロール（通常は再生/一時停止、動画のタイムラインのスクラブ、音量調整、およびミュート）を含むように構成されています。 また、{{htmlattrxref("muted", "video")}} 属性が含まれているため、動画は自動再生されますが、音声はミュートされています。 ただし、ユーザーはコントロールのミュート解除ボタンをクリックして音声を再度有効にすることができます。

## ブラウザー設定オプション

ブラウザーには、自動再生が機能する方法や自動再生のブロックの処理方法を制御する設定があります。 ここでは、ウェブ開発者として特別な意味または重要性があるかもしれないそのような設定がリストされています。 これらには、テストやデバッグに役立つ可能性のあるものや、あなたが処理する準備を整える必要がある方法で設定できるものが含まれます。

### Firefox

- `media.allowed-to-play.enabled`
  - : {{domxref("HTMLMediaElement.allowedToPlay")}} プロパティをウェブに公開するかどうかを指定するブール設定。 これは現在デフォルトでは `false` です（デフォルトで `true` になっているナイトリービルドを除く）。 これが `false` の場合、`allowedToPlay` プロパティは `HTMLMediaElement` インターフェイスにないため、{{HTMLElement("audio")}} 要素にも {{HTMLElement("video")}} 要素にも存在しません。
- `media.autoplay.allow-extension-background-pages`
  - : このブール値設定が `true` の場合、ブラウザー拡張機能のバックグラウンドスクリプトは音声メディアを自動再生できます。 この値を `false` に設定すると、この機能は無効になります。 デフォルト値は `true` です。
- `media.autoplay.allow-muted`
  - : `true`（デフォルト）の場合、現在ミュートされている音声メディアを自動的に再生することを許可するブール設定。 これが `false` に変更された場合、音声トラックのあるメディアはミュートされていても再生が許可されません。
- `media.autoplay.block-webaudio`
  - : [ウェブオーディオ API](/ja/docs/Web/API/Web_Audio_API) に自動再生のブロックを適用するかどうかを示すブール設定。 デフォルトは `true` です。
- `media.autoplay.default`
  - : デフォルトで自動再生サポートのドメインごとの設定を許可する（`0`）、ブロックする（`1`）、使用時のプロンプト（`2`）のどちらにするかを指定する整数設定。 デフォルト値は `0` です。
- `media.autoplay.enabled.user-gestures-needed`（ナイトリービルドのみ）
  - : ユーザーのジェスチャーの検出によって `media.autoplay.default` の設定をオーバーライドできるかどうかを制御するブール設定。 `media.autoplay.default` が `0`（デフォルトで自動再生が許可）に設定されて*いない*場合、この設定が `true` の場合、ページがユーザーのジェスチャーによってアクティブにされ、可聴でないメディアはまったく制限されていない場合に、音声トラック付きのメディアを自動再生できます。
- `media.block-autoplay-until-in-foreground`
  - : バックグラウンドタブで開始したときにメディアの再生がブロックされるかどうかを示すブール設定。 デフォルト値の `true` は、他の方法で利用できる場合でも、タブが前面に表示されるまで自動再生は実行されないことを意味します。 これは、タブが音を出し始めて、ユーザーがすべてのタブやウィンドウの中からそのタブを見つけることができないという煩わしい状況を防ぎます。

## 関連情報

- [ウェブメディア技術](/ja/docs/Web/Media)
- [動画と音声のコンテンツ](/ja/docs/Learn/HTML/Multimedia_and_embedding/Video_and_audio_content)（学習ガイド）
- [Web Audio API の使用](/ja/docs/Web/API/Web_Audio_API/Using_Web_Audio_API)
- [クロスブラウザーオーディオの基本](/ja/docs/Web/Apps/Fundamentals/Audio_and_video_delivery/Cross-browser_audio_basics)
