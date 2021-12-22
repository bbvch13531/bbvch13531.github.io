---
title: RxSwift error handling in network request
date: 2021-12-23 00:29:14
tags: [RxSwift]
---


Network Request 요청하는 기능은 iOS앱 개발할 때 항상 구현하는 아주 당연한 기능이다.

내가 사용하는 일반적인 NetworkService Class는 다음과 같다.

```swift
// NetworkService.swift

protocol NetworkServiceType {
    func requestSomething(completion: @escaping (Result<MyData, NetworkError>) -> Void)
}

class NetworkService {
    func requestSomething(completion: @escaping (Result<MyData, NetworkError>) -> Void)
        let url = "someApiEndpointURL"
        AF.request(url, method: .get)
            .validate()
            .responseData { response in 
            switch response.result {
                case .success(let data):
                    let decoder = JSONDecoder()
                    guard let data = data,
                        let myData = try? decoder.decode(MyData.self, from: data) else {
                        completion(.failure(NetworkError.JSONParseError))
                    }
                    completion(.success(myData))

                case .failure(let error):
                    completion(.failure(NetworkError.InvalidResponse))
                }
            }
    }
}
```

`ViewController`의 `viewDidLoad()`에서 호출한다고 가정했을 때 
```swift
// ViewController.swift

override func viewDidLoad() {
    super.viewDidLoad()
    let service = NetworkService()

    service.requestSomething { result in 
        switch result {
            case .success(let myData):
                print(myData)
            case .failure(let error):
                print(error)
        }
    }
}
```

이 코드를 MVVM과 RxSwift를 이용해 개선하는 과정에서 에러처리가 의도한 대로 동작하지 않아 시행착오를 겪었다.

개선한 구조는 다음과 같다.


```swift
// ViewController.swift

class ViewController: UIViewController {
    let viewModel = ViewModel()
    let disposeBag = DisposeBag()
    let button: UIButton = {
        let button = UIButton(frame: .zero)
        button.setTitle("request", for: .normal)
        return button
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bind()
    }

    func bind() {
        button.rx.tap
            .flatMapLatest { [weak self] _ -> Observable<MyData> in
                guard let ss = self else { return Observable.empty() }
                return ss.viewModel.service.request()
                    .do(onError: { error in print("error \(error) in flatMapLatest")})
                    .catch { _ in return Observable.empty() }
            }
            .subscribe(onNext: { myData in
                print(myData)
            }, onError: { error in
                print("onError \(error)")
            }, onCompleted: {
                print("onCompleted")
            }, onDisposed: {
                print("onDisposed")
            })
            .disposed(by: self.disposeBag)
    }
}
```

```swift
// ViewModel.swift

class ViewModel {
    let service = NetworkService()
}
```

```swift
// NetworkService.swift

 func request() -> Observable<MyData> {
        return Observable.create { observer in
            
            AF.request(url)
                .responseData { response in
                    switch response.result {
                    case .success(let data):
                        let decoder = JSONDecoder()
                        guard let decodedData = try? decoder.decode(MyData.self, from: data) else {
                            observer.onError(NetworkError.JSONParseError)
                            return
                        }
                        observer.onNext(decodedData)
                    case .failure(_):
                        observer.onError(NetworkError.InvalidResponse)
                    }
                    observer.onCompleted()
                }
            return Disposables.create()
        }
    }
```

이번 글에서 말하고 싶은 부분은 `ViewController.swift`의 `bind()`의 코드다.

```swift
// ViewController.swift

button.rx.tap
    .flatMapLatest { [weak self] _ -> Observable<MyData> in
        guard let ss = self else { return Observable.empty() }
        return ss.viewModel.service.request()
            .do(onError: { error in print("error \(error) in flatMapLatest")})
            .catch { _ in return Observable.empty() }
    }
    .subscribe(onNext: { myData in
        print(myData)
    }, onError: { error in
        print("onError \(error)")
    }, onCompleted: {
        print("onCompleted")
    }, onDisposed: {
        print("onDisposed")
    })
    .disposed(by: self.disposeBag)
```

button의 tap 이벤트 마다 `viewModel.service.request()`를 실행하고 그 결과를 `flatMapLatest`에서 리턴한다.

`flatMapLatest`는 RxSwift의 공식 문서를 참조

`Observable의 element마다 새로운 Observable를 생성하고, 생성된 여러개의 새로운 시퀀스를 하나의 시퀀스로 합쳐준다.`
버튼 탭 이벤트가 여러번 발생하고, 그 이벤트마다 `Observable<MyData>`를 생성한다.
이때 각각의 `Observable<MyData>`를 하나의 시퀀스로 합쳐주는 역할을 한다.


`flatMapLatest`는 network response를 리턴한다. 여기서 에러가 발생하면 `.do(onError:)` 를 실행하고 `catch()` 에서 `Observable.empty()`를 리턴한다.
에러가 발생하지 않는 경우에는 `.subscribe(onNext:)`에서 print(myData)를 실행한다.

기존에는 에러 핸들링을 하기 위해 `catch()` 에서 `Observable.empty()`를 리턴했다. 하지만 이 경우에도 시퀀스는 종료됨을 새로 알게 되었다. (completed, disposed 되지는 않았다.)
네트워크가 실패한 경우 종료된 시퀀스에 다시 바인딩이 필요했다. request 요청 후 에러가 발생할 때마다 매번 다시 바인딩하는건 비효율적이라고 생각되어 개선했다.

button의 tap 이벤트를 `tableView`의 `refreshControl`이나, `scrollEvent`으로 활용할 수 있을 것 같다.