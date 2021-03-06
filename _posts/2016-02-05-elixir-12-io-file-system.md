---
layout: post
title: "Elixir - 12: IO와 File System"
date: 2016-02-05 12:33:41 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 12: IO와 File System

이 장에서는 input/output 방식과 파일 시스템을 처리하는 방법, 그리고 [`IO`](http://elixir-lang.org/docs/stable/elixir/IO.html), [`File`](http://elixir-lang.org/docs/stable/elixir/File.html), 그리고 [`Path`](http://elixir-lang.org/docs/stable/elixir/Path.html) 모듈에 대해서도 알아봅니다.

원래는 이 챕터를 튜토리얼의 앞에 가져올 계획이었습니다. 하지만 IO 시스템에 대해 설명하다보면 Elixir의 철학과 <abbr title="Virtual Machine">VM</abbr>에 대한 호기심을 해결할 수 있는 좋은 기회를 제공할 것이라는 것을 깨달았죠.

## `IO` 모듈

`IO` 모듈은 표준 입출력(`:stdio`), 표준 에러(`:stderr`), 파일과 다른 IO 장치들을 읽고 쓰는 Elixir의 가장 기본이 되는 방법입니다. 이 모듈을 쓰는 방법은 무척 직관적입니다.

```iex
iex> IO.puts "hello world"
hello world
:ok
iex> IO.gets "yes or no? "
yes or no? yes
"yes\n"
```

기본으로 IO 모듈에 있는 함수는 표준 입력을 읽으며, 표준 출력에 씁니다. 이러한 타겟을 예를 들어 `:stderr`를 넘겨서 변경할 수 있습니다.

```iex
iex> IO.puts :stderr, "hello world"
hello world
:ok
```

## `File` 모듈

[`File`](http://elixir-lang.org/docs/stable/elixir/File.html) 모듈은 파일을 IO 장치인것처럼 다룰 수 있게 해주는 함수들을 포함하고 있습니다. 기본으로 파일들은 바이너리 모드로 열리며, 다시 말해 개발자들에게 `IO.binread/2`나 `IO.binwrite/2`와 같은 함수를 쓰게끔 합니다.

```iex
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
iex> IO.binwrite file, "world"
:ok
iex> File.close file
:ok
iex> File.read "hello"
{:ok, "world"}
```

파일을 `:utf8` 인코딩을 사용해서 열 수도 있습니다.

나아가 `File` 모듈에는 파일을 열고, 읽고, 쓸 때에 사용하는 많은 함수들이 존재하며, 이러한 함수들은 UNIX 명령어와 유사하게 명명되어 있습니다. 예를 들어 `File.rm/1` 는 파일을 제거할 때에 사용하며, `File.mkdir/1`는 폴더를 만들 때, `File.mkdir_p/1`는 폴더를 체이닝해서 생성할 때(='mkdir -p')에 사용합니다. 심지어 `File.cp_r/2`와 `File.rm_rf/1`와 같이 재귀적인 명령을 처리하는 함수들도 있습니다.

그리고 `File` 모듈의 함수들을 보다보면 두 가지 종류가 있다는 점을 발견할 수 있습니다. 하나는 일반적인 함수이며, 나머지 하나는 뒤에 `!`가 붙는 함수들입니다. 예를 들어 우리가 위의 예제에서 만들었던 `"hello"` 파일을 읽는다면 `File.read/1`를 사용할 수 있습니다. 또는, `File.read!/1`를 사용할 수도 있죠.

```iex
iex> File.read "hello"
{:ok, "world"}
iex> File.read! "hello"
"world"
iex> File.read "unknown"
{:error, :enoent}
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
```

다른 점은, 파일이 존재하지 않는 경우 `!` 버전은 에러를 발생시킨다는 점입니다. 반면 `!`가 없는 경우에는 개발자가 패턴 매칭을 통해서 직접 에러를 다룰 수 있는 형태의 결과를 돌려줍니다.

```elixir
case File.read(file) do
  {:ok, body}      -> # do something with the `body`
  {:error, reason} -> # handle the error caused by `reason`
end
```

하지만 파일이 반드시 있어야 한다는 전제를 한다면 `!` 버전이 좀 더 의미있는 에러 메시지를 던져주므로 좀 더 유용합니다. 그러니 다음과 같이 작성하지 마세요.

```elixir
{:ok, body} = File.read(file)
```

이러한 경우, 에러가 발생하면 `File.read/1`는 `{:error, reason}`를 돌려줄 것이기 때문에 패턴 매칭이 실패합니다. 물론 여전히 에러가 발생한다는 결과는 얻을 수 있습니다만, 매칭이 되지 않았기 때문에 어떤 에러가 발생했는지는 미궁속에 빠져들게 됩니다.

그러므로 에러를 직접 다룰 생각이 아니라면 `File.read!/1`를 사용하세요.

## `Path` 모듈

`File` 모듈에 있는 많은 함수들이 경로를 매개변수로 넘겨 받습니다. 일반적으로 이러한 경로들은 표준 바이너리[?]입니다. [`Path`](http://elixir-lang.org/docs/stable/elixir/Path.html) 모듈은 다음과 같이 경로를 다룰 수 있게 해줍니다.

```iex
iex> Path.join("foo", "bar")
"foo/bar"
iex> Path.expand("~/hello")
"/Users/jose/hello"
```

`Path` 모듈에 포함되어 있는 함수들은 단순히 바이너리 경로를 생성하는 것 뿐만이 아니라, 서로 다른 운영체제들의 파일 경로들을 동일한 방식으로 다루어줍니다. 마지막으로 Elixir는 윈도우에서 파일 시스템을 다루는 경우 `/`를 자동으로 `\`로 변환한다는 점을 기억하세요.

여기까지로 우리는 IO와 파일 시스템을 다루는 Elixir의 모듈을 살펴보았습니다. 다음 절에서는 IO와 관련된 좀더 나아간 주제를 다루어볼까 합니다. 이 절은 모르더라도 Elixir 코드를 작성하는데에 아무런 지장도 없으니 넘어가도 됩니다만, 여기에서는 IO 시스템이 어떻게 VM에서 다루어지는지를 멋지게 설명합니다.

## Processes와 그룹 리더

`File.open/2`가 `{:ok, pid}`와 같은 튜플을 반환하던 것을 기억하시나요?

```iex
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
```

이는 `IO` 모듈이 프로세스를 통해서 동작하기 때문입니다([chapter 11](http://elixir-lang.org/getting-started/processes.html) 참조). `IO.write(pid, binary)`라는 코드를 작성하게 되면 `IO` 모듈은 `pid`에 요구받은 작업을 수행하라는 메시지를 보내게 됩니다. 실제로 어떤 일이 일어나는지 살펴보시죠.

```iex
iex> pid = spawn fn ->
...>  receive do: (msg -> IO.inspect msg)
...> end
#PID<0.57.0>
iex> IO.write(pid, "hello")
{:io_request, #PID<0.41.0>, #PID<0.57.0>, {:put_chars, :unicode, "hello"}}
** (ErlangError) erlang error: :terminated
```

`IO.write/2` 호출 뒤에 어떤 메시지를 보냈는지 확인할 수 있습니다. 그리고 곧 바로 우리가 생각하지 않은 에러가 발생하는 것도 볼 수 있죠.

[`StringIO`](http://elixir-lang.org/docs/stable/elixir/StringIO.html) 모듈은 문자열을 처리하기 위한 `IO` 장치의 메시지를 구현하고 있습니다.

```iex
iex> {:ok, pid} = StringIO.open("hello")
{:ok, #PID<0.43.0>}
iex> IO.read(pid, 2)
"he"
```

프로세스를 사용해서 IO 장치를 모델링하는 것은 Erlang <abbr title="Virtual Machine">VM</abbr>이 각 노드간의 파일 읽기/쓰기를 처리하기 위해서입니다. 그리고 모든 IO 장치들은 하나의 특별한 프로세스를 가지고 있습니다. 바로 **그룹 리더**죠.

`:stdio`에 출력을 하고 싶다면, 그룹 리더에게 메시지를 보내서, 표준 출력 파일 디스크립터에 쓰기 요청을 해야합니다.

```iex
iex> IO.puts :stdio, "hello"
hello
:ok
iex> IO.puts Process.group_leader, "hello"
hello
:ok
```

그룹 리더는 프로세스 별로 설정할 수 있으며, 각각 다른 상황에서 사용됩니다. 예를 들어 원격 터미널에서 코드를 실행하는 경우, 원격 노드에 있는 메시지가 전송되고, 출력한 뒤 요청이 실행되는 것을 보장합니다.

## `iodata`와 `chardata`

지금까지의 예제에서는 파일 쓰기를 위해서 바이너리를 사용했습니다. ["Binaries, strings and char lists"](http://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html)에서 문자열이 바이트로 구성되어 있으며 문자 리스트가 문자 코드인지를 배웠습니다.

`IO`와 `File`에 있는 함수들도 매개변수로 리스트를 받을 수 있습니다. 정수, 바이너리 뿐 아니라 중첩된 리스트도 받구요.

```iex
iex> IO.puts 'hello world'
hello world
:ok
iex> IO.puts ['hello', ?\s, "world"]
hello world
:ok
```

하지만 그렇게 사용하는 경우에는 약간의 주의가 필요합니다. 리스트는 한 뭉치의 바이트 또는 문자들을 표현하며, 이는 IO 장치의 인코딩에 의존합니다. 만약 파일이 인코딩 설정 없이 열려있다면, 이 파일은 raw mode로 동작할 것이며, `IO` 모듈에서 `bin*` 함수들이 쓰여야만 합니다. 이러한 함수들은 `iodata`를 매개변수로 요구합니다. 다시 말해 바이트나 바이너리를 표현하는 숫자를 기대합니다.

반면 `:stdio`나 `:utf8`로 열린 파일의 경우에는 `IO` 모듈의 다른 함수들과 함께 동작할 수 있습니다. 이러한 함수들은 `char_data`를 매개변수로 기대하며, 다시 말해서 문자 리스트나 문자열을 원한다는 의미입니다.

다만 이러한 섬세한 차이점은 함수들에 리스트를 넘기는 경우에만 고려하면 됩니다. 바이너리들은 이미 기저에서 바이트와 이와 비슷한 표현식으로 구성되어 있기 때문이죠.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir IO and the file system](http://elixir-lang.org/getting-started/io-and-the-file-system.html)

