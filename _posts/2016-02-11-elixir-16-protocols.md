---
layout: post
title: "Elixir - 16: Protocols"
date: 2016-02-11 12:58:20 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 16: Protocols

프로토콜은 Elixir에서 다형성을 구현하기 위한 한 가지 방법입니다. 프로토콜을 통해서 정보를 보내는 것은 해당하는 프로토콜을 구현하기만 한다면, 어떤 데이터 타입에서라도 가능합니다. 예제를 보죠.

Elixir에서는 `false`와 `nil`만이 부정값으로 취급됩니다. 그 이외의 것들은 모두 긍정으로 평가됩니다. 애플리케이션에 따라서는 다른 데이터 타입에서 비어있다고 취급되는 경우에 참을 반환하는 `blank?` 프로토콜이 중요할 때가 있습니다. 예를 들어서 빈 리스트나 빈 바이너리 역시 비어있다고 평가됩니다.

이 프로토콜은 다음과 같이 정의해볼 수 있습니다.

```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

이 프로토콜은 `blank?`라는 함수가 하나의 매개변수와 함께 호출될 것을 기대하고 있습니다. 우리는 이 프로토콜을 다른 Elixir 데이터 타입을 위해서 다음과 같이 구현할 수 있습니다.

```elixir
# Integers are never blank
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# Just empty list is blank
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

# Just empty map is blank
defimpl Blank, for: Map do
  # Keep in mind we could not pattern match on %{} because
  # it matches on all maps. We can however check if the size
  # is zero (and size is a fast operation).
  def blank?(map), do: map_size(map) == 0
end

# Just the atoms false and nil are blank
defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end
```

그리고 다음의 모든 기본 데이터 타입에도 똑같이 구현할 수 있을겁니다.

* `Atom`
* `BitString`
* `Float`
* `Function`
* `Integer`
* `List`
* `Map`
* `PID`
* `Port`
* `Reference`
* `Tuple`

이제 구현된 프로토콜을 통해서 호출해봅시다.

```iex
iex> Blank.blank?(0)
false
iex> Blank.blank?([])
true
iex> Blank.blank?([1, 2, 3])
false
```

프로토콜에서 정의하지 않은 데이터 타입을 넘기면 에러가 발생합니다.

```iex
iex> Blank.blank?("hello")
** (Protocol.UndefinedError) protocol Blank not implemented for "hello"
```

## Protocol과 구조체

Elixir의 확장성은 프로토콜과 구조체가 함께 쓰일 때로 부터 옵니다.

[이전 장](http://elixir-lang.org/getting-started/structs.html)에서, 구조체는 맵임에도 불구하고 프로토콜을 공유하지 않는다는 점을 배웠습니다. 그럼 그 장에서 구현했던 `User` 구조체를 다시 정의해봅시다.

```iex
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
```

그리고 확인해보면,

```iex
iex> Blank.blank?(%{})
true
iex> Blank.blank?(%User{})
** (Protocol.UndefinedError) protocol Blank not implemented for %User{age: 27, name: "john"}
```

맵과 프로토콜의 구현을 공유하는 대신에, 구조체는 자신만의 프로토콜 구현을 요구합니다.

```elixir
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

만약 원한다면, 당신이 원하는 방식대로 '비다'의 정의를 할 수도 있습니다. 그 뿐만이 아니라 `Enumerable`나 `Blank`처럼 모든 쓸모있는 프로토콜들을 전부 구현하는 것으로 구조체를 좀 더 견고한 데이터 타입으로 만들 수 있습니다.

## `Any` 구현하기

모든 구조체들에 대해서 프로토콜을 전부 일일히 구현하는 것은 반복적이고 매우 지루한 일입니다. 이러한 경우를 위해서 Elixir는 두가지 옵션을 제공합니다. 프로토콜들의 구현을 명시적으로 상속하거나, 또는 모든 타입을 위한 프로토콜을 자동적으로 구현하는 것입니다. 어느 경우든 간에 우리는 `Any`를 위한 프로토콜을 정의할 필요가 있습니다.

### 상속

Elixir는 `Any` 구현을 기반으로 프로토콜 구현을 상속할 수 있도록 해줍니다. 우선 하나 만들어 보죠.

```elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

이제 구조체를 정의할 때에 명시적으로 `Blank` 프로토콜의 구현을 상속받을 수 있습니다. `DeriveUser`라는 다른 구조체를 정의해보죠.

```elixir
defmodule DeriveUser do
  @derive Blank
  defstruct name: "john", age: 27
