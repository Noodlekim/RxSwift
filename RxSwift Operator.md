# Rx Operator（作成中）
参考：[ReactiveX](http://reactivex.io/documentation/operators.html)
> 購読: Subscribe

## 作成目標
- Operator概要
- 例
- 画像

## Observable生成
Observable生成と関わっているOperator一覧

#### Create
onNext, onErrorで通信する
  ```
例）Observable.create
```

#### Defer
  - Subscribeする前までObservebleを返却しない
  - 条件によるObservableを生成する時便利
  - 例） // TODO: 例を追加

#### Empty, Never, Error
  - Empty: onCompletedになる
    - flatMapでそのままonCompletedさせる時使われる
  - Never: onDisposeになる
  - Error: onErrorになる
    - flatMapでErrorをvalueで持つ別のObservableを返す時使われる
    ```
      例）Observable.empty(), Observable.never(), Observable.error()
    ```

#### From
- 購読時Observableの配列の中の値単位で処理する
   - 本来ならArrayそのままOnNextから受け取る
```
  例） Observable.from([1,2,3])
```

#### Interval
  - タイマーのような概念
    - Observableの購読繰り繰り返す
    - .take, .flatMap, .takeWhileなどと一緒によく使われる
    ```
      例）TODO
    ```

#### Just
  - 一つの値または配列をObservableで返却

#### Range
  - 連続項目を返すObservableを生成
    - start, count를 지정해서 카운트된 것 만큼 반복해줌.
        - 지정된 옵션에 따라서 +1씩 순차적으로 for문을 돌리는 것 같음.

#### Repeat
  - Observable生成時指定した値を繰り返しながら購読ができる
    - take, takeWhileとセットで使う必要がある

#### StartWith
    - 생성사 Operator이긴 한데…
    - subscribe시 처음에 생성된 Observable의 value보다 이녀석이 먼저 들어오도록 함.
    - 그렇다고 원본 Observable의 value가 무시 되는건 아님..
        - 끼워넣기 격
    - 반환가능한 형태는 처음 생성시 지정한 타입만 가능한 듯

- Timer
    - 지정된 시간이 지난후 항목을 배출함.
    - playground에서 테스트가 안됨.
    - 최초에 timer형태로 생성 + 값들은 flatMap등을 이용을 해서 처리를 해줘야 할 것 같다
    - 단순히 딜레이를 주는 녀석 같음.
    - 최초 Observable를 생성시 timer를 지정?
        -     let observable = Observable<Int>.timer(3, scheduler: MainScheduler.instance)
        -     observable
        -         .flatMap({ (value) -> Observable<Int> in
        -             return Observable<Int>.of(1,2,3)
        -         })
        -         .subscribe(onNext: { (i) in
        -             print(i)
        -         }, onCompleted: {
        -             print("Completed")
        -         })


Observable 변환

- Buffer
    - 복수의 Observable로 부터 buffer의 조건을 충족시키면 subscribe하는 구조같음.
    - Observable로부터 정기적으로 항목들을 수집하고 묶음으로 만든 후에 묶음 안에 있는 항목들을 한번에 하나씩 배출하지 않고 수집된 묶음 단위로 배출한다 <- 라고 되어있는데.. 잘 모르겠다. 그냥 interval같음.
    - ???

- FlatMap
    - 하나의 Observable이 발행하는 항목들을 여러개의 Observable로 변환하고, 항목들의 배출을 차례차례 줄 세워 하나의 Observable로 전달한다
    - 다른 Observable 형태로 반환. 허용이 되는게 변환계까지만 되는지 아니면 필터링계도 포함하는지 확인해야할 필요성이 있음.
    - 아마 제일 많이 쓰이지 않을까 싶은 녀석..
        - Observable의 결과를 다른 Observable로 변환해서 사용하기도 하고..
            - 예) fetch후 Error핸들링 + JSON값들을 가공 + map등으로 가공해서 처리를 한다던지…

