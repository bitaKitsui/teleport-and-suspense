---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
---

# TeleportとSuspenseについて

---

# Agenda

- Teleport
- Suspense

---

# 今日のひとこと

<div style="display: flex; gap: 40px">
  <ul>
    <li>ウォンバットのコウくんに会う！</li>
    <li>鳥刺を食べて食中毒に</li>
    <li>
      久しぶりに映画館で4本見て疲れる
      <ul style="list-style: circle">
        <li>アケルマン特集3本</li>
        <li>クレール・ドゥニ1本</li>
      </ul>
    </li>
  </ul>
  <img src="/maxresdefault.jpeg" alt="" style="width: 450px">
</div>


---

# Teleportとは？

<p style="color: white; font-size: 24px; margin-top: 20px">コンポーネントのテンプレートの一部を、そのコンポーネントのDOM階層外に存在するDOMノードに「テレポート」することができるビルトインのコンポーネントのこと</p>

---

# 基本的な使い方

- 視覚的な観点で、現在のコンポーネントの外側のDOMに表示させたい場合
- 一般的な例
  - モーダルを構築するとき
    - モーダルを表示させるボタンとモーダル自体は同じコンポーネント内に存在するのが理想的
    - しかし、モーダルがボタンと一緒にレンダリングされ、アプリケーションのDOM階層に深くネストされることになる
    - つまり、CSSの動作に影響を及ぼすことがある（`position: fixed`や`z-index`など）
- Teleportは、ネストされたDOM構造から抜け出すことが可能になる

---

# 基本的な使い方

以下のようなHTML構造

```html
<div class="outer">
  <h3>Vue Teleport Example</h3>
  <div>
    <MyModal />
  </div>
</div>
```

---

# 基本的な使い方

`<MyModal>` コンポーネントの中身（scriptタグ割愛）

```vue
<template>
  <button @click="open = true">Open Modal</button>

  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  z-index: 999;
  top: 20%;
  left: 50%;
  width: 300px;
  margin-left: -150px;
}
</style>
```

---

# 基本的な使い方

Teleportを使用した例

```html
<button @click="open = true">Open Modal</button>

<Teleport to="body">
  <div v-if="open" class="modal">
    <p>Hello from the modal!</p>
    <button @click="open = false">Close</button>
  </div>
</Teleport>
```

---

# コンポーネントと一緒に使う

- TeleportはレンダリングされたDOM構造を変更するだけで、コンポーネントの論理階層には影響を与えない
- つまり、Teleportがコンポーネントを含む場合、そのコンポーネントはTeloportを含む親コンポーネントの論理的な子のままとなる
- したがって、propsの受け渡しやイベントの発生などは、引き続き同じように機能する

---

# Teleportの無効化

- デスクトップではオーバーレイとしてレンダリングし、モバイルではインラインでレンダリングしたい場合など
- `disabled` 属性を付けることで可能

```html
<Teleport :disabled="isMobile">
  ...
</Teleport>
```

---

# 複数のTeleport

- 特に順番やkeyの指定は不要で、上から順番にレンダリングされていく

```html
<Teleport to="#modals">
  <div>A</div>
</Teleport>
<Teleport to="#modals">
  <div>B</div>
</Teleport>
```

---

# Suspenseとは？

- 2022年5月時点では実験的機能
- Suspenseはコンポーネントツリー内の非同期依存関係をオーケストレーションするためのビルトインコンポーネント
- コンポーネントツリーの下にネストされた複数の非同期依存関係が解決されるのを待つ間、ロード状態をレンダリングすることができる

---

# 非同期依存関係

```
<Suspense>
└─ <Dashboard>
   ├─ <Profile>
   │  └─ <FriendStatus> (component with async setup())
   └─ <Content>
      ├─ <ActivityFeed> (async component)
      └─ <Stats> (async component)
```

- 何らかの非同期リソースに依存する複数のネストされたコンポーネントがある場合
- ネストされたコンポーネントでは、それぞれが独自のロードやエラー、ロード状態を処理する必要がある
- 最悪の場合、複数のロードスピナーが表示される場合も

---

# 非同期依存関係

- Suspenseコンポーネントでは、これらのネストされた非同期依存関係が解決されるのを待つ間、トップレベルのロードやエラー状態を表示する機能を提供する
  - `setup()` が使用されているコンポーネント
  - 非同期コンポーネント

---

# async setup()

- Composition APIで `setup()` を使う場合

```ts
export default {
  async setup() {
    const res = await fetch(...)
    const posts = await res.json()
    return {
      posts
    }
  }
}
```

---

# async setup()

- `<script setup>` 構文の場合は、トップレベルで `await` が使える

```vue
<script setup>
const res = await fetch(...)
const posts = await res.json()
</script>

<template>
  {{ posts }}
</template>
```

---

# 非同期コンポーネント

- 非同期コンポーネントの場合は、デフォルトでSuspenseに対する非同期依存と扱われる
- 読み込みの状態はSuspenseによって制御され、コンポーネント自身の読み込み、エラー、遅延、タイムアウトのオプションは無視される

---

# Loading State

- Suspenseには2つのスロットがある
  - #defaultと#fallback
  - どちらも直下の子ノードを1つだけ許容する

```html
<Suspense>
  <!-- component with nested async dependencies -->
  <Dashboard />

  <!-- loading state via #fallback slot -->
  <template #fallback>
    Loading...
  </template>
</Suspense>
```

---

# Loading State

- 最初のレンダリングで、Suspenseはデフォルトのスロットの内容をメモリ上にレンダリングする
- この処理中に非同期の依存関係が発生した場合、pending状態になる
- pending状態には#fallbackが表示される
- 全て解決されると#defaultが表示される
- 最初のレンダリング時に非同期が発生しない場合、解決された状態に移行する

---

# Events

- pending, resolve, fallbackの3つのeventを発生させる
  - pending: 保留状態になったときに発生
  - resolve: 解決し終わったときに発生
  - fallback: fallbackスロットのコンテンツが表示されたときに発生
- 新しいコンポーネントがロードされている間、スピナーを表示するために使用できる

---

# Error Handling

- Suspenseには現在、コンポーネント自身によるエラー処理の機能を提供していない
- `errorCaptured`　オプションまたは、 `onErrorCaptured()` フックを使用すると、Suspenseの親コンポーネントの非同期エラーを補足して処理することができる

---

# 他のコンポーネントとの組み合わせ

- SuspenseはTransitionとKeepAliveのコンポーネントと組み合わせて使うのが一般的
- 他にはVue Routerの `<RouterView>` と組み合わせる