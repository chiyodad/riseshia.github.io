---
layout: post
title: "Elixir - 04: Pattern matching"
date: 2016-01-10 08:28:55 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 04: Pattern matching

## The match operator

우리는 `=`를 사용해서 변수에 값을 대입합니다.

```iex
iex> x = 1
1
iex> x
1
```

그런데 엘릭서는 사실 `=`가 대입이 아닌, **매칭 연산자**입니다. 무슨 의미냐면,

```iex
iex> x = 1
1
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

위처럼 `1 = x`는 참인데, `2 = x`는 아닙니다. 심지어 매칭이 안되면 `MatchError`를 던져주기까지 하죠.

`=`를 통해서 값을 대입하는 것은 오른쪽에서 왼쪽으로만 가능합니다.

```iex
iex> 1 = unknown
** (RuntimeError) undefined function: unknown/0
````

Elixir는 `unknown`라는 변수가 정의되지 않은 상태에서는 사용자가 `unknown/0`을 호출하려고 시도한다고 생각합니다.

## Pattern matching

Elixir에서 `=`를 매칭 연산자라고 부르는 이유는 단순히 하나의 값만을 사용할 수 있는 것이 아닌, 복잡한 데이터 타입에 대해서도 여전히 유용하게 사용할 수 있기 때문입니다. 예를 들자면, 튜플을 다음처럼 매칭할 수 있죠.

```iex
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

위에서 봤듯, 매칭에 실패한다면 에러를 돌려줍니다. 예를 들어, 매칭하려는 튜플의 길이가 다르거나 할 때 말이죠.

```iex
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

물론 아예 다른 타입일 때도 마찬가지입니다.

```iex
iex> {a, b, c} = [:hello, "world", "!"]
** (MatchError) no match of right hand side value: [:hello, "world", "!"]
```

재미있는 점은 특정 값만을 매칭할 수 도 있다는 점입니다. 아래 예제에서는 값을 2개 가지는 튜플끼리 매칭을 하는데, 오른쪽의 튜플이 `:ok`를 첫번째 원소로 가질 때에만 매칭해줍니다.

```iex
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13

iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

이 패턴 매칭 개념은 `Prolog`에서 쓰던거랑 같은데, 약간 고도화가 되었다는 느낌입니다. 원래 출처는 어디길래 Erlang이나, 그 위에서 도는 Elixir에서도 이걸 봐야 하는 것인가... ㅇ<-<

마찬가지로 리스트도 매칭할 수 있으며,

```iex
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

머리와 꼬리로도 매칭할 수 있습니다.

```iex
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

`hd/1`과 `tl/1`의 동작과 마찬가지로 빈 리스트에서는 매칭이 동작하지 않습니다. 허허.

```iex
iex> [h|t] = []
** (MatchError) no match of right hand side value: []
```

`[head | tail]`같은 방식은 매칭할 때 뿐만이 아니라, 리스트에 값을 넣을 때도 사용할 수 있습니다. 이는 리스트의 실제 구현이 링크드 리스트랑 비슷하다는 걸 고려하면 매우 자연스럽습니다.

```iex
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0|list]
[0, 1, 2, 3]
```

비록 보고 싶지 않았더라고 하더라도요.

아무튼 패턴 매칭은 데이터 타입을 쉽게 분해할 수 있게 해주는 강력한 도구입니다. 그리고 이는 Elixir의 재귀 처리 또는 다른 데이터 타입들에도 적용되는 가장 기초가 되는 도구입니다.

## The pin operator

Elixir의 변수는 새로운 값을 할당할 수 있습니다.

```iex
iex> x = 1
1
iex> x = 2
2
```

만약 패턴 매칭시에 존재하는 변수값을, 매칭 대상의 값보다 우선하고 싶다면 핀 연산자 `^`를 사용해야 합니다.

```iex
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {y, ^x} = {2, 1}
{2, 1}
iex> y
2
iex> {y, ^x} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
```

x에 1을 대입한 뒤 핀 연산자를 통해서 새로운 대입을 막고 있으며, 때문에 마지막 예제는 다음과 동일합니다.

```
iex> {y, 1} = {2, 2}
** (MatchError) no match of right hand side value: {2, 2}
```

만약 하나의 변수가 패턴에서 한번 이상 사용된다면 모든 레퍼런스는 같은 방법으로 매칭되어야 합니다.

```iex
iex> {x, x} = {1, 1}
1
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

좀 더 자세히 설명하면, 첫번째 `x`는 이미 `1`과 매칭된 상태이기 때문에 두번째 `x`가 `2`와 매칭할 수 없는 것이죠.

때로는 매칭은 해야하지만, 뭐가 들어가든 상관 없는 경우가 있습니다. 이런 경우는 매칭을 위해서 일반적으로 언더스코어(`_`)를 사용합니다.

```iex
iex> [h|_] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

이 예제를 보면 꼬리를 매칭하기 위해서 `_`를 사용하고 있으며, 이를 통해서 명시적으로 꼬리의 데이터는 사용하지 않겠다는 것을 나타낼 수 있습니다. 이 변수는 일반적인 변수들과는 다른 특별한 녀석입니다. 위의 예제에서 `_`를 호출해보세요.

```iex
iex> _
** (CompileError) iex:1: unbound variable _
```

아무런 값도 대입되지 않습니다.

패턴 매칭은 개발자들에게 강력한 도구입니다만, 용도는 생각보다 제한적입니다. 예를 들어, 

```iex
iex> 3 = length([1,[2],3])
3
iex> length([1,[2],3]) = 3
** (CompileError) iex:1: illegal pattern
```

이런 식으로요. 정확한 동작 원리는 확인해봐야하지만, 아마도 처리 순서가 문제가 아닐까 하고. Prolog에서도 비슷한 이슈가 있었습니다[...] 물론 이러한 문제는 고의적으로 이런 식으로 작성하지 않는 이상은 발생하기 힘들거라고 생각하지만요. 예제는 예제일 뿐이죠. :)

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Pattern matching](http://elixir-lang.org/getting-started/pattern-matching.html)