- GroupBy
    - 원본 Observable이 배출하는 항목들을 키(Key) 별로 묶은 후 Observable에 담는다. 이렇게 키 별로 만들어진 Observable들은 자기가 담고 있는 묶음의 항목들을 배출한다
    - 복수의 Observable를 하나의 Observable에 담아두면서 그룹을 지정을 하면 그룹별로 key, value식으로 분리되어서 subscribe를 할 수 있다??

- Map
    - Observable이 배출한 항목에 함수를 적용한다
    - subscribe의 value의 값을 바꾸고 싶을 때 사용.

- Scan
    - 변환계로 배열의 값을 합할때 자주 쓰임.
    - 예를 들어서
        -     let source = Observable<Int>.of(1,3,5,7,9)    
        -     let observable = source.scan(0, accumulator: +)
        -     observable.subscribe(onNext: { (value) in
        -         print(value)
        -     })
    - 2개의 Observable를 합하는것도 있는 것 같은데 잘 모르겠다??
    -     let stream = Observable.of(0, 1, 2, 3)
    -     _ = stream
    -         .scan(0) { acum, elem in
    -             acum + elem
    -         }
    -         .subscribe {
    -             print($0)
    -     }


- Window
    - 정기적으로 Observable의 항목들을 더 작은 단위의 Observable 윈도우로 나눈 후에, 한번에 하나씩 항목들을 발행하는 대신 작게 나눠진 윈도우 단위로 항목들을 배출한다
    - 특정 Observable포를 한다.의 복수의 요소를 그룹핑해서 별도의 Observable로 배출함.
        - 어디에서 쓰면 좋을지는 잘 모르겠음.
    - 예
    - example(of: "window") {
    -     let bag = DisposeBag()
    -     let observable = Observable.of(0,1,2,3,4,5,6,7,8)
    -     
    -     observable
    -         .window(timeSpan: 1000, count: 2, scheduler: MainScheduler.instance)
    -         .subscribe(onNext: { (observable) in
    -             observable
    -                 .subscribe({ (event) in
    -                     print("event \(event)")
    -                 })
    -                 .addDisposableTo(bag)
    -         }, onError: { (error) in
    -             print("error \(error)")
    -         }, onCompleted: {
    -             print("onCompleted")
    -         }, onDisposed: {
    -             print("onDisposed")
    -         })
    -         .addDisposableTo(bag)
    - }
    -


Observable필터링

- Debounce/ Throttle
    - Observable의 시간 흐름이 지속되는 상태에서 다른 항목들은 배출하지 않고 특정 시간 마다 그 시점에 존재하는 항목 하나를 Observable로부터 배출한다
    - throttle처럼 일정시간 시그널을 무시함.
        - 단, 처음꺼는 반응함.
    - throttle의 경우에는 옵션으로 latest를 줄 수가 있음.
        - 이거 true랑 false일 경우 뭐가 다르지? 테스트해봤는데 뭐가 다른지 모르겠음.

- Distinct
    - Observable이 배출하는 항목들 중 중복을 제거한 항목들을 배출한다
    - RxSwift는 distinctUntilChanged 임

- ElementAt
    - n번째 항목을 배출

- Filter
    - 우리가 아는 필터

- First
    - RxSwift에서는 single()이 나옴.

- IgnoreElements
    - 항목들을 배출하지는 않고 종료 알림은 보낸다
    -  onCompleted랑 onDisposed만 출력

- Last
    - 마지막 항목만 출력
    - 이건 존재를 안함.
        - takeLast, take같은건 있음.

- Sample
    - 특정 시간 간격으로 최근에 Observable이 배출한 항목들을 배출한다

- Skip
    - n개의 항목을 숨김

- SkipLast
    - n개의 항목을 뒤에서 부터 숨김
    - takeLast랑 반대

- Take
    - n개의 항목만 배출

- TakeLast
    - 마지막에서 n개의 항목막 배출


Observables 결함

- And/ Then/ When
    - RxSwift에는 없는 듯..

