---
layout: post
title: "Elixir - 15: Structs"
date: 2016-02-05 12:34:50 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 15: Structs

[7장](http://elixir-lang.org/getting-started/keywords-and-maps.html)에서 맵을 배웠습니다.

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

구조체는 맵 위에 만들어지는 확장이며, 컴파일 시간 검증과 기본값을 사용할 수 있습니다.

## 구조체 정의하기

구조체를 정의하려면 `defstruct` 예약어를 사용하세요.

```iex
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

`defstruct`와 함께 사용되는 키워드 목록은 구조체에 들어갈 속성과 기본값으로 사용될 값을 지정합니다.

구조체는 자신이 정의될 이름도 받을 수 있습니다. 이 예제에서는 구조체의 이름을 `User`라고 정의했습니다.

이제 이를 통해서 맵을 생성하듯 `User` 구조체를 만들 수 있습니다.

```iex
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

구조체는 **컴파일 시간**에 `defstruct` 선언시에 허가되었던 속성값만을 사용했는지를 검증합니다.

```iex
iex> %User{oops: :field}
** (CompileError) iex:3: unknown key :oops for struct User
```

## 구조체에 접근/변경하기

맵에 대해서 이야기 할 때 어떻게 접근하고 변경할 지에 대해서 보였습니다. 이는 구조체에도 동일하게 적용할 수 있습니다.

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "Meg"}
```

변경용 문법인 `|`를 사용할 때, <abbr title="Virtual Machine">VM</abbr>은 구조체에 새로운 키가 추가될 리 없다는 사실을 이미 알고 있으며, 메모리상에 있는 구조를 공유할 수 있게 해줍니다. 이 예제에서는 `john`과 `meg`는 같은 키 구조를 메모리에서 공유하게 됩니다.

구조체는 패턴 매칭에서도 사용되며, 각 키에 맞는 값이 매칭해야 하며, 매칭되는 값이 같은 타입의 구조체일 경우에만 매칭됩니다.

```iex
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## 구조체는 거의 맵과 동일함

위 예제에서는 패턴 매칭이 동작합니다. 왜냐하면 구조체의 기저에 있는 것은 그저 고정된 필드 목록을 가지는 맵이기 때문입니다. 구조체는 맵으로서, `__struct__`라는 "특별한" 필드에 구조체의 이름을 저장하고 있습니다.

```iex
iex> is_map(john)
true
iex> john.__struct__
User
```

위에서 구조체는 **거의** 맵과 동일하다고 말했다는 점에 주의하세요. 이는 맵에 구현된 프로토콜이 구조체에서는 동작하지 않기 때문입니다. 예를 들어서, 구조체를 열거하거나 접근할 수 없습니다.

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (UndefinedFunctionError) undefined function: User.fetch/2
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

하지만 구조체도 사실은 맵이기 때문에 `Map` 모듈에 있는 함수들은 사용할 수 있습니다.

```iex
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

구조체는 프로토콜과 함께. Elixir 개발자들에게 중요한 기능 - 다형성 -을 제공합니다.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Structs](http://elixir-lang.org/getting-started/struct.html)
