# Flow Control (흐름 제어)

템플릿에서 보통 "actions (액션)" 라고 불리는 것은 템플릿 생성의 흐름을 제어할 수 있는 기능을 제공합니다. Helm 템플릿 언어는 다음의 흐름 제어 구문을 제공하고 있습니다.

- 조건문 사용을 위한 `if`/`else` 구문
- 영역 지정을 위한 `with` 구문
- for 방식의 반복문 사용을 위한 `range` 구문

이러한 것 외에도, 이름이 부여된 (named) 템플릿 부문에 대해서 아래의 액션을 제공합니다.

- 템플릿 안에서 이름이 부여된 (named) 템플릿을 정의해주는 `define` 구문
- 이름이 부여된 (named) 템플릿을 import할 수 있는 `template` 구문
- 채워 넣을 수 있는 (fillable) 템플릿 영역을 정의를 의미하는 `block` 구문

이번 가이드에서는 `if`, `with`, `range`에 대한 사용 방법을 알아보겠습니다. 다른 구문들에 대해서는 "Named Templates" 가이드 챕터에서 다룰 예정입니다.

## If/Else 구문

우리가 살펴볼 첫 번째 흐름 제어 구문은 템플릿에서 if/else 구문을 이용해 텍스트 값을 템플릿에 포함시키는 것입니다. 가장 기본적인 조건문 구조는 아래와 같습니다.

```yaml
{{ if PIPELINE }}
  # 뭔가 씁니다.
{{ else if OTHER PIPELINE }}
  # 뭔가 다른 내용을 씁니다.
{{ else }}
  # 기본 값을 씁니다.
{{ end }}
```

위의 if 조건문에서 다루고 있는 것은 특정 값이 아닌 _pipelines_ 라는 점에 유의합니다. 값이 아닌 파이프라인을 굳이 조건문에서 사용하는 이유는 흐름 제어 구문을 통해서도 파이프라인을 실행할 수 있다는 것을 보여주기 위함입니다.

파이프라인은 아래의 값들로 평가되었을 경우에 한해서 _false_ 로 취급됩니다.

- 불린 값 false
- 숫자 0
- 비어 있는 문자열
- `nil` (비어 있거나 null인 경우)
- 비어 있는 컬렉션 (`map`, `slice`, `tuple`, `dict`, `array`)

위를 제외한 나머지 경우에는 전부 _true_ 로 취급되며, 파이프라인이 실행될 것입니다.

컨피그맵을 이용해 간단한 조건문을 정의해 보겠습니다. if문을 이용해 drink 변수의 값이 coffee로 설정되어 있으면 다른 설정 값을 추가하는 예시를 살펴보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{ if and .Values.favorite.drink (eq .Values.favorite.drink "coffee") }}mug: true{{ end }}
```

`.Values.favorite.drink`의 값은 어딘가에 정의되어 있어야 합니다. 그렇지 않으면 coffee라는 값과 비교할 때 에러를 출력할 것입니다. 이전 가이드 챕터에서 `drink: coffee` 를 주석처리했기 때문에, 렌더링된 최종 템플릿은 `mug: true` 라는 플래그를 포함하고 있지 않을 것입니다. 그렇지만 `values.yaml` 파일에서 다시 주석처리된 라인을 주석 해제한다면, 템플릿의 렌더링된 결과물은 아래와 같을 것입니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eyewitness-elk-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
```

## Controlling Whitespace (공백 사용하기)

조건문을 사용하려면 템플릿에서 공백이 어떻게 사용되는지도 함께 알아두는 것이 좋습니다. 위의 예시를 좀 더 읽기 쉽게 다시 작성해 보겠습니다.

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{if eq .Values.favorite.drink "coffee"}}
    mug: true
  {{end}}
```

새롭게 고친 템플릿은 읽기에 좋아보이긴 하지만, 실제로 템플릿 렌더링 엔진에 전달하면 아래와 같은 에러를 출력할 것입니다.

```console
$ helm install --dry-run --debug ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
Error: YAML parse error on mychart/templates/configmap.yaml: error converting YAML to JSON: yaml: line 9: did not find expected key
```

도대체 뭐가 잘못된 것일까요? 

위의 YAML 파일에서는 공백을 잘못 사용했기 때문에 올바르지 않은 YAML 파일이 출력되었습니다. 아래의 YAML 파일을 보겠습니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: eyewitness-elk-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
    mug: true
```

`mug` 의 라인에서 인덴트가 잘못 입력되었습니다. 이를 수정하기 위해서 단순히 아래와 같이 인덴트를 수정하고 다시 템플릿 엔진에 전달해 보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{if eq .Values.favorite.drink "coffee"}}
  mug: true
  {{end}}
```

위의 YAML 파일은 유효하지만, 실제로 템플릿이 렌더링되면 조금 웃기게 결과가 출력됩니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: telling-chimp-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"

  mug: true

```

