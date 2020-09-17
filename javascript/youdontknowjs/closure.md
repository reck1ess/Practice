# 클로저

클로저는 렉시컬 스코프에 의존해 코드를 작성한 결과로 그냥 발생한다. 이용하려고 굳이 의도적으로 클로저를 생성할 필요도 없다. 모든 코드에서 클로저는 생성되고 사용된다.

## 핵심

클로저는 함수가 속한 렉시컬 스코프를 기억하여 함수가 렉시컬 스코프 밖에서 실행될 때에도 이 스코프에 접근할 수 있게 하는 기능을 뜻한다.

```javascript
function foo() {
    var a = 2;
    function bar() {
        console.log(a); // 2
    }
    bar();
}

foo();
```

위 코드는 중첩 스코프 예제와 비슷하다. 함수 bar\(\)는 렉시컬 스코프 검색 규칙을 통해 바깥 스코프의 변수 a에 접근할 수 있다.

다시 말해, bar\(\)는 foo\(\) 스코프에서 닫힌다. 이는 bar\(\)가 중첩되어 foo\(\) 안에 존재하기 때문이다.

```javascript
function foo() {
    var a = 2;
    function bar() {
        console.log(a);
    }
    return bar;
}

var baz = foo();
baz(); // 2
```

1. 함수 bar\(\)는 foo\(\)의 렉시컬 스코프에 접근할 수 있고, bar\(\) 함수 자체를 값으로 넘긴다. 이 코드는 bar를 참조하는 함수 객체 자체를 반환한다.
2. foo\(\)를 실행하여 반환한 값을 baz라는 변수에 대입하고 실제로는 baz\(\) 함수를 호출했다. 이는 그저 다른 확인자 참조로 내부 함수인 bar\(\)를 호출하게 된다.
3. bar\(\)는 의심할 여지없이 실행됐다. **그러나 이 경우에 함수 bar는 함수가 선언된 렉시컬 스코프 밖에서 실행됐다.**

> 일반적으로 foo\(\)가 실행된 후에는 foo\(\)의 내부 스코프가 사라졌다고 생각할 것이다. 아마 엔진이 가비지 콜렉터를 이용해 더는 사용하지 않는 메모리를 해제시킨다는 걸 알기 때문이다.
>
> 그러나 클로저의 마법이 이를 내버려두지 않는다. 사실 foo의 내부 스코프는 여전히 '사용 중'이므로 해제되지 않는다. 바로 bar\(\) 자신이 스코프를 사용 중이다.

**선언된 위치 덕에 bar\(\)는 foo\(\) 스코프에 대한 렉시컬 스코프 클로저를 가지고, foo\(\)는 bar\(\)가 나중에 참조할 수 있도록 스코프를 살려둔다. 즉, bar\(\)는 여전히 해당 스코프에 대한 참조를 가지는데, 그 참조를 바로 클로저라 부른다.**

## 반복문과 클로저

클로저를 설명하는 가장 흔하고 표준적인 사례는 for 반복문이다.

```javascript
for (var i=1; i<=5; i++) {
    setTimeout(function timer() {
        console.log(i);
    }, i * 1000);
}
```

이 코드의 목적은 1, 2, ..., 5까지 한 번에 하나씩 일 초마다 출력하는 것이지만 실제로 코드를 돌려보면 일 초마다 한 번씩 '6'만 5번 출력된다.

반복문이 처음으로 끝나는 조건이 갖춰졌을 때 i의 값은 6이다. 즉, 출력된 값은 반복문이 끝났을 때 i 값을 반영한 것이다. 또한 timeout 함수 콜백은 반복문이 끝나고 나서야 작동한다.

애초에 기대한 것과 같이 코드를 작동시키기 위해서는 반복마다 각각의 i 복제본을 '잡아'두어야 한다. 그러나 반복문 안 총 5개의 함수들은 반복마다 따로 정의됐음에도 모두 같이 글로벌 스코프 클로저를 공유해 해당 스코프 안에는 오직 하나의 i만이 존재한다. 따라서 모든 함수는 당연하게도 같은 i에 대한 참조를 공유한다.

```javascript
for (var i=1; i<=5; i++) {
    (function() {
        setTimeout(function timer() {
            console.log(i);
        }, i * 1000);
    })();
}
```

각각의 timeout 함수 콜백은 확실히 반복마다 각각의 IIFE가 생성한 자신만의 스코프를 가진다. 그러나 닫힌 스코프만으로는 부족하다. 자세히 살펴보면, IIFE는 아무것도 하지않는 빈 스코프일 뿐이니 무언가 해야한다. 각 스코프는 자체 변수가 필요하다. 즉, 반복마다 i의 값을 저장할 변수가 필요하다.

```javascript
for (var i=1; i<=5; i++) {
    (function() {
        var j = i;
        setTimeout(function timer() {
            console.log(j);
        }, j * 1000);
    })();
}
```
