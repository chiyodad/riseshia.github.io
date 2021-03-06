---
layout: post
title: "Elixir - 07: Keywords and maps"
date: 2016-01-10 08:29:55 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 07: Keywords and maps

이제 연결 데이터 구조에 대해서 이야기해볼 시간입니다. 다른 언어에서는 사전이라든가, 해시, 연관 배열 등으로 불리는 바로 그것입니다.

Elixir에서는 2개의 연결 데이터 구조가 있습니다. 키워드 목록과 맵입니다.

## Keyword lists

많은 함수형 언어에서는 2개의 원소를 가지는 튜플을 사용하는 것은 무척 흔합니다. Elixir에서는 첫번째 원소로 아톰을 가지는 튜플의 리스트를 키워드 목록이라고 부릅니다.

```iex
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
iex> list[:a]
1
```

위에서 볼 수 있듯, Elixir는 저런 리스트를 처리하기 위한 특별한 문법을 지원합니다. 이것들은 간단한 리스트이므로, 리스트에 사용할 수 있는 연산은 똑같이 적용될 수 있습니다. 예를 들어, `++`를 사용해서 새 값을 저장할 수 있을겁니다.

```iex
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```

같은 아톰이 여러개 들어 있는 경우 앞에서 첫번재로 나오는 것이 반환됩니다.

```iex
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

키워드 목록은 다음과 같은 3개의 특징을 가집니다.

  * 키는 반드시 아톰이어야 합니다.
  * 키는 개발자가 정의한 순서가 있습니다.
  * 키는 중복될 수 있습니다.

예를 들어, [Ecto library](https://github.com/elixir-lang/ecto) 는 이러한 기능으로 멋진 DSL를 정의하여 데이터베이스 쿼리를 작성할 때 사용합니다.

```elixir
query = from w in Weather,
      where: w.prcp > 0,
      where: w.temp < 20,
     select: w
```

키워드 목록을 통하여 함수에 옵션을 넘기는 것은 Elixir의 기본 기능입니다. 이전 5장에서 우리는 `if/2` 매크로에 대해서 언급했으며, 다음과 같은 문법이 지원된다고 이야기한 적이 있습니다.

```iex
iex> if false, do: :this, else: :that
:that
```

`do:`와 `else:` 쌍은 키워드 목록입니다. 사실 이 호출은 다음과 동일합니다.

```iex
iex> if(false, [do: :this, else: :that])
:that
```

일반적으로 키워드 목록은 함수의 마지막 인자이며, `[]`는 선택적이라는 의미입니다.

키워드 목록을 만들기 위해 Elixir는 [`Keyword`](http://elixir-lang.org/docs/stable/elixir/Keyword.html) 모듈을 제공합니다. 다시 한 번 말씀드리지만, 키워드 목록은 단순한 리스트이므로 다른 리스트와 마찬가지로 선형적인 성능을 보여줍니다. 리스트가 길어질 수록 키를 찾는 데에 걸리는 시간과 원소를 세는데 걸리는 시간이 길어집니다. 이러한 이유로 키워드 목록은 Elixir에서 주로 옵션으로 사용됩니다. 만약 많은 원소를 저장하고 싶거나 한 키를 반드시 하나의 값과 연결하고 싶다면 맵을 사용해야합니다.

키워드 목록에도 패턴 매칭을 사용할 수 있지만, 이 때 원소의 갯수와 순서가 정확히 매칭되어야 하므로 실제로는 거의 쓰이지 않습니다.

```iex
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```

## Maps

키-값 저장소가 필요한 경우, 맵은 Elixir의 유일한 방안입니다. 맵은 `%{}` 문법을 통해서 정의할 수 있습니다.

```iex
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
iex> map[:c]
nil
```

키워드 목록과 비교하여 다음과 같은 두 가지의 다른점을 발견할 수 있습니다.

  * 맵은 키로 임의의 타입을 받습니다.
  * 맵의 키는 순서를 고려하지 않습니다.

키워드 목록에서와는 다르게 맵에서는 패턴 매칭이 매우 유용합니다. 맵에서 패턴 매칭할 때의 다른 점은 주어진 값의 부분 집합을 사용한다는 점입니다.

```iex
iex> %{} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```

이 코드를 살펴보면, 맵은 주어진 맵의 패턴에 있는 모든 키들과 가능한 매칭을 시도합니다. 그러므로 빈 맵은 모든 맵들과 매칭할 수 있습니다.

이 시점에서 Elixir의 패턴 매칭 규칙이 점점 확신이 안가는 상황. 예를 들어, 평가는 어떤 방향으로 이루어 지는가, 어떤 건 매칭하고 어떤 건 매칭하지 않는가, 등등. 기회가 된다면 이건 나중에 좀 살펴볼 필요가 있을 듯.

변수를 통해서 값에 접근, 매칭, 또는 값을 추가할 수도 있습니다.

```iex
iex> n = 1
1
iex> map = %{n => :one}
%{1 => :one}
iex> map[n]
:one
iex> %{^n => :one} = %{1 => :one, 2 => :two, 3 => :three}
```

이 시점에서 발견한 사실. `[]` 접근자를 통해서는 값에 접근, 매칭만 할 수 있다는 점.

```iex
iex> map[:by] = "blabla"
** (CompileError) iex: cannot invoke remote function Access.get/2 inside match
```

[`Map` 모듈](http://elixir-lang.org/docs/stable/elixir/Map.html)은 조작을 위해 `Keyword` 모듈과 무척 유사한 API를 제공합니다.

```iex
iex> Map.get(%{:a => 1, 2 => :b}, :a)
1
iex> Map.to_list(%{:a => 1, 2 => :b})
[{2, :b}, {:a, 1}]
```

맵에 있는 모든 키가 아톰일 때, 편의를 위해 다음과 같이 작성할 수도 있습니다.

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

Ruby Hash 문법이랑 비슷해서 좋네요. 그리고 재미있는 것 하나는 맵에서 각 키에 접근하기 위한 전용 문법을 제공한다는 점입니다.

```iex
iex> map = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}

