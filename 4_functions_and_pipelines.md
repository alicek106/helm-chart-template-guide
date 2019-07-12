# 템플릿 함수와 파이프라인

지금까지 템플릿에 어떻게 데이터를 입력하는지에 대해서 알아보았습니다. 그러나 데이터는 템플릿 내부에 가공되지 않은 채로 전달되죠. 아마 가끔씩은 데이터를 특정 방법으로 가공해 좀 더 사용하기 쉽도록 만들고 싶을 것입니다.

템플릿 작성의 좋은 습관 (Best Practice) 하나를 예시로서 간단히 설명해 보겠습니다. `.Values` 오브젝트에서 템플릿으로 값을 주입할 때, 문자열은 따옴표 처리 하는 것이 좋습니다. 따옴표 처리는 템플릿 지시자 중 `quota` 라는 함수를 통해서 간단히 처리할 수 있습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ quote .Values.favorite.drink }}
  food: {{ quote .Values.favorite.food }}
```

템플릿 함수는 `함수이름 인자1 인자2...` 와 같은 형태의 문법을 따릅니다. 위의 예시에서는 `quote .Values.favorite.drink` 와 같은 방식으로 사용했는데, 이는 `quote` 함수에 파라미터 하나만 전달한 셈입니다.

Helm은 대략 60개 이상의 함수를 지원합니다. 그러한 함수들 중 일부는 [Go template language](https://godoc.org/text/template) 자체에 의해 정의되는 것도 있죠. 몇몇 함수는 [Sprig template library](https://godoc.org/github.com/Masterminds/sprig) 의 일부인 것도 있습니다. 이러한 것들에 대해서는 예제를 계속 다뤄보며 배워보도록 하겠습니다.

> 가끔식 "Helm 템플릿 언어" 가 Helm에서만 사용하는 뭔가 특수한 것처럼 느껴질 때도 있을 것입니다. 그렇지만 사실 Helm 템플릿 언어는 1. Go 템플릿 언어, 2. 여러 함수들, 3. 그리고 오브젝트 값들을 템플릿에 제공하기 위한 다양한 Wrapper들로 구성되어 있습니다. GO 템플릿에서 사용 가능한 다양한 리소스들을 알아두면 Helm 템플릿 언어를 배우는 데에 도움이 될지도 모릅니다.

## 파이프라인

템플릿 언어의 가장 강력한 기능 중 하나는 _pipelines_ 을 이용한 파이프라인 입니다. 파이프라인은 여러 개의 템플릿 명령어를 일련의 변형된 형태로 단순하게 표현하기 위한 연결 도구라고 생각하면 됩니다. 쉽게 이해하자면, 파이프라인은 여러 개의 동작을 순서대로 표현할 수 있는 효율적인 방법이라고 생각하면 됩니다. 위에서 다뤘던 예시를 파이프라인을 통해 좀 더 간단하게 다시 작성해보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | quote }}
```

이 예시에서는 `quote 인자` 처럼 사용하는 대신, {{ }} 내부의 순서를 조금 바꿨습니다. 위 예시에서는 함수에 인자를 파이프라인 (`|`) 을 통해 전달했습니다. 파이프라인을 사용하면 여러 개의 함수를 연달아 사용할 수도 있습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
```

> {{ }} 내부의 순서를 뒤집는 것은 템플릿에서는 흔한 일입니다. 앞으로 여러분은 `quote .val` 보다는 `.val | quote` 를 좀 더 자주 보게 될 것입니다. 그렇지만 딱히 정답이 정해져 있는 것은 아니며, 어느 것을 사용하더라도 상관은 없습니다.

위의 템플릿 값이 실제로 변환되면 아래와 같이 템플릿이 렌더링됩니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: trendsetting-p-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
```

`pizza` 라는 값이 `"PIZZA"` 로 변형되었습니다!

위와 같이 인자들을 파이프라인에 넣을 때, 첫 번째 값이 처리된 결과 (`.Values.favorite.drink`) 값은 _함수의 마지막 인자_로서 전달됩니다. 이를 좀 더 직관적으로 이해하기 위해, 위의 예시에서 {{ }} 내부의 값이 두 개의 인자를 전달할 수 있도록 조금 바꿔보겠습니다. 아래처럼 `repeat COUNT STRING` 을 사용해 보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink | repeat 5 | quote }}
  food: {{ .Values.favorite.food | upper | quote }}