위의 YAML 파일에서는 빈 라인이 함께 출력되었습니다. 왜일까요? 템플릿 엔진이 실행될 때, 엔진은 `{{` 와 `}}` 항목을 _삭제합니다_. 그렇지만 남은 공백은 정확히 그대로 남겨두기 때문이죠.

다행히도, Helm 템플릿에는 이를 해결할 수 있는 몇 가지 방법이 존재합니다. 

첫 번째로, 둥그렇게 말린 브레이스 (curly brace) 문법을 사용하면 템플릿 엔진이 공백을 삭제하도록 설정할 수 있습니다. `{{- ` ( {{- 에 공백이 한칸 추가된 것 ) 은 왼쪽의 공백이 사라져야 한다는 것을 의미하며,  ` -}}`는 오른쪽의 공백이 사라져야 한다는 것을 의미합니다. _새로운 라인은 공백으로 취급된다는 것에 유의합니다._

> `-`와 나머지 구문 사이에 공백 하나가 반드시 존재해야 합니다. 예를 들어 `{{- 3 }}`는 왼쪽의 공백을 삭제하고, 3을 출력해라 라는 것을 의미합니다. 그렇지만 `{{-3}}` 는 단순히 -3을 출력해라 라는 것을 의미합니다.

이러한 문법을 사용하면 새로운 라인을 삭제하기 위해서 템플릿을 아래처럼 수정할 수 있습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee"}}
  mug: true
  {{- end}}
```

조금만 더 깔끔하게 사용해보기 위해서, 위의 내용을 조금만 수정해 보겠습니다. 이 규칙에 의해 삭제될 공백을 `*` 로 교체해 보겠습니다. 라인의 마지막에 쓰이는 `*` 는 삭제될 새로운 라인을 의미하는 문자열 (\n) 을 뜻합니다. 

> 역주 : 이 항목은 딱히 의미는 없습니다. 어느 부분이 실제로 삭제되는지를 *를 통해서 나타냈을 뿐입니다. 그냥 참고만 하면 됩니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | default "tea" | quote }}
  food: {{ .Values.favorite.food | upper | quote }}*
**{{- if eq .Values.favorite.drink "coffee"}}
  mug: true*
**{{- end}}

```

이 점을 유의해서, Helm 템플릿을 다시 렌더링 엔진에 전송하면 아래와 같은 결과를 얻을 수 있을 것입니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: clunky-cat-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  mug: true
```

단, 이러한 새로운 라인을 삭제할 때는 주의할 점이 하나 있습니다. 아래와 같이 실수로 사용할 수도 있기 때문입니다.

```yaml
  food: {{ .Values.favorite.food | upper | quote }}
  {{- if eq .Values.favorite.drink "coffee" -}}
  mug: true
  {{- end -}}

```

위의 YAML 템플릿은 `food: "PIZZA"mug:true` 를 출력합니다. 브레이스 양쪽에서 서로 새로운 라인을 삭제해버리기 때문입니다.

> 템플릿에서 공백을 다루는 자세한 방법에 대해서는, [Go 공식 문서를 참고하기 바랍니다.](https://godoc.org/text/template)

마지막으로, 때로는 이러한 템플릿 공백 구문을 사용하는 것 대신에 명시적으로 인덴트를 얼마나 사용할 것인지를 선언하는 것이 더 쉬울 수도 있습니다. 따라서, `indent` 함수 ((`{{indent 2 "mug:true"}}`))를 사용하는 것이 더 쉬울수도 있습니다. 

## `with`를 이용해 범위를 수정하기

다음으로 다뤄볼 제어 구조는 `with` 액션입니다. 이는 변수의 범위 (scope) 를 지정할 때 쓰입니다. 이전에 Values를 사용했을 때 `.` 가 _현재 범위 (scope)_ 를 나타내는 것이었음을 다시 기억해 보겠습니다. 따라서 `.Values` 는 현재 범위에서 `Values` 오브젝트를 찾아서 사용하라는 것을 의미했었습니다.

`with` 문법은 `if` 구문과 비슷합니다.

```yaml
{{ with PIPELINE }}
  # restricted scope
{{ end }}
```

scope는 변할 수 있습니다. `with` 는 현재 scope (`.`) 를 특정 오브젝트로 설정할 수 있도록 지원합니다. 예를 들어, 우리는 지금까지 `.Values.favorites` 에서 작업해 왔었지만, 이번에는 `.` scope가  `.Values.favorites` 를 가리키도록 컨피그맵을 살짝 수정해 보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
```

> 이전에 사용했던 예제에서 `if` 구문을 삭제했다는 사실에 유의합니다.

`with` 구문 안에서 `.drink` 나 `.food` 처럼 사용하고 있다는 사실에 주목합니다. 이는 `with` 구문의 내부에서는 `.` 를 사용하면`.Values.favorite` 를 가리키도록 설정되기 때문입니다. `.`는 `with` 구문의 `{{` 와 `}}` 를 벗어나면 원래대로 돌아갑니다.

그렇지만 주의할 점이 한 가지 있습니다. 제한된 scope 내부에서는 상위 scope의 다른 오브젝트에 접근하지 못합니다. 예를 들어, 아래와 같은 사용은 실패할 것입니다.

```yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  release: {{ .Release.Name }}
  {{- end }}
```

`Release.Name` 은 `.` scope에서 사용할 수 없기 때문에 에러를 출력합니다. 그러나 마지막의 두 라인의 순서를 바꾼다면 의도한대로 잘 동작할 것입니다. 왜냐하면 `{{end}}` 뒤로는 다시 scope가 원래대로 돌아가기 때문입니다.

```yaml
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  release: {{ .Release.Name }}
```

`range` 구문을 살펴본 뒤, 위의 이슈를 해결할 수 있는 다른 해결 방법인 템플릿 변수에 대해서 알아보겠습니다.

## `range` 를 통한 루프

많은 프로그래밍 언어는 `for` 루프를 통한 반복문을 지원합니다. Helm 템플릿 언어에서도 `range` 오퍼레이터로 컬렉션의 값을 통해 반복문을 사용할 수 있습니다.

시작하기 앞서, `values.yaml` 파일에 아래와 같은 피자 토핑 (ㅋㅋ..) 의 목록을 추가해 보겠습니다.

```yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```

템플릿에서 `slice` 라고 불리는 `pizzaToppings` 리스트를 새롭게 정의했습니다. 이제 컨피그맵에서 이 리스트를 출력하도록 템플릿을 조금 수정해 보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}

```

`toppings` 리스트의 내용을 조금 자세히 살펴보겠습니다. `range` 함수는 `pizzaToppings` 리스트를 순회할 것입니다. 그러나 조금 재밌는 사실을 알 수 있는데, 방금 사용해보았던 `with` 가 `.`를 scope로서 새롭게 정의했던 것처럼, `range` 오퍼레이터 또한 `.`를 새롭게 정의한다는 것입니다. 루프를 순회할 때마다, `.`는 현재 피자 토핑의 값으로 설정됩니다. 즉, 첫 번째 순회에서 `.`는 `mushrooms` 로 설정됩니다. 두 번째 순회에서는 `cheese` 로 설정됩니다,

`.` 를 파이프라인으로 직접 보내서 사용할 수도 있습니다. `{{ . | title | quote }}` 처럼 사용하면 `.` 를 `title` 로 보내고 (title case라는 함수입니다), 그리고 다시 `quota` 함수로 보냅니다. 이 템플릿을 렌더링하면 아래와 같은 출력을 볼 수 있을 것입니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: edgy-dragonfly-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"
```

자, 이제 이 예제에서 약간 tricky한 방법을 사용했습니다. `toppings: |-` 라인은 여러 줄을 정의하는 문자열을 의미합니다. 따라서 토핑 리스트의 값은 사실은 실제 YAML 리스트가 아닙니다. 이는 단지 큰 일련의 문자열일 뿐입니다. 왜 이렇게 사용했을까요? 컨피그맵의 `data`는 키-값의 페어로 이루어져 있는데, 이 키와 값은 문자열 값을 가지기 때문입니다. 이것이 무엇을 의미하는지 궁금하다면 [쿠버네티스의 컨피그맵 공식 문서](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)를 참고하기 바랍니다. 지금 당장 우리에게 있어서는 이러한 세세한 것이 크게 중요하지는 않습니다.

> `|-` 는 YAML 파일에서 여러 줄의 문자열을 정의할 때 사용합니다. 이는 위의 예시에서 설명했던 것처럼 여러 문자열로 이루어진 큰 블럭을 YAML 파일에서 정의할 때 매우 유용합니다.

때로는 템플릿 안에서 직접 문자열을 만들고 이를 순회하는 것이 더 편할수도 있습니다. Helm 템플릿은 `list` 라고 불리는 함수를 제공합니다.

```yaml
  sizes: |-
    {{- range list "small" "medium" "large" }}
    - {{ . }}
    {{- end }}
```

위의 템플릿은 아래와 같이 렌더링될 것입니다.

```yaml
  sizes: |-
    - small
    - medium
    - large
```

리스트뿐만 아니라, `range` 는 맵이나 딕셔너리처럼 키와 값을 가지는 컬렉션을 순회할 때 매우 유융하게 사용될 수 있습니다. 다음 가이드 챕터에서는 이를 템플릿 변수로 가져올 때 어떻게 사용할 수 있는지에 대해서 살펴보겠습니다.



이 문서의 원본 리포지터리 커밋 : [b2eed76](https://github.com/helm/helm/commit/b2eed7644ae5bab07925fd1bcf8e42132e769901) (on 5 Jun 2019)