- CombineLatest
    - 두 개의 Observable 중 하나가 항목을 배출할 때 배출된 마지막 항목과 다른 한 Observable이 배출한 항목을 결합한 후 함수를 적용하여 실행 후 실행된 결과를 배출한다
    - 갯수가 다르면 가장 마지막게 반영이 됨.
    - PublicSubject등 탸입이 다른 녀석이면 에러가남.
    - 예)
    -     let observable = Observable.combineLatest([left, right], { (strings) -> String in        
    -         strings.joined(separator: "|||")
    -     })
    - > 2개의 Observable 가 onNext가 되어야 동작하도록 할 수 있다.
    - take같은 걸로 제한을 걸어두는 것도 좋아보임.
        - 아니면 바로 dispose를 하는 것도 좋아보임.

- Join(RxSwift에는 없음)
    - A Observable과 B Observable이 배출한 항목들을 결합하는데, 이때 B Observable은 배출한 항목이 타임 윈도우를 가지고 있고 이 타임 윈도우가 열린 동안 A Observable은 항목의 배출을 계속한다. Join 연산자는 B Observable의 항목을 배출하고 배출된 항목은 타임 윈도우를 시작시킨다. 타임 윈도우가 열려 있는 동안 A Observable은 자신의 항목들을 계속 배출하여 이 두 항목들을 결합한다
    -


- Merge
    - 복수 개의 Observable들이 배출하는 항목들을 머지시켜 하나의 Observable로 만든다
    - merge된 어떤 Observable 든지.. subscribe에 반응을 함.
    - 그렇다면? 2개의

- Zip
    - 명시한 함수를 통해 여러 Observable들이 배출한 항목들을 결합하고 함수의 실행 결과를 배출한다
    - 각각의 Observable의 요소들을 차례대로 받음. 이걸 튜플같은걸로 반환을 해서 사용하고 그럼.
    - 쌍으로 onNext가 안되면 subscribe가 반응을 안함.
    - 에러가 발생하면 onCompleted가 되어버려서 더이상 반응을 안함.



오류처리 연산자

- Catch
    - RxSwift에서는 catchError, catchErrorJustReturn
    - catchError
        - onError같은걸로 에러가 반환이 되었을 때 onError가 아니라 그전에 캐취를 해서
            - subscribe 쪽에 에러에 따른 default값을 전달할 때 용이하다고 생각됨.
    - 일단 Error가 되면


- Retry
    - 만약 소스 Observable이 'onError' 알림을 보낼 경우, 오류 없이 실행이 완료되기를 기대하며 재구독을 시도한다
        - 보통 Error가 되면 재구독이 안되는데 이 경우에는 n수만큼 재구독을 시키는 듯
    - 에러가 났을 때 정해놓은 횟수만큼 재구독을 함.
        - 로그인 실패시등 사용하면 좋음.
    - 예)
    - observer
    -     .delay(1.0, scheduler: MainScheduler.instance)
    -     .retry(3)
    -     .subscribe(onNext: { event in
    -         print("Receive event: \(event)")
    -     })
    -


Observable 유틸리티 연산자
> Observable과 함께 동작하는 도우미 연산자들

- Delay
    - 정해놓은 시간동안 Delay시킴

- Do
    - Observable의 생명주기 동안 발생하는 여러 이벤트에서 실행 될 액션을 등록한다
    - subscribe보다 더 상세한 이벤트를 체크를 할 수 있다.
        - 말그대로 도우미 연산자인듯… 안써도 될 듯

- Materialize/Dematerialize  
    - RxSwift 에는 없음.

- ObserveOn
    - 옵저버가 어느 스케줄러 상에서 Observable을 관찰할지 명시한다
    - 보통 해당 Observable를 무슨 스레드에서 돌릴 건지 정하는데..
백그라운드에서 돌리는 것도 가능한듯..
    - subscribeOn이랑 비슷하게 쓰이는데 이거 2개를 명확하게 어떻게 쓸지 감은 잘 안옴??
    - http://qiita.com/gomi_ningen/items/37cfd4e04fbc0483e23d

- Serialize
    - Observable이 직렬화된 호출을 생성하고 제대로 동작하도록 강제한
    - RxSwift에는 없음.

- Subscribe
    - Observable이 배출하는 항목과 알림을 기반으로 동작한다