```

`repeat` 함수는 주어진 인자 값을 N번 만큼 출력 (echo) 합니다. 즉, 최종적으로는 아래와 같이 렌더링됩니다. 

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: melting-porcup-configmap
data:
  myvalue: "Hello World"
  drink: "coffeecoffeecoffeecoffeecoffee"
  food: "PIZZA"
```

## `default` 함수 사용하기

템플릿에서 꽤 자주 사용되는 함수는 `default` 입니다. 이는 `default [기본 값] [주어진 값]` 형태로 사용됩니다. 이 함수는 템플릿 내부에서 값이 생략되었을 때 기본 값을 설정하는 기능을 제공합니다. `default` 함수를 이용해 위에서 사용한 예시를 좀 더 바꿔보도록 하죠.

```yaml
drink: {{ .Values.favorite.drink | default "tea" | quote }}
```

이전처럼 그냥 템플릿을 렌더링해 보면, 아래와 같이 템플릿이 출력됩니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: virtuous-mink-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
```

자, 그렇다면 `values.yaml` 파일에서 drint 라는 값을 삭제해 보겠습니다.

```yaml
favorite:
  #drink: coffee
  food: pizza
```

다시 `helm install --dry-run --debug ./mychart` 명령어를 실행해 보면 아래와 같이 렌더링됩니다.

```yaml
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fair-worm-configmap
data:
  myvalue: "Hello World"
  drink: "tea"
  food: "PIZZA"
```

실제 차트를 사용할 때는 모든 정적 값들이 values.yaml 파일 내부에 정의되어 있어야 합니다. 그러나, `default` 명령어는 values.yaml 파일 내부에서 선언될 수 없는, 예를 들어 뭔가 프로세싱된 값들에 대해서는 사용하기에 적합하죠. 아래의 예시처럼 말입니다.

```yaml
drink: {{ .Values.favorite.drink | default (printf "%s-tea" (include "fullname" .)) }}
```

몇몇 경우에는 `if` 조건문이 `default` 보다 더 효율적일 수도 있습니다. 이에 대해서는 다음 장에서 알아보도록 하죠.

## 오퍼레이터 (Operator) 와 함수

오퍼레이터는 불린 값을 반환하는 함수로서 구현되어 있습니다. `eq`, `ne`, `lt`, `gt`, `and`, `or`, `not` 와 같은 지시자를 사용하기 위해서는 함수를 사용하던 것과 마찬가지로 여러 개의 파라미터 앞에 오퍼레이터 지시자를 사용하면 됩니다. 여러 개의 오퍼레이터를 연달아서 사용하려면 괄호를 사용해 명시적으로 각 오퍼레이터를 구분해야 합니다.

```yaml
{{/* .Values.fooString의 값이 존재하고, "foo" 라는 값으로 설정되었을 때 이하의 내용을 포함시킵니다. */}}
{{ /* 역주 : and 함수의 인자가 .Values.fooString과 (eq .Values.fooString "foo") 라고 이해하면 됩니다. */}}
{{ /* 그리고 그 결과값이 if로 전달됩니다. */}}
{{ if and .Values.fooString (eq .Values.fooString "foo") }}
    {{ ... }}
{{ end }}
```

```
{{/* 설정되지 않은 값은 false로 변환되며, .Values.setVariable이 not 함수를 통해 반전되었으므로 (true -> false) 이하의 내용이 포함되지 않습니다. */}}
{{ if or .Values.anUnsetVariable (not .Values.aSetVariable) }}
   {{ ... }}
{{ end }}
```



함수와 파이프라인을 사용해 보았으니, 다음 장에서는 조건문과 반복문, 그리고 범위 지시자 (scope modifiers) 에 대해 사용해 보겠습니다.

이 문서의 원본 리포지터리 커밋 : [63c970c](https://github.com/helm/helm/commit/63c970c5ce29b0971cdc6409d9c0e156321ea32a) (on 28 Feb 2019)


