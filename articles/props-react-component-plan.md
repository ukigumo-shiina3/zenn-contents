---
title: "Propsのバケツリレーを回避する方法"
emoji: "🌟"
type: "tech"
topics: ["react", "nextjs", "typescript"]
published: true
---

## はじめに

適切にコンポーネント設計ができていないと下に下 props をとにかく渡し、バケツリレーが多くなるという問題がある
あまり知られていないが、React には JSX.Element などの JSX をそのまま渡すパターンがあり、この方法でバケツリレー問題を解決できる

## JSX をそのまま渡すことで何が良くなるか

- 親から JSX を props として渡し、一階層リフトアップさせることができる
- 可読性がよくなり、バケツリレー感がなくなる
- プロパティを渡すのが少なくなるので、パフォーマンスがいい

:::details リフトアップとは
ネストされたコンポーネントから必要な要素を親コンポーネントに持ち上げ、親コンポーネントから JSX として子コンポーネントに渡す手法です。これにより、以下のメリットが得られます。

- コードの可読性向上: 子コンポーネントの内部構造を親コンポーネントから直接確認できるため、コードの理解が容易になります。
- 再利用性の向上: 共通的に使用される要素を親コンポーネントで定義することで、複数のコンポーネントで再利用することができます。
- 保守性の向上: コード変更箇所が明確になるため、変更後の影響範囲を把握しやすくなり、保守性が向上します。
  :::

## サンプルコード

```
type User = { name: string };

const Test = () => {
  const user = { name: "John" };

  return (
    <Layout
      aside={
        <Aside
          user={user}
          nav={
            <nav>
              <ul>
                {["Foo1", "Foo2", "Foo3"].map((item) => {
                  return <li key={item}>{item}</li>;
                })}
              </ul>
            </nav>
          }
        />
      }
      main={
        <Main
          header={
           <Header
              title="タイトル"
              user={user}
              tabs={
                <nav>
                <ul>
                  {["Bar1", "Bar2", "Bar3"].map((item) => {
                    return <li key={item}>{item}</li>;
                </ul>
                </nav>
              }
            />
          }
        >
        メインコンテンツ
      </Main>
      }
    />
  );
};

const Layout = (props: { aside: JSX.Element; main: JSX.Element; }) => {
  return (
    <div>
      {props.aside}
      {props.main}
    </div>
  );
};

const Aside = (props: { nav: JSX.Element; user: User }) => {
  return (
    <aside>
      {props.nav}
      {props.user.name}
    </aside>
  );
};

const Main = (props: { header: JSX.Element; children: ReactNode }) => {
  return (
    <main>
      {props.header}
      {props.children}
    </main>
  );
};

export default Test;
```

## 解説

- 構造としては Layout コンポーネントの中に Aside と Main コンポーネントがあり、Main の中に Header コンポーネントがある状態
- Test コンポーネントは Layout コンポーネントに、aside と main プロパティを渡し、Layout コンポーネントはこれらのプロパティを受け取ってレンダリングします。Aside と Main コンポーネントはそれぞれ Layout 内での役割を担っています。
- 例として`JSX.Element`の React Component を Props として渡し、`aside: JSX.Element`のように`JSX.Element` 型の値を受け取ることができ階層のリフトアップができます。

## コンポーネントに対して意識すること

- 例えば Layout コンポーネントは Layout に関するプロパティのみで良い。Layout は部品の配置するという役割なので、Header や Sidebar だけ受け取って user は受け取るべきでないという感じ
- 渡す props が少なくなるので、コンポーネントのプロパティの変更が少なくなり、レンダリングの回数が減ることでパフォーマンスがよくなる

## 最後に

バケツリレーが多発するなら状態管理ライブラリに任せたらという意見もあります。
ただしバケツリレーを回避するのは、基本的に大抵のプロジェクトでは、Redux や Context でグローバル管理しなくても、今回のように適切にコンポーネント設計と役割を意識していれば、ローカル管理だけで十分に済むと考えています。

この方法はすぐに取り入れることができ、今まで書いてたコードをリファクタリングしたいと思ってもらえたら幸いです。

最後まで読んでいただきありがとうございました！

## 参考資料

[React Component Composition Explained](https://felixgerschau.com/react-component-composition/)