- SubscribeOn
    - Observable을 구독할 때 사용할 스케줄러를 명시한다
    - UI쪽 갱신할 땐 반드시 이걸 MainScheduler로 해야하나?

- TimeInterval
    - 항목들을 배출하는 Observable을, 항목을 배출하는데 걸린 시간이 얼마인지를 가리키는 Observable로 변환한다
    - RxSwift에는 없음.

- TimeOut
    - 소스 Obvservable을 그대로 전달하지만, 대신 특정 시간 동안 배출된 항목이 없으면 오류 알림을 보낸다
    - 아마 정기적으로 배출을 하는 Observable에 적합한 듯..
        - 일정시간 동안 반응이 안오면 타임아웃을 보냄.
    - 예를 들어서 fetch()쪽 매소드의 Observable에 timeout을 걸어두면
        - 실제로 onNext가 시간내 안이뤄지면 timeout으로 에러처리가 되는 듯

- Timestamp
    - Observable이 배출한 항목에 타임 스탬프를 추가한다

- Using
    - 소스 Observable과 동일한 생명주기를 갖는 Observable을 생성하는데, 이 Observable은 생명주기가 완료되면 리소스를 종료하고 반환한다
    - RxSwift 에도 있다고는 하는데… 잘은 모르겠음.



조건과 불리언 연산자

- All
    -  Observable이 배출한 전체 항목들이 어떤 조건을 만족시키는지 판단한다
    - RxSwift에는 없음

- Amb
    - 두 개 이상의 소스 Observable이 주어 질때, 그 중 첫 번째로 항목을 배출한 Observable이 배출하는 항목들을 전달한다

- Contains
    - RxSwift에는 없음

- DefaultIfEmpty
    - 소스 Observable이 배출하는 항목을 전달한다. 만약 배출되는 항목이 없으면 기본 항목을 배출한다
    - RxSwift에는 없음

- SequenceEqual
    - 두 개의 Observable이 항목을 같은 순서로 배출하는지 판단한다
    - RxSwift에는 없음

- SkipUntil
    - 두 번째 Observable이 항목을 배출하기 전까지 배출된 항목들을 버린다
    - 두번째 Observable이 배출되는 시점까지 다 버리고, 그이후 onNext에 대해서 반응을 함.

- SkipWhile
    - 특정 조건이 false를 리턴하기 전까지 Observable이 배출한 항목들을 버린다

- TakeUntil
    - 두 번째 Observable이 항목을 발행하기 시작햤거나 두 번째 Observable이 종료되면 그 때부터 발행되는 항목들은 버린다

- TakeWhile
    -  특정 조건이 false를 리턴하기 시작하면 그 이후에 배출되는 항목들을 버린다


수학과 집계 연산자

- Average
    - Observable이 발행한 항목의 평균 값을 발행한다
    - RxSwift에는 없음
- Concat
    - 두 개 이상의 Observable들이 항목을 발행할 때 Observable 순서대로 배출하는 항목들을 하나의 Observable 배출로 연이어 배출한다
    - Observable안에 든 배열의 요소들을 하나로 합쳐줌.
    -
- Count
    - 소스 Observable이 발행한 항목의 개수를 배출한다
- Max Observable이 발행한 항목 중 값이 가장 큰 항목을 배출한다
    - RxSwift에는 없음
- Min Observable이 발행한 항목 중 값이 가장 작은 항목을 배출한다
    - RxSwift에는 없음
- Reduce
    - Observable이 배출한 항목에 함수를 순서대로 적용하고 함수를 연산한 후 최종 결과를 발행한다
    - Observable의 요소들을 합산해줌. 숫자 말고도 합산이 가능한지는 모르겠음??
- Sum
    - Observable이 배출한 항목의 합계를 배출한다
    - RxSwift에는 없음


역압(Backpressure) 연산자??
> backpressure operators — 옵저버가 소비하는 것보다 더 빠르게 항목들을 생산하는 Observable을 복재하는 전략



연결 가능한 Observable 연산자

- Connect
    - 구독자가 항목 배출을 시작할 수 있도록 연결 가능한 Observable에게 명령을 내린다
    - RxSwift에는 없음