iex> map.a
1
iex> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}

iex> %{map | :a => 2}
%{:a => 2, 2 => :b}
iex> %{map | :c => 3}
** (KeyError) key :c not found in: %{2 => :b, :a => 1}
```

단 접근, 변경 모두 주어진 키가 존재한다는 전제 하에서만 이루어집니다. 예를 들어서 존재하지 않는 `:c`라는 키를 변경하려고 시도하면 실패합니다.

Elixir 개발자들은 대부분 선언적인 프로그래밍을 할 수 있다는 이유로 `Map` 모듈에 있는 함수들보다는 `map.field` 문법과 패턴 매칭을 선호합니다. [이 글](http://blog.plataformatec.com.br/2014/09/writing-assertive-code-with-elixir/)에서는 Elixir에서 선언적인 코드를 통해 어떻게 좀 더 튼튼하고 빠른 프로그램을 작성할 수 있는지에 대한 통찰력과 예제를 확인할 수 있다.

최근에 맵이 Erlang<abbr title="Virtual Machine">VM</abbr>에 도입되었으며, 이를 통해 Elixir v1.2에서는 수백만개의 키들을 호율적으로 다룰 수 있게 되었습니다. 그러므로 만약 Elixir v1.0이나 v1.1을 사용하고 대량의 키를 사용해야하는 경우라면, [`HashDict` 모듈](http://elixir-lang.org/docs/stable/elixir/HashDict.html)의 사용을 고려해보세요.

## Nested data structures

맵 속에 맵이나 키워드 목록을 넣는다든가, 더 집어넣는다는가, 하는 경우는 꽤 빈번하게 있습니다. Elixir에서는 이러한 중첩된 자료구조를 편하게 조작하기 위해서 다른 불변 속성을 사용하는 명령형 언어들에서 찾아볼 수 있는 것과 동일한 기능을 하는 `put_in/2`와 `update_in/2`, 그리고 다수의 매크로들을 지원하고 있습니다.

다음과 같은 자료 구조를 생각해봅시다.

```iex
iex> users = [
  john: %{name: "John", age: 27, languages: ["Erlang", "Ruby", "Elixir"]},
  mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}
]
```

이 키워드 목록에서는 각 사용자들의 이름, 나이, 좋아하는 언어를 저장하고 있습니다. 이 구조를 정확히 설명하자면 '리스트 -> 튜플 -> 맵 -> 리스트' 같은 느낌이 되겠네요. 이 변수에서 John의 나이를 가져오고 싶다면 다음과 같이 작성할 수 있습니다.

```iex
iex> users[:john].age
27
```

같은 문법을 통해서 값을 변경해봅시다.

```iex
iex> users = put_in users[:john].age, 31
[john: %{name: "John", age: 31, languages: ["Erlang", "Ruby", "Elixir"]},
 mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#", "Clojure"]}]
```

`update_in/2` 매크로도 이와 유사합니다만, 값을 어떻게 바꿀지에 대한 방법을 함수로 던질 수 있다는 차이점이 있습니다. 예를 들어 "Clojure"를 Mary가 좋아하는 언어 목록에서 제거해보죠.

```iex
iex> users = update_in users[:mary].languages, &List.delete(&1, "Clojure")
[john: %{name: "John", age: 31, languages: ["Erlang", "Ruby", "Elixir"]},
 mary: %{name: "Mary", age: 29, languages: ["Elixir", "F#"]}]
```

`put_in/2`와 `update_in/2`에 대해서는 `get_and_update_in/2`는 값을 꺼내오면서 동시에 변경할 수 있다는 것을 포함해서 배울 것들이 더 있습니다. `put_in/3`, `update_in/3`, `get_and_update_in/3` 는 자료구조에 동적으로 접근할 수 있게 해줍니다. [`Kernel` 모듈 문서에서 각각의 자세한 설명을 확인해보세요](http://elixir-lang.org/docs/stable/elixir/Kernel.html).

## Consideration

 * 맵에서 각 원소에 접근할 때에 사용할 수 있는 접근함수를 제공해주는건 신선한 발상이라는 생각이 들었는데, 생각해보니 js에서 이미 비슷한걸 제공하고 있었음.
 * 더불어서 이건 중첩된 자료구조에서의 접근을 용이하게 만들기 위한 기법이 아니었을까 하는 생각도. 반면 오히려 method call과 getter의 반복된 사용으로 오히려 실수를 하기 쉬운 구조를 만들지는 않을까, 하는 생각도 들고.
 * 계속해서 선언적 언어랑 OOP를 적당히 절충했다는 느낌이 듬. 다만 hd/ht말고 평범한 setter를 제공해줬으면 안됬을까 하는 생각은 여전히... 물론 이게 재귀에서 무척 강력하고 깔끔한 기술을 제공한다는 점을 알고 있지만.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Keywords and maps](http://elixir-lang.org/getting-started/keywords-and-maps.html)
