---
layout: post
title: "Elixir - 21: Where to go next"
date: 2016-02-11 13:00:29 +0900
categories:
---

Elixir Tutorial 시리즈입니다. 대부분은 튜토리얼의 한글 번역에 가깝습니다만, 생략되거나 추가로 주석을 달거나 하는 부분이 많습니다. 원문은 최하단의 링크를 참고하세요.

# Elixir - 21: Where to go next

더 배우고 싶어요?

## 일단 한번 만들어 보세요 XD

첫 프로젝트를 시작하려면 우선 Mix라는 이름의 Elixir 빌드 도구가 있습니다. 이 도구를 통해서 간단하게 새 프로젝트를 생성할 수 있습니다.

    mix new path/to/new/project

관리 트리, 설정, 테스트 등등을 포함한 Elixir 애플리케이션을 작성하는 가이드가 이미 준비되어 있습니다. 애플리케이션은 여러 노드에 분산된 키-값 저장소처럼 동작합니다.

* [Mix and OTP](http://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html)

## Meta-programming

Elixir는 메타 프로그래밍을 통해 확장 가능하고 쉽게 변경 가능한 프로그래밍 언어입니다. Elixir의 많은 메타 프로그래밍은 매크로를 통해서 동작하며 DSL을 작성할 경우에 무척 유용합니다. 매크로를 동작시키는 몇몇 기본적인 구조와 DSL을 위해서 어떻게 매크로를 작성하고 사용할지를 보여주는 가이드도 있으니 참고하세요.

* [Meta-programming in Elixir](http://elixir-lang.org/getting-started/meta/quote-and-unquote.html)

## 커뮤니티와 다른 자료들

Elixir를 배우기 위한 좋은 책이나 스크린 캐스트 등의 자료들을 [Learning](http://elixir-lang.org/learning.html)에서 공개하고 있으며, 여기에는 컨퍼런스 발표나 관련 오픈 소스 프로젝트 등도 포함되어 있습니다.

어떤 문제가 있을 때에는 **irc.freenode.net**의 **#elixir-lang** 채널에 찾아오는 것을 주저하지 마세요. 아니면 [메일링 리스트](https://groups.google.com/group/elixir-lang-talk)에 질문해도 좋습니다. 분명 거기에는 도움을 줄 의지가 있는 사람들이 있을거예요. 최신 뉴스나 안내를 확인하고 싶은 경우에는 [블로그](http://elixir-lang.org/blog/)와 언어 개발팀의 [메일링 리스트](https://groups.google.com/group/elixir-lang-core)를 구독해도 좋습니다.

아니면 직접 [Elixir의 코드](https://github.com/elixir-lang/elixir)를 확인해볼 수 있습니다. 대부분의 코드는 `lib` 코드에 존재하며, 아니면 [Elixir의 문서](http://elixir-lang.org/docs.html)를 읽어보는 것도 좋습니다.

## A byte of Erlang

Elixir는 Erlang 가상 머신에서 동작하며, 개발자는 이미 존재하는 Erlang 라이브러리를 사용해야하는 경우도 있을겁니다. 여기에서는 Erlang의 기본과 고급 기능들을 설명하는 온라인 자료들을 소개합니다.

* [Erlang Syntax: A Crash Course](http://elixir-lang.org/crash-course.html)는 Erlang의 문법에 대한 정확한 개요를 제공합니다. 각 코드 스니핏은 동일한 동작을 하는 Elixir 코드와 함께 제공됩니다. 이는 Erlang의 문법을 배울 수 있을 뿐만 아니라, 가이드에서 배웠던 내용들을 복습할 수 있는 기회가 될겁니다.

* Erlang의 공식 사이트에서도 그림을 포함한 짧은 [튜토리얼](http://www.erlang.org/course/concurrent_programming.html)을 통해 병렬 프로그래밍의 방법을 간단하게 보여주고 있습니다.

* [Learn You Some Erlang for Great Good!](http://learnyousomeerlang.com/)은 Erlang에 대한 디자인 원칙, 표준 라이브러리, 베스트 프랙티스 등을 설명하는 멋진 가이드를 제공합니다. 위에서 언급한 코드들을 전부 읽고 난 뒤라면 이 책의 1장에서 다루고 있는 대부분의 문법에 대한 설명을 넘길 수 있습니다. [The Hitchhiker's Guide to Concurrency](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency)까지 읽었다면, 이제 정말 재밌는 부분의 시작입니다.

## 여기까지의 감상
Elixir가 어떤 식으로 구성되어 있는지, 함수형 프로그램을 어떻게 잘 구현하는지, Fault-tolerant 할 수 있는 이유가 무엇인지, Ruby에서 볼 수 있던 요소들이 어떻게 여기에 반영이 되어있는지에 대해서는 잘 이해했음. 다만 그래서 뭐? 라는 감상이 드는데, 마치 무척 좋다는 도구를 보고 '그래서 이걸 어떻게 쓰면 좋은데?'라는 생각이 먼저 떠오르는 것처럼 어떻게 짜야 이 특징들을 잘 살릴 수 있는가, 라는 점에 대해서는 여전히 안개 속을 해매이는 듯한 느낌이 들고...

그래서 본가에서도 하나의 잘 동작하는 실제 프로젝트를 제공하는 모양이니 이것도 차근차근 진행해 볼 계획입니다. 아무튼 지금까지 짜오던 방식과는 완전히 다른-물론 Prolog에서도 많이 보던 애들도 있었지만- 관점에서 코드를 볼 수 있었던 좋은 경험이었고 앞으로도 꾸준히 보고 싶다는 생각이 드는 언어였네요.

## Reference
 * [Elixir Homepage](http://elixir-lang.org)
 * [Elixir Where to go next](http://elixir-lang.org/getting-started/where-to-go-next.html)