end
```

상속을 받을 때 Elixir는 `DeriveUser`를 위한 `Blank` 프로토콜을 `Any`를 위한 구현을 사용해서 구현합니다. 이 방식은 선택적이라는 것을 기억하세요. 구조체는 프로토콜이 명시적으로 구현되었거나, 상속되었을 경우에만 이를 사용합니다.

### `Any`에 Fallback하기

`@derive`의 또다른 선택지는 명시적으로 프로토콜에게 구현체가 없는 경우에 `Any`를 사용하라고 알려주는 것입니다. 이는 프로토콜 정의에서 `@fallback_to_any`를 `true`로 설정하면 됩니다.

```elixir
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```

이전 절에서 `Any`를 정의했다고 해보죠.

```elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

이제 우리가 구현하지 않은 (구조체를 포함한) 모든 데이터 타입은  `Blank` 프로토콜에 의해서 비어있지 않다고 평가됩니다. `@derive`와 대조적으로 `Any`에 대한 fallback은 선택적이지 않습니다. 자기 자신에 대한 프로토콜 구현이 존재하지 않는 모든 데이터 타입들은 `Any`를 위한 구현을 사용하게 됩니다. 이들은 각각 상황에 따라서 선택될 수 있습니다만 Elixir 개발자들은 암묵적인 것 보다는 명시적인 것을 선호하기 때문에 많은 라이브러리에서는 `@derive`식 접근을 하고 있는걸 볼 수 있을 겁니다.

## 내장 프로토콜

Elixir는 몇몇 내장 프로토콜을 가지고 있습니다. 이전 장에서 `Enumerable` 프로토콜을 구현한, 어떤 데이터 구조와도 동작하는 많은 함수를 제공하는 `Enum` 모듈에 대해서 이야기를 했습니다.

```iex
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2,4,6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

다른 유용한 예제로는 데이터 구조의 문자들을 문자열로 변환해주는 `String.Chars` 프로토콜이 있습니다. 이는 `to_string` 함수를 제공합니다.

```iex
iex> to_string :hello
"hello"
```

Elixir에서의 문자열 보간이 `to_string`를 통해서 이루어진다는 점을 기억하세요.

```iex
iex> "age: #{25}"
"age: 25"
```

이 코드는 숫자에 `String.Chars` 프로토콜이 구현되어 있기 때문에 잘 동작합니다. 튜플을 넘기면 다음과 같이 에러가 발생할 겁니다.

```iex
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

만약 더 복잡한 데이터 구조를 출력해야한다면 `Inspect` 프로토콜의 `inspect` 함수를 사용하세요.

```iex
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

`Inspect` 프로토콜은 어떤 데이터 구조든 읽을 수 있는 형태로 변환해주는 프로토콜입니다. 이는 IEx와 같은 도구에서 결과를 출력할 때 사용하는 함수죠.

```iex
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "john", age: 27}
```

기억해두어야 할 점은 관습적으로 `#`으로 시작되는 값을 조사하는 경우에는 Elixir 문법에 유효하지 않은 데이터 구조라는 의미입니다. 이는 다시 말해서 조사 프로토콜은 이전 데이터를 복원할 수 없는(not reversible)한 형태로 결과를 돌려줍니다.

```iex
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```


## Protocol 뭉치기

Elixir 프로젝트에서 작업을 하다 보면 Mix 빌드 도구를 사용하다가 다음과 같은 출력을 자주 보게 될겁니다.

```
Consolidated String.Chars
Consolidated Collectable
Consolidated List.Chars
Consolidated IEx.Info
Consolidated Enumerable
Consolidated Inspect
```

이 모든 프로토콜들은 Elixir에 포함되어 있으며 그것들은 이미 병합된 것들입니다. 프로토콜은 어떤 데이터 타입으로부터도 호출될 수 있기 때문에, 주어진 타입의 구현이 존재한다면 프로토콜은 모든 호출에 대해서 검증을 해야합니다. 이는 프로그램 입장에서 무척 큰 비용입니다.

하지만 Mix와 같은 툴을 사용하여 프로젝트를 컴파일하면 프로토콜과 각 구현을 포함한 모든 모듈이 이미 정의되어 있다는 것을 알 수 있습니다. 이 방법에서는 프로토콜이 전부 하나의 매우 간단하고 빠른 디스패치 모듈에 병합됩니다.

Elixir v1.2부터는 프로토콜 병합이 모든 프로젝트에서 자동으로 이루어집니다.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Protocols](http://elixir-lang.org/getting-started/protocols.html)