- Publish
    - 일반 Observable을 연결 가능한 Observable로 변환한다
- RefCount
    - 일반 Observable처럼 동작하는 연결 가능한 Observable을 만든다
    - replay같은 연산자의 return치를 보면 ConnectableObservable<Self.E> 가 되어 있는데..
        - 이것을 일반 observable로 만들어줌?
    -
Replay — 비록 옵저버가 Observable이 항목 배출을 시작한 후에 구독을 했다 하더라도 배출된 모든 항목들을 볼 수 있도록 한다




Observable 변환 연산자

- To
    - toArray
        - Observable안의 요소들을 하나의 배열로 만들어줌.


Observable 연산자 결정 트리
> 어떤 경우에 Operator를 선택하면 될지..



その他
- NotificationCenter
    - NotificationCenter.rx.notification()
- KVO
    - Variable？




의문점

- [ ] cold vs hot
> cold는 subscribe까지 동작을 안함.

replay, shareReplay의 차이?
dispose와 complete의 차이?
- complete나 Error의 경우 해당 Observable는 완결이 된다고 함.
    - 이경우엔 해제가 되어서 메모리릭이 안되지만…
- 완결이 안되는 경우?엔 메모리릭의 원인이 되는 것 같음.
- 그럼 Variable를 asObservable()형태로 데이터 갱신을 처리하는 경우?
    - set value가 되는 순간 해당 subscribe는 더이상 안움직어야 하는거 아닌가?
        - 신기하게도 Variable의 경우에는 subscribe는 반응은 하지만 onComplete는 반응을 안함.
        - 관련 링크 http://tiny-wing.hatenablog.com/entry/2016/01/11/172915ㅁ.
            - onComplete나 onError가 존재를 안한다고 하

http://univ.peraichi.com/73
https://matome.naver.jp/odai/2137103611420070601?page=2


변환계랑 필터계의 차이?
- 예를 들어서 flatMap에서 어디까지 가능한지?

    public func flatMap<O: ObservableConvertibleType>(_ selector: @escaping (E) throws -> O)
        -> Observable<O.E> {
        return FlatMap(source: asObservable(), selector: selector)
    }

    public func filter(_ predicate: @escaping (Self.E) throws -> Bool) -> RxSwift.Observable<Self.E>


Delay의 사용방법?
- http://qiita.com/t-ae/items/8cbc3ba2baece14279f2

multiCase
publish
replay
replayAll
같이 ConnectableObservable 를 반환하는 녀석은 뭐지?


weak, unowned를 사용하는 경우.


Disposing with RxCocoa
The last topic of this chapter goes beyond the project and is pure theory. As explained at the beginning of the chapter, there’s a bag inside the main view controller that takes care of disposing all the subscriptions when the view controller is released. But in this example, there’s no usage of weak or unowned in all closures. Why?
The answer is simple: this application is a single view controller and the main view controller is always on screen while the application is running — so there’s no need to guard against retain cycles or wasted memory.
unowned vs weak with RxCocoa
When dealing with RxCocoa or RxSwift with Cocoa, it might be hard to understand when to use weak or unowned. You’d use weak when a closure can be called at some point in the future when the current self object has already been released. For this reason, self becomes an Optional. unowned is used to avoid the Optional self. But the code has to be sure the object will never be released before the closure gets called — otherwise, the app will crash.
In RxSwift – and especially with RxCocoa – there are some good guidelines to follow when choosing to use weak, unowned or nothing at all:
• nothing: Inside singletons or a view controller which are never released (e.g. the root view controller).
• unowned: Inside all view controllers which are released after the closure task is performed.
• weak: Any other case.
These rules prevent against the classic EXC_BAD_ACCESS error. If you always respect these rules, it’s unlikely you will have any trouble with memory management. And if you want to be extra safe, the raywenderlich.com Swift Guidelines https:// github.com/raywenderlich/swift-style-guide#extending-object-lifetime recommend against using unowned at all.


그리고 GitHub에도 어느정도 가이드라인이 있음.
https://github.com/raywenderlich/swift-style-guide#extending-object-lifetime





Alert
