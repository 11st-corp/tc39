# ECMAScript 제안서 : 문자열의 속성 + 집합 표기법과 정규표현식 `v` 플래그

## 저자

- Markus Scherer
- Mathias Bynens

## 상태

이 제안서는 the 2023년 5월 16일 회의 동안 [the TC39 절차](https://tc39.es/process-document/)의 4단계에 도달하였습니다. 이 것은 2023년 6월 15일에 [the ECMAScript 사양](https://tc39.es/ecma262/)으로 [추가](https://github.com/tc39/ecma262/pull/2418)되어 the ECMAScript 2024 스냅샷에 포함될 예정입니다.

2021년 5월 25일 TC39 회의를 기준으로, 이 제안은 공식적으로 [문자열 제안의 속성](https://github.com/tc39/proposal-regexp-unicode-sequence-properties)으로 가정합니다.

## 요약

ECMAScript의 정규 문자 클래스에서는, 우리는 다음과 같은 집합 연산자들을 위한 문법과 어휘들을 추가하는 것을 제안합니다.

- 차집합 / 뺄셈 (_A 중에서 B가 아닌 것_)
- 교집합 (_A에 포함되면서 B도 포함되는 것_)
- 중첩된 문자 클래스(_위의 연산을 가능하게 하기위해 필요한 것_)

추가적으로, [문자열 속성의 제안](https://github.com/tc39/proposal-regexp-unicode-sequence-properties)에 추가됨으로써 문자열의 특정 유니코드 속성과 문자열 리터럴을 문자 클래스에 추가할 것을 제안합니다.

이 제안서의 JavaScript 개발자를 위한 설명은 [v8.dev의 기능 기사를 참조하세요](https://v8.dev/features/regexp-v-flag).

## 제안 동기

대부분의 정규 표현식 엔진은 수백 개의 범위가 필요하고 유니코드의 새 버전에 따라 변경될 수 있는 하드코딩 문자 클래스를 방지하기 위해 대부분 유니코드 문자 속성을 반영하는 명명된 문자 속성을 지원합니다.

그러나 이러한 문자 속성은 시작점에 불과합니다. 일반적으로 덧셈(결합), 뺄셈(제외), "이것과 저것 모두"(교차)가 필요합니다. [UTS #18: 유니코드 정규 표현식](https://www.unicode.org/reports/tr18/#Subtraction_and_Intersection) 내의 집합 연산들을 지원하기 위한 권장사항을 확인해보세요.

ECMA스크립트 정규식 패턴이 이미 제한된 형태로 하나의 집합 작업을 지원합니다. 이러한 클래스가 '\s' 또는 '\p{Decimal_Number}'와 같은 'CharacterClassEscape'인 경우, 문자들이나 범위, 클래스들의 집합을 만들 수 있습니다.

특정한 집합 연산과 정규 표현식의 질문에 대한 웹 검색은 집합 연산들(몀명된 속성들의 장점을 잃음)이나 주장을 미리 확인(이러한 목적에 맞지않고 성능이 떨어짐) 로부터 발생하는 범위를 하드코딩하는 방식으로 나타납니다.

우리는 중첩된 문자 클래스와 차집합과 교집합들을 위한 문법과 어휘들을 추가하는 것을 제안합니다.

## 제안된 해결책

집합의 차이/뺄셈, 교집합, 중첩된 문자 클래스의 지원을 추가하기 위해 문자 클래스의 문법을 확장하는 것을 제안합니다.

## 고수준 API

정규 표현식 패턴 내에서 다음 기능을 제안합니다.

```
// 차집합
[A--B]

// 교집합
[A&&B]

// 중첩된 문자 클래스
[A--[0-9]]
```

이러한 고차원 예제들을 통해 `A`와 `B`는 (`[a-z]`와 같은) 문자 클래스나 (`\p{ASCII}`와 같은) 속성 이스케이프, 어쩌면 (구체적인 논의에 따른) 단일 문자 또는 문자 범위의 자리 표시자로 간주됩니다. 조금 더 구체적인 실사용 사례는 [실질적인 예제 영역](#illustrative-examples)를 확인하세요.

## 실질적인 예제

정규 문자 클래스와 유사한 문법 규칙을 구현한 ICU의 `UnicodeSet`를 사용한 실 사용 사례 입니다.(`[:POSIX syntax for properties:]`와 `UnicodeSet` 모두가 아닌 '\p{Perl for properties}'를 사용하도록 수정되었습니다.)

- ASCII가 아닌 숫자를 ASCII 숫자로 변환하는 코드

    ```
    [\p{Decimal_Number}--[0-9]]
    ```

- 특정 스크립트의 "단어/식별자" 범위 탐색

    ```
    [\p{Script=Khmer}&&[\p{Letter}\p{Mark}\p{Number}]]
    ```

- "줄 바꿈 없는 공백" 탐색

    ```
    [\p{White_Space}--\p{Line_Break=Glue}]
    ```

    ECMAScript는 현재 `\p{Line_Break=…}`를 지원하지 않습니다. 이 것은 단순히 예제입니다.

- ASCII 문자를 제외한 이모지 문자 탐색

    ```
    [\p{Emoji}--[#*0-9]]

    // …or…

    [\p{Emoji}--\p{ASCII}]
    ```

- 비 스크립트인 특수 결합 표기 탐색

    ```
    [\p{Nonspacing_Mark}&&[\p{Script=Inherited}\p{Script=Common}]]
    ```

- ASCII 공간을 제외한 "보이지 않는 문자" 탐색

    ```
    [[\p{Other}\p{Separator}\p{White_Space}\p{Default_Ignorable_Code_Point}]--\x20]
    ```

- 다음으로 시작하는 "각 스크립트의 첫 글자" 탐색

    ```
    [\P{NFC_Quick_Check=No}--\p{Script=Common}--\p{Script=Inherited}--\p{Script=Unknown}]
    ```

    ECMAScript는 현재 [`\p{NFC_Quick_Check=…}`](https://www.unicode.org/reports/tr15/#Quick_Check_Table)를 지원하지 않습니다. 이 것은 단순히 예제입니다.

- 문자, 마크(문자 구분 기호) 또는 십진수인 모든 그리스 코드 포인트 탐색

    ```
    [\p{Script_Extensions=Greek}&&[\p{Letter}\p{Mark}\p{Decimal_Number}]]
    ```

- "Other" 'General_Category'를 제외하되, 후진 제어 문자를 추가한 모든 코드 포인트 탐색

    ```
    [[\p{Any}--\p{Other}]\p{Control}]
    ```

- 구분자를 제외한 할당된 모든 코드 포인트 탐색

    ```
    [\p{Assigned}--\p{Separator}]
    ```

- 할당되지 않은 코드 포인트는 제거하되, RTL 및 아라비아 문자 코드 포인트 탐색

     ```
     [[\p{Bidi_Class=R}\p{Bidi_Class=AL}]--\p{Unassigned}]
     ```

     ECMAScript는 현재 [`\p{Bidi_Class=…}`](https://www.unicode.org/reports/tr44/#Bidi_Class)를 지원하지 않습니다. 이 것은 단순히 예제입니다.


- `General_Category` “Letter”이며 RTL 및 아라비아 문자 코드 포인트 탐색

     ```
     [\p{Letter}&&[\p{Bidi_Class=R}\p{Bidi_Class=AL}]]
     ```

     ECMAScript는 현재 [`\p{Bidi_Class=…}`](https://www.unicode.org/reports/tr44/#Bidi_Class)를 지원하지 않습니다. 이 것은 단순히 예제입니다.

- 형식 및 제어 문자(또는 동등하게 모든 대리, 개인 사용 및 할당되지 않은 코드 포인트)를 제외한 “Other” `General_Category` 내의 모든 문자 탐색

    ```
    [\p{Other}--\p{Format}--\p{Control}]
    ```

## 자주 묻는 질문들

### 새 문법이 이전 버전과 호환되나요? 다른 정규식 플래그가 필요한가요?

이전 버전에 대한 호환성을 깨지 않는 것이 이 제안의 명시적인 목표입니다. 구체적으로, 우리는 현재 예외가 발생하지 않는 정규 표현식 패턴의 동작을 변경하고 싶지 않습니다. 새로운 문법이 사용 중임을 나타내는 어떤 방법이 필요합니다.

우리는 네 개의 선택지를 고려하였습니다.

- 표현식 외부에 자체적인 새로운 플래그

- 표현식 내부에 `L`은 하나의 ASCII 문자인 `(?L)` 형식의 지시자

- 현재 `u` 플래그(유니코드 모드)에서는 유효하지 않는 접두사 `\U…`. 하지만 `u` 플래그가 아닌 `\U`는 `U`와 동일하다는 것을 기억하세요.
        - `u` 정규 표현식 내에서 알려지지 않는 이스케이프 문자열의 사용을 금지하는 것은 [의식적인 선택](https://web.archive.org/web/20141214085510/https://bugs.ecmascript.org/show_bug.cgi?id=3157) 이었습니다. 이는 확장의 종류를 가능하게 하였습니다.

- _flag와 상관없이_ 존재하는 패턴들에서 유효하지 않은 [`(?[`](https://github.com/tc39/proposal-regexp-set-notation/issues/39)와 같은 접두사


접두사를 사용하는 아이디어는 TC39 초기 회의에서 제안되었기 때문에, 우리는 다음과 같은 다양한 방식으로 작업하고 있었습니다.

```
UnicodeCharacterClass = '\UniSet{' ClassContents '}'
```

그러나 이 것은 개발자 친화적이지 않다는 것을 발견하였습니다.

특히, 이 것은 접두사와 `u`플래그를 작성해야만 합니다. Waldemar는 전치사가 이것 만으로도 충분하게 보일 수 있고, 그러므로 개발자가 'u' 플래그를 추가하는 것을 생략하는 실수를 할 수 있다고 지적했습니다. 이러한 측면은 `u` 플래그를 사용하지 않고 현재 유효하지 않은 (`(?[`와 같은) 더 복잡한 전치사를 사용하여 해결할 수 있으나, 이는 가독성이 떨어지게 됩니다.

또한 백슬래시 문자 접두사의 사용은 `{중괄호}`에 새로운 문법을 포함시킬 수 있습니다. (`\p{property}`, `\u{12345}`와 같은) 다른 특정 문법은 문자 클래스의 가장 바깥쪽 계층이 이상하게 보이므로 `[대괄호]` 대신 중괄호를 사용하기 때문입니다.

마지막으로 식 내부에 여러 개의 새로운 문법 문자 클래스가 있는 경우 각 클래스에 접두사를 사용해야 하는데, 이는 투박합니다.

표현식 내의 지시자는 매력적인 대안이지만, ECMAScript는 아직 이러한 지시자를 사용하지 않았습니다.

따라서 새 플래그는 새 문자 클래스 문법을 나타내는 가장 간단하고 사용자 친화적이며 구문론적으로 가장 깨끗한 방법입니다. 이 것은 반드시 `u` 플래그를 암시하고 함께 사용되어야 합니다.

`u`의 다음 문자인 `v` 플래그를 제안합니다.

또한 [문자열의 속성](https://github.com/tc39/proposal-regexp-unicode-sequence-properties)에서 동일한 새로운 플래그를 사용하는 것을 제안합니다.

즉, 새로운 플래그는 속성 및 문자 클래스와 관련된 몇 가지 연결된 변경 사항을 나타냅니다.

- 문자열의 속성
- 문자 클래스에 문자열 리터럴 또는 특정 속성을 통해 다중 문자 문자열 요소가 포함될 수 있습니다.
- 중첩된 클래스
- 집합 연산자
- 대괄호와 띄어쓰기의 단순한 파싱
- [개선된 IgnoreCase 매칭](https://github.com/tc39/proposal-regexp-set-notation/issues/30)

더 많은 의논을 위해서 [issue 2](https://github.com/tc39/proposal-regexp-set-notation/issues/2)를 확인하세요.

### `v`플래그와 `u`플래그의 차이점이 무엇인가요?

이 질문에 대한 답은 기존의 'u' RegExp를 'v'를 사용하도록 "업그레이드"할 때 유용할 수 있습니다. 차이점은 다음과 같습니다.

1. (명백한 부분입니다.) 새로운 문법의 사용으로 만들어진 이전의 유효하지 않은 패턴들을 유효하게 할 수 있습니다. 예제는 다음과 같습니다.

     ```
    [\p{ASCII_Hex_Digit}--[Ff]]
    \p{RGI_Emoji}
    [_\q{a|bc|def}]
    ```
2. 이전의 특정 유효한 패턴들, 이스케이프되지 않은 [특정 문자](https://arai-a.github.io/ecma262-compare/snapshot.html?pr=2418#prod-ClassSetSyntaxCharacter) `(` `)` `[` `{` `}` `/` `-` `|` (`\` and `]`는 문자 클래스 내부에서 이스케이핑이 필요하지만, `u` 플래그에서는 항상 참입니다.) 또는 [두개의 구두점](https://arai-a.github.io/ecma262-compare/snapshot.html?pr=2418#prod-ClassSetReservedDoublePunctuator)를 포함한 경우 에러를 발생시킵니다.

    ```
    [(]
    [)]
    [[]
    [{]
    [}]
    [/]
    [-]
    [|]
    [&&]
    [!!]
    [##]
    [$$]
    [%%]
    [**]
    [++]
    [,,]
    [..]
    [::]
    [;;]
    [<<]
    [==]
    [>>]
    [??]
    [@@]
    [``]
    [~~]
    [^^^]
    [_^^]
    ```

3. `u` 플래그는 대소문자 구분 없이 매칭되는 동직으로 인해 어려움을 겪습니다. `v`플래그는 문법이 개선되었습니다. 자세한 사항은 [the explainer](https://v8.dev/features/regexp-v-flag#ignoreCase) 또는 [issue #30](https://github.com/tc39/proposal-regexp-set-notation/issues/30)을 확인하세요.


## 다른 정규식의 특징의 선례는 무엇인가요?

몇몇 다른 정규식 엔진은 제안된 확장의 일부 또는 전부를 특정 형식으로 지원합니다.

| 언어/구현사항                                                                                                                      | 합집합 | 뺄셈      | 차집합 | 중첩된 클래스 | 문법적 차이 |
| -------------------------------------------------------------------------------------------------------------------------------------------- | ----- | ---------------- | ------------ | -------------- | -------------------- |
| [ICU regex](https://unicode-org.github.io/icu/userguide/strings/regexp.html#set-expressions-character-classes)                               | ✅    | ✅               | ✅           | ✅             | ❌                   |
| [`java.util.regex.Pattern`](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/util/regex/Pattern.html)                       | ✅    | 🤷 <sup>\*</sup> | ✅           | ✅             | ❌                   |
| [Perl (“experimental feature available starting in 5.18”)](https://perldoc.perl.org/perlrecharclass#Extended-Bracketed-Character-Classes)    | ✅    | ✅               | ✅           | ✅             | ✅                   |
| [.Net](https://docs.microsoft.com/en-us/dotnet/standard/base-types/character-classes-in-regular-expressions#CharacterClassSubtraction)       | ✅    | ✅               | ❌           | ✅             | ❌                   |
| [XML Schema](https://www.w3.org/TR/xmlschema-2/#charcter-classes)                                                                            | ✅    | ✅               | ❌           | ✅             | ❌                   |
| [Apache Xerces2 XPath regex](https://xerces.apache.org/xerces2-j/javadocs/xerces2/org/apache/xerces/impl/xpath/regex/RegularExpression.html) | ✅    | ✅               | ✅           | ✅             | ❌                   |
| [Python regex module](https://pypi.org/project/regex/) (not built-in "re")                                                                   | ✅    | ✅               | ✅           | ✅             | ✅                   |
| [Ruby Regexp](https://docs.ruby-lang.org/en/2.0.0/Regexp.html#class-Regexp-label-Character+Classes)                                          | ✅    | ❌               | ✅           | ❌             | ❌                   |
| ECMAScript prior to this proposal                                                                                                            | ✅    | ❌               | ❌           | ❌             | ❌                   |
| ECMAScript with this proposal

<sup>\*</sup> 뺄셈은 부정의 교집합으로서 [명시](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/util/regex/Pattern.html#subtraction1)되었습니다. 오직 부정과 중첩된 클래스에 대한 지원으로 교집합과 차집합 `[^[^ab][^cd]] === [[ab]&&[cd]]` and `[^[^ab][cd]] === [[ab]--[cd]]`에 대한 함수적 동일성을 구현할 수 있습니다. 이것은 쉽게 읽혀지지 않습니다. 이러한 이유로, 우리의 제안은 교집합과 차집합에 대한 전용 구문도 포함합니다.


연산 순서와 같은 문법적인 차이점에 대한 레퍼런스는 아래와 같습니다.

- [regular expression flavors that support character class subtraction](https://www.regular-expressions.info/charclasssubtract.html)
- [regular expression flavors that support character class intersection](https://www.regular-expressions.info/charclassintersect.html)

특정 Stack Overflow 회의입니다.

- [#3201689](https://stackoverflow.com/q/3201689/96656)
- [#10777728](https://stackoverflow.com/q/10777728/96656)
- [#15930181](https://stackoverflow.com/q/15930181/96656)
- [#17327765](https://stackoverflow.com/q/17327765/96656)
- [#29859968](https://stackoverflow.com/q/29859968/96656)
- [#44771741](https://stackoverflow.com/q/44771741/96656)
- [#55095497](https://stackoverflow.com/q/55095497/96656)


## 이 것은 순열 속성 제안으로 불리는 문자열 속성과 어떻게 상호작용을 하나요?

2단계로 가는 과정에서 두 제안 사이의 상호작용을 설명하였습니다. (그 배경은 [issue #3](https://github.com/tc39/proposal-regexp-set-notation/issues/3)을 참고하세요.)

새로운-구문-문자 클래스가 (클래스 내부에서 사용되는 문자열-속성이나 문자 리터럴로부터 오는)다중-문자-문자열 요소를 포함할 수 있도록 할 뿐만 아니라 문자열-속성을 가능하게 하기 위한 새로운 플래그가 필요하다는 것을 제안합니다.

## 문자열 속성이 문자의 속성으로 변경될 수 있습니까, 아니면 그 반대입니까?

단답으로는 아닙니다.

길게 말하자면, 우리는 2019년 5월 유니코드 기술 위원회(UTC)에 이 문제를 제기하였고 ([L2/19-168](https://www.unicode.org/cgi-bin/GetMatchingDocs.pl?L2/19-168)와 [meeting notes](https://www.unicode.org/L2/L2019/19122.htm#:~:text=45-,B.13.8%20Supporting,Action%20Item%20for,-Mathias))를 참고하세요.) 이후(2021년 4월)에 구체적인 새로운 안정성 정책을 제안하였습니다. ([L2/21-091](https://www.unicode.org/cgi-bin/GetMatchingDocs.pl?L2/21-091)와 [meeting notes](https://www.unicode.org/L2/L2021/21066.htm#:~:text=D.2%20Stability,C11%5D%20Consensus)를 참고하세요.)
UTC는 우리의 제안을 승인하기로 합의하였습니다. 규범적이거나 정보적인 유니코드 속성의 도메인은 변경되지 않아야합니다. 특히 문자의 속성을 문자열의 속성으로 변경해서는 안되며, 그 반대의 경우도 마찬가지입니다.


## 속성 또는 문자열 클래스가 무한 문자열 집합과 일치할 수 있습니까?

단답으로는 아닙니다.

이 제안서는 단순히 [문자열 속성의 제안](https://github.com/tc39/proposal-regexp-unicode-sequence-properties)이며, 특정 문자열의 유한하고 명확한 문자열 집합(`Basic_Emoji`는 많은 단일 문자에 적용될 수 있습니다.)으로 확장되는 속성에 대한 지원을 추가합니다. 그리고 이 제안은 유한한 집합을 만드는 명시적으로 열거된 문자열을 가진 문자 클래스에 대한 구문을 추가합니다. 이것은 문자의 유한한 속성과 문자의 유한한 클래스/집합에서 자연스럽게 확장된 것입니다.

예를 들어, UTS #51에서는 다음과 같은 매우 분명한 차이가 있습니다.

1. 무한 문자열 집합과 일치하는 *정규식을 통해 정의된* [이모지 zwj 순열](https://www.unicode.org/reports/tr51/#def_emoji_zwj_sequence)
2. *데이터 파일에 나열된 문자열의 유한집합*인 (the RGI_Emoji_ZWJ_Sequence 속성과 동일한) [RGI 이모지 ZWJ 순열 집합](https://www.unicode.org/reports/tr51/#def_emoji_ZWJ_sequences)

무한한 문자열 집합, 즉 일종의 명명된 하위-정규 표현식에 대해 명명된 일치에 대한 지원은 이론적으로 가능합니다. 이는 분명히 이 제안의 일부가 아니며, 그러한 가상 표현의 구문과 의미론에 대한 어떠한 추측도 이 제안의 일부가 아닙니다.

향후 광범위한 확장을 가능하게 할 수 있는 예비 구문(중괄호와 같은)이 충분히 있지만, 제안된 사양 변경에 추가할 계획은 없습니다.


## 문자열이 포함된 문자 클래스의 일치 순서는 어떻게 됩니까?

이 제안은 가장 긴 문자열이 가장먼저 일치되도록 보장합니다. 그래서 `'xy'`와 같은 접두사는 `'xyz'`와 같은 긴 문자열을 숨기지 않습니다. 예를 들어 `[a-c\q{W|xy|xyz}]` 패턴을 `'a'`, `'b'`, `'c'`, `'W'`, `'xy'`, 그리고`'xyz' 문자열에 대해 적용합니다. 이 패턴은 `xyz|xy|a|b|c|W`이나 `xyz|xy|[a-cW]`과 같이 동작합니다.

가장 긴 문자열을 먼저 일치시키는 것은 `\p{RGI_Emoji}`와 같은 문자열 속성과의 통합에서 가장 중요합니다. 유니코드 속성은 수학적 의미의 문자/문자열 집합을 정의합니다. 그러므로 `[\p{RGI_Emoji}--\q{🇧🇪}]`와 같은 순서가 존재하지 않은 문자열에 대해 보존할 수 있는 문자열의 순서는 없습니다.

가장 긴 문자열을 처음 일치하는 이론적 근거에 대한 자세한 내용은 [issue #25](https://github.com/tc39/proposal-regexp-set-notation/issues/25)를 확인하세요.

문자 클래스에는 동일한 길이의 문자열이 여러 개 포함될 수 있습니다. `[xyz]`는 단일 문자로 구성된 3개의 문자열을 포함하며, (새로운 문자열 문법을 사용한) `[\q{xx|yy|zz}]`은 두 문자로 구성된 3개의 문자열을 포함합니다. 동일한 길이의 문자열에 대해 고유하거나 관찰 가능한 일치 순서가 없습니다. 위원회는 논의하였고 문자 클래스는 내재된 순서가 존재하지 않는 수학적 집합이라고 결론지었습니다. 마찬가지로, `[xyz]`와 `[zyx]` 사이의 명백한 일치 순서가 존재하지 않으며, `[\q{xx|yy|zz}]`와 `[\q{zz|yy|xx}]` 사이에도 일치 순서가 존재하지 않습니다. 이러한 뉘앙스는 구현자를 집합(수학적 집합의 구현)을 사용할 수 있도록 하며, 런타임 최적화를 시도합니다.


### 문자열의 속성이 원자성을 띄고 있나요?

아니요. 이전의 FAQ에서 보았듯이 `\p{PropertyOfStrings}`는 단일성을 포함하는 [atomic group](https://www.regular-expressions.info/atomic.html)라기보다는 일반적인 단일성에서 분리됩니다. 이 동작은 다음과 같은 이유로 미래 지향적이라고 생각합니다.

만약 분리된 제안의 일부로서 [atomic groups](https://github.com/rbuckton/proposal-regexp-atomic-operators)이 다른 언어의 통사적 선례를 따르는 ECMAScript에 추가된다면, 사용자가 다음과 같이 직접 선택할 수 있습니다.

- 만약 원자성을 가진 동작을 원한다면, `(?>\p{PropertyOfStrings})`을 사용하세요.
- 먼역 원자성을 원하지 않는다면, `\p{PropertyOfStrings}`을 사용하세요.

다른 한편으로 우리가 문자열의 속성의 원자성을 강요한다면, 다른 정규 표현식 종류에 새로운 "비원자" 정규식 연산자를 만드는 것 이외에는 원자성 동작에서 벗어날 수 있는 사용자의 선택은 없을 것입니다.

자세한 것은 [issue #50](https://github.com/tc39/proposal-regexp-set-notation/issues/50)을 참고하세요.


## `B`가 `A`의 적절한 부분집합이 아닌 `A--B`의 경우에 차집합은 어떻게 작용합니까?

이전 질문에 대한 답변에서 언급한 바와 같이, 현재의 ECMAScript 사양 및 다른 정규 표현식의 구현에 따라, **문자 클래스는 수학적 집합입니다.** 따라서, 기존 집합에 없는 문자열을 제거하는 것은 오류가 아니라 no-op입니다. `RGI_Emoji`에는 문자열 `🇧🇪`이 포함되지만 `RGI_Emoji_ZWJ_Sequence`에는 포함되지 않습니다.

```
# Proper subset.
[\p{RGI_Emoji}--\q{🇧🇪}]
# Not a proper subset.
[\p{RGI_Emoji_ZWJ_Sequence}--\q{🇧🇪}]
```

이러한 패턴 중 하나에 대한 예외가 발생한다면 혼란스럽고 역효과를 발생할 수 있습니다.

[이 설명자의 실제 사례](https://github.com/tc39/proposal-regexp-set-notation#illustrative-examples) 중 일부 이 유용한 `A--B` 패턴을 사용하고 있으며, 우리가 지원하는 것은 매우 중요합니다. [배경에 대해서는 issue #32를 보세요.](https://github.com/tc39/proposal-regexp-set-notation/issues/32)


### 대칭차는 무엇입니까?

대칭차에 대한 연산자를 제안하는 것 또한 고려하였습니다. ([issue #5](https://github.com/tc39/proposal-regexp-set-notation/issues/5)를 확인해보세요) 하지만, 좋은 사례를 발견하지 못했고 제한을 단순하게 유지하고자 하였습니다.

그 대신, 추후 사용을 위해 쌍 ASCII 구두점과 기호를 예약할 것을 제안합니다. 이는 대칭차에 대한 [UTS \#18](https://www.unicode.org/reports/tr18/#Subtraction_and_Intersection)의 제안과 같이 `~~`를 추가하는 이후의 제안을 허용합니다.


### 이 제안이 ECMAScript 렉싱에 영향을 주나요?

아니요. 이 제안 이전의 올바른 ECMAScript 렉서는 이 제안 이후에도 올바른 ECMAScript 렉서로 남는다는 것이 우리 제안의 명시적인 목표입니다.

## TC39 회의록 및 슬라이드

- [11월 2020](https://github.com/tc39/notes/blob/master/meetings/2020-11/nov-18.md#adopting-unicode-behavior-for-set-notation-in-regular-expressions) ([slides](https://docs.google.com/presentation/d/1kroPvRuQ8DMt6v2XioFmFs23J1drOK7gjPc08owaQOU/edit))
- [1월 2021](https://github.com/tc39/notes/blob/master/meetings/2021-01/jan-28.md#regexp-set-notation-for-stage-1) ([slides](https://docs.google.com/presentation/d/1vXlLpf3mEa_8Y-GDiRKLCqSzNXPOKWCF7tPb0H2M9hk/edit))
- [3월 2021](https://github.com/tc39/notes/blob/master/meetings/2021-03/mar-10.md#regexp-set-notation-update) ([slides](https://docs.google.com/presentation/d/1dWEHdfSsWPwoln5RD2dwnBomIIytX20UFzYSnmRsIPY/edit))
- [4월 2021 인큐베이터 호출](https://github.com/tc39/incubator-agendas/blob/master/notes/2021/04-08.md) ([slides](https://docs.google.com/presentation/d/1H2Doh8gbsRCthUoKCgFS-mTKxG2qY0vfC6D0dKvv4kc/edit))
- [4월 2021](https://github.com/tc39/notes/blob/master/meetings/2021-04/apr-20.md#regexp-unicode-set-notation--properties-of-strings-update) ([slides](https://docs.google.com/presentation/d/1nV0NHUG5bd201rUSfJinLl8NTmnnyL5gTIhD0llsW1c/edit))
- [5월 2021](https://github.com/tc39/notes/blob/master/meetings/2021-05/may-26.md#regexp-unicode-set-notation--properties-of-strings-for-stage-2) ([slides](https://docs.google.com/presentation/d/1nb_6ZcAjG4AKwVrwpalu1Ep-h7TONxoSm-uxKx83Wik/edit))
- [8월 2021](https://github.com/tc39/notes/blob/master/meetings/2021-08/aug-31.md#regexp-set-notation--properties-of-strings) ([slides](https://docs.google.com/presentation/d/1foloLW13Elu0kslVsmD1hR_qZQBn8INcNpdWl0rlHrI/edit))
- [12월 2021](https://github.com/tc39/notes/blob/master/meetings/2021-12/dec-14.md#regexp-set-notation--unicode-properties-of-strings-ready-for-stage-3-reviews) ([slides](https://docs.google.com/presentation/d/14AWHZvUeaKNHh_b_1xyqVlnDlYW8fWC4Lfgr0lHf2W4/edit))
- [3월 2022](https://github.com/tc39/notes/blob/main/meetings/2022-03/mar-29.md#regexp-set-notation--unicode-properties-of-strings-for-stage-3) ([slides](https://docs.google.com/presentation/d/1_rcjmR2YLZMMB0i4SdZ4RV6eiJB7WmXo_i82_nh7vNg/edit))
- [5월 2023](https://github.com/tc39/notes/blob/main/meetings/2023-05/may-16.md#regexp-v-flag-for-stage-4) ([slides](https://docs.google.com/presentation/d/1pjP06RhOAbYlh-6rYhe3v1SsxD2qLpiWAC2dKDBSJZo/edit))

## 사양

- [Ecmarkup source](https://github.com/tc39/ecma262/pull/2418)
- [HTML 버전](https://arai-a.github.io/ecma262-compare/?pr=2418)

## 구현사항

- [SpiderMonkey/Firefox](https://bugzilla.mozilla.org/show_bug.cgi?id=1713657), shipping in Firefox 116
- [V8/Chrome](https://bugs.chromium.org/p/v8/issues/detail?id=11935), enabled by default in V8 v11.2 / Chrome 112 (behind the `--harmony-regexp-unicode-sets` flag in earlier versions)
- [JavaScriptCore/Safari](https://bugs.webkit.org/show_bug.cgi?id=241593), enabled by default in [Safari Technology Preview 166](https://webkit.org/blog/13964/release-notes-for-safari-technology-preview-166/) & [Safari 17](https://webkit.org/blog/14205/news-from-wwdc23-webkit-features-in-safari-17-beta/)
- [Babel](https://babeljs.io/blog/2022/02/02/7.17.0) via [regexpu-core](https://github.com/mathiasbynens/regexpu-core)
- [ICU class UnicodeSet](https://unicode-org.github.io/icu/userguide/strings/unicodeset.html) can be built from a string with syntax like a regular expression character class. UnicodeSet has long supported set operations and multi-character strings, and recently ([in ICU 70](https://unicode-org.atlassian.net/browse/ICU-21652)) added support for emoji properties of strings.
- [C++ SRELL (`std::regex`-like library)](https://www.akenotsuki.com/misc/srell/en/#ecmascript_regexp)


[HTML `pattern` 속성에서의 `v`플래그](https://github.com/whatwg/html/pull/7908)에 대한 지원은 아래애서 가능합니다.

- [Chrome 114](https://bugs.chromium.org/p/chromium/issues/detail?id=1412729)
- [Firefox 116](https://bugzilla.mozilla.org/show_bug.cgi?id=pattern-v)
- [Safari 17](https://bugs.webkit.org/show_bug.cgi?id=pattern-v)

