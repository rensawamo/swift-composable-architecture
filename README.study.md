# TCA アーキテクチャのキャッチアップ

### 主な特徴

state: あなたのアプリの機能がロジックを実行し、UI を徹底的にするために必要なデータを認識する型。
→ 　変数の状態を保持する

action: ユーザーのアクション、通知、イベントソースなど、アプリの機能で限りなく全てのアクションを表す型。
→ 関数のビジネスロジックを定義する

Reducer : Action が与えられたときに、アプリの現在の状態を次の状態に進化させる方法を定義する関数。Reducer は API リクエストのような実行すべき効果を返す役割も担っています、を Effectreturn することでそれを実行。
→ case で　 action を受け取り、state を更新する。

Store : 実際にアプリの機能を動かす runtime。全てのユーザーアクションを Store に送信し、Store は Reducer と Effect を実行できるように、Store の状態変化を観察して UI を更新できます。

→ riverpod の Provider と同じような役割。

###　 ex1

```sh
import ComposableArchitecture

@Reducer
struct CounterFeature {
  @ObservableState
  struct State {
    var count = 0
  }

  enum Action {
    case decrementButtonTapped
    case incrementButtonTapped
  }

  var body: some ReducerOf<Self> {
    Reduce { state, action in
      switch action {
      case .decrementButtonTapped:
        state.count -= 1
        return .none

      case .incrementButtonTapped:
        state.count += 1
        return .none
      }
    }
  }
}
```

state で状態を保持し、action でビジネスロジックを定義する。それを reducer で受け取り、state を更新する。

### SwiftUI での利用

```sh
struct CounterView: View {
  let store: StoreOf<CounterFeature>

  var body: some View {
    VStack {
      Text("\(store.count)")
        .font(.largeTitle)
        .padding()
        .background(Color.black.opacity(0.1))
        .cornerRadius(10)
```

store を定義して、state を取得する。

###　　機能内で外部と通信し、外部からのデータを機能にフィードする方法

外部の api との通信を担う場合、 Action に Effect を追加することで、外部のデータを取得し、それを元に state を更新することができる。

```sh

 enum Action {
    case decrementButtonTapped
    case factButtonTapped
    case incrementButtonTapped   ← 追加
  }

...

#　ボタンを押した時の関数の種類をActionで定義したので　処理を与える
 case .decrementButtonTapped:
        state.count -= 1
        state.fact = nil
        return .none

      case .factButtonTapped:
        state.fact = nil
        state.isLoading = true

        let (data, _) = try await URLSession.shared
          .data(from: URL(string: "http://numbersapi.com/\(state.count)")!)
        // 🛑 'async' call in a function that does not support concurrency
        // 🛑 Errors thrown from here are not handled

        state.fact = String(decoding: data, as: UTF8.self)
        state.isLoading = false

        return .none
```



### ref.watchみたいなやつ
```sh
  case .toggleTimerButtonTapped:
        state.isTimerRunning.toggle()
        return .run { send in
        }
      }
    }
```
.runでタイマーなど連続して実行する処理を行うことができる。


