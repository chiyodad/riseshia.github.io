---
layout: post
title: "Elixir - 10: Enumerables and Streams"
date: 2016-01-25 11:39:19 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 10: Enumerables and Streams

## Enumerables

Elixir는 열거 가능한 구조와 이를 위한 [`Enum` 모듈](http://elixir-lang.org/docs/stable/elixir/Enum.html)을 제공합니다.

```iex
iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]
```

`Enum` 모듈은 열거 가능한 목록으로부터 변환, 정렬, 그룹, 필터, 추출 등의 작업을 수행할 수 있는 다양한 함수들을 제공합니다.

또한, 범위 객체도 제공합니다.

```iex
iex> Enum.map(1..3, fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.reduce(1..3, 0, &+/2)
6
```

Enum 모듈의 함수들은 이름 그대로, 데이터 구조에 있는 값을 열거하는 데에 주안점을 두고 있습니다. 그러므로 값을 추가하거나, 변경하는 것과 같은 특정 데이터 타입에 의존하는 작업을 하려고 하는 경우에는 그에 알맞는 모듈의 함수가 필요합니다. 예를 들어서 리스트의 어떤 위치에 값을 삽입하고 싶다면 [`List` 모듈](http://elixir-lang.org/docs/stable/elixir/List.html)에 있는 `List.insert_at/3`를 사용해야합니다.

열거 함수들은 다양한 데이터 형식과 함께 동작해야하므로 이 모듈에 속해 있는 함수들은 모두 다형적입니다. 다시 말하자면 [`Enumerable` 프로토콜](http://elixir-lang.org/docs/stable/elixir/Enumerable.html)을 구현하고 있는 어떤 종류의 데이터 타입과도 동작할 수 있습니다. 프로토콜에 대해서는 나중에 설명하고, 우선 스트림이라고 불리는 특별한 열거자를 봅시다.

## Eager vs Lazy

`Enum` 모듈에 속해 있는 모든 함수들은 우선 평가됩니다. 많은 함수들은 열거가 가능할 것으로 기대하며 또한 리스트를 반환할 것이라 기대합니다.

```iex
iex> odd? = &(rem(&1, 2) != 0)
#Function<6.80484245/1 in :erl_eval.expr/5>
iex> Enum.filter(1..3, odd?)
[1, 3]
```

이 말은 `Enum`과 함께 연속된 연산을 할 때에, 각 연산은 즉각즉각 리스트를 생성해야한다는 점입니다.

```iex
iex> 1..100_000 |> Enum.map(&(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
7500000000
```

이 예제에서는 파이프라인 연산자를 사용합니다. 우선, 범위 객체로 시작해서 각 값들에 3을 곱합니다. 이 첫번째 연산은 `100_000`개의 원소 리스트를 반환합니다. 그리고 여기에서 모든 짝수 객체를 필터링하여 새 리스트를 생성하며, 결과로 `50_000`개의 원소들을 가지게 됩니다. 마지막으로 이 모든 원소들을 하나로 합쳐서 반환합니다.

## 파이프 연산자

`|>` 심볼은 **파이프 연산자**라고 불립니다. 이는 단순하게 왼쪽 연산의 출력을 오른쪽 함수의 첫번째 인자로 넘겨줍니다. 이는 Unix에 있는 `|`와 비슷하죠. 이는 여러 함수를 사용하며 변경되는 데이터의 흐름을 강조하기 위한 목적으로 사용됩니다. 이 방식이 어떻게 코드를 보기 좋게 만드는지는 `|>` 연산자를 쓰지 않는, 위와 동일한 동작을 하는 코드와 비교해보세요.

```iex
iex> Enum.sum(Enum.filter(Enum.map(1..100_000, &(&1 * 3)), odd?))
7500000000
```

파이프 연산자에 대한 추가 정보는 [이 문서](http://elixir-lang.org/docs/stable/elixir/Kernel.html#|>/2)를 참고하세요..

## Streams

`Enum`의 대체재로 Elixir는 지연 평가를 지원하는 [`Stream` 모듈](http://elixir-lang.org/docs/stable/elixir/Stream.html)을 제공합니다.

```iex
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
7500000000
```

스트림은 게으르며, 변경가능한 열거자입니다.

이 예제에서는 `1..100_000 |> Stream.map(&(&1 * 3))`는 데이터 타입, `1..100_000`를 `map` 연산하는 실제 스트림을 반환합니다.

```iex
iex> 1..100_000 |> Stream.map(&(&1 * 3))
#Stream<[enum: 1..100000, funs: [#Function<34.16982430/1 in Stream.map/2>]]>
```

나아가서 스트림 연산을 파이프를 통해 연결할 수 있으므로, 이는 간단하게 조합 가능하다는 장점도 있습니다.

```iex
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
#Stream<[enum: 1..100000, funs: [...]]>
```

곧바로 리스트를 생성하는 대신, 스트림은 계산 목록을 생성하며, 이 목록은 마지막에 `Enum` 모듈에 넘겨지기 전까지 계산되지 않습니다. 스트림은 커다란, **때때로는 무한한** 컬렉션들을 연산할 때에 유용합니다.

`Stream`에 존재하는 많은 함수들은 어떤 열거 가능한 객체라도 인자로 받을 수 있으며, 스트림 객체를 반환합니다. 이외에도 스트림을 생성하기 위한 함수도 제공합니다. 예를 들어, `Stream.cycle/1`는 주어진 열거 가능한 객체를 빙글빙글 무한대로 반환하는 스트림을 생성할 수 있습니다. 이러한 스트림에는 `Enum.map/2`같은 함수를 호출해서는 안됩니다. 왜냐하면 끝이 없기 때문에 무한 루프에 빠지게 됩니다.

```iex
iex> stream = Stream.cycle([1, 2, 3])
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
```

반면 `Stream.unfold/2`는 주어진 초기값으로부터 값을 생성할 수 있습니다.

```iex
iex> stream = Stream.unfold("hełło", &String.next_codepoint/1)
#Function<39.75994740/2 in Stream.unfold/2>
iex> Enum.take(stream, 3)
["h", "e", "ł"]
```

`Stream.resource/3`라는 녀석은 다른 리소스를 감싸기 위해서 사용되며, 이를 통해 열거 작업이 시작되기 전에 열리며, 끝나고 나서 닫힌다는 것을 보장합니다. 심지어 작업이 실패한 경우에도 말이죠. 예를 들어 파일에 사용할 수 있을 겁니다.

```iex
iex> stream = File.stream!("path/to/file")
#Function<18.16982430/2 in Stream.resource/3>
iex> Enum.take(stream, 10)
```

이 예제는 주어진 파일에서 처음 10개의 라인을 가져옵니다. 이 말인 즉슨, 스트림은 큰 사이즈의 파일, 또는 무척 느린 네트워크의 리소스를 다룰 때에도 유용하다는 의미죠.

[`Enum`](http://elixir-lang.org/docs/stable/elixir/Enum.html)과 [`Stream`](http://elixir-lang.org/docs/stable/elixir/Stream.html) 모듈의 수많은 함수들에 압도될 수는 있지만, 쓰다보면 하나씩 익숙해질겁니다. 우선 `Enum` 모듈에 집중하고, 지연 평가가 필요한 경우에 `Stream` 모듈에 있는 함수들을 하나씩 다루기 시작하는 것이 이상적입니다.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Enumerables and Streams](http://elixir-lang.org/getting-started/enumerables-and-streams.html)
