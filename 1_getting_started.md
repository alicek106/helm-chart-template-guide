# 차트 템플릿 작성 시작하기

이번에는 새로운 차트를 하나 생성한 뒤, 첫 번째 템플릿을 추가해보겠습니다. 이번에 생성하는 차트는 이 가이드 전반에 걸쳐 계속 사용될 것이므로, 삭제하지 말고 보관해두는 것을 추천합니다 :D

## 차트 생성하기

이전의 [Charts Guide](https://github.com/helm/helm/blob/master/docs/charts.md)에서 설명했던 것처럼, Helm 차트는 아래와 같은 구조로 구성되어 있습니다.

```
mychart/
  Chart.yaml
  values.yaml
  charts/
  templates/
  ...
```

'templates/' 는 템플릿 파일이 저장되어 있는 디렉터리입니다. Tiller가 차트를 받아들여 처리할 때, Tiller는 'templates/' 디렉터리 안에 있는 모든 파일을 Helm의 템플릿 렌더링 엔진에게 전송합니다. 그리고 나서 Tiller는 이 템플릿의 렌더링 결과물을 다시 모은 다음, 쿠버네티스로 전송해 실제로 리소스를 생성합니다.

'values.yaml' 파일도 템플릿을 사용하기 위해서는 필수적인 파일입니다. 이 파일은 차트에 사용되는 _기본 값들_ 을 포함하고 있습니다. 이 값들은 사용자가 'helm install' 이나 'helm upgrade' 명령어를 사용할 때 --set 옵션을 통해 직접 재정의해 사용할 수도 있습니다.

'Chart.yaml' 파일은 차트에 대한 명세 (description) 를 저장하고 있습니다. 여러분은 템플릿 내부에서도 이 파일의 내용에 접근할 수가 있습니다. 

'charts/' 디렉터리는 다른 차트들을 포함하고 _있을 수도_ 있습니다. (예를 들어 _subchart_, 즉 하위 차트들과 같은 것들이 이에 해당됩니다. 차트를 생성하면 subchart도 함께 생성됩니다) 이 가이드를 계속 읽다 보면, 이러한 차트나 템플릿들이 템플릿 렌더링 시 어떻게 동작하는지를 이해할 수 있을 것입니다.

## A Starter Chart

'mychart' 라는 이름의 간단한 차트를 새로 하나 만든 뒤, 'mychart' 차트 안에 템플릿을 생성해 보겠습니다.

```console
$ helm create mychart
Creating mychart
```

지금부터 우리는 이 'mychart' 디렉터리 안에서 작업할 것입니다.

### 'mychart/templates/' 디렉터리 훑어보기

`mychart/templates/` 디렉터리 내부를 확인해 보면, 몇 개의 파일이 이미 존재합니다.

- `NOTES.txt`: 여러분의 차트를 설명할 수 있는 'help text' 입니다. 이 파일에는 'helm install' 명령어를 통해 실제로 차트를 쿠버네티스에 생성했을 때 터미널에 출력되는 내용을 담고 있습니다.
- `deployment.yaml`: 쿠버네티스 디플로이먼트를 생성하기 위한 기본 매니페스트 파일입니다.
- `service.yaml`: 디플로이먼트에 접근하기 위한 서비스를 생성하는 기본 매니페스트 파일입니다.
- `_helpers.tpl`: 차트 전반에 걸쳐 재사용할 수 있는 템플릿 헬퍼 (Helper) 를 저장해두는 파일입니다.

그리고 다음으로 여러분이 해야할 것은.. _이 파일들을 전부 지우는 것입니다!_ 그렇게 해야만 아예 처음부터 튜토리얼을 진행하며 템플릿 사용 방법을 학습할 수 있기 때문입니다. 앞으로 가이드를 진행해 나가면서 여러분만의 'NOTES.txt' 파일과 '_helpers.tpl' 파일을 생성할 것입니다.

```console
$ rm -rf mychart/templates/*.*
```

참고로, 여러분이 실제 운영 (production) 단게 의 차트를 작성하고 있다면, 차트들의 기본 버전들을 설정하는 것이 좋습니다. 예를 들어, Github를 통해 각 차트의 버저닝을 하는 것도 나쁘지는 않겠죠.

## 첫 번째 템플릿 작성해보기

우리가 생성해 볼 첫 번째 템플릿은 'ConfigMap' 입니다. 쿠버네티스에서 컨피그맵은 설정 데이터를 저장하기 용도로 사용됩니다. 컨피그맵은 매우 기본적인 리소스이기 때문에, 템플릿을 처음 시작하기로는 적합할 것 같습니다 :D

`mychart/templates/configmap.yaml` 라는 파일을 새롭게 생성해 내용을 작성해 보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

**TIP:** 템플릿 파일의 접미어 (또는 확장자) 는 엄격한 이름 짓기 패턴 (Naming Pattern) 을 따르지는 않습니다. 그렇지만, YAML 파일에 대해서는 '.yaml' 을, 헬퍼 파일에 대해서는 '.tpl' 를 권장합니다.

위의 YAML 파일은 컨피그맵을 생성하기 위한 최소한의 필드만을 포함하고 있는, 기본 뼈대에 불과합니다. 이 파일이 `templates/` 디렉터리 내부에 위치해 있기 때문에, 이 파일은 템플릿 엔진을 통해 쿠버네티스에 전송될 것입니다.

그렇지만 템플릿 형식이 아닌 단순한 (plain) YAML 파일을 'templates/' 디렉터리에 두어도 크게 상관은 없습니다. Tiller가 이 템플릿을 읽어들일 때, 렌더링 없이 그 파일을 있는 그대로 쿠버네티스로 전송할 뿐입니다.

놀랍게도 이 단순한 템플릿만으로도 차트 설치가 가능합니다. 아래와 같이 차트를 설치해 보겠습니다.

```console
$ helm install ./mychart
NAME: full-coral
LAST DEPLOYED: Tue Nov  1 17:36:01 2016
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                DATA      AGE
mychart-configmap   1         1m
```

위의 출력 결과에서, 컨피그맵이 생성된 것을 확인할 수 있습니다. Helm을 이용해 release의 상태를 확인할 수 있으며, 실제로 템플릿이 로드되었음을 볼 수 있습니다.

```console
$ helm get manifest full-coral

---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

'helm get manifest' 명령어는 release 이름 (위의 경우에는 'full-coral') 을 인자로 받아서, 실제로 서버로 업로드된 쿠버네티스 리소스의 모든 정보를 출력합니다. 각 파일은 YAML 파일의 시작을 의미하는 '---' 로 시작하고, 그 뒤의 주석 ('# Source:.. ') 은 해당 YAML 파일이 어떠한 템플릿 파일로부터 생성된 것인지를 의미합니다.

위의 출력으로 미루어보아, 저 YAML 데이터는 우리가 'configmap.yaml' 파일에 저장했던 데이터가 맞는 것 같습니다.

이번에는 `helm delete full-coral` 명령어를 통해 release를 삭제한 뒤 넘어가겠습니다. 뒤에서 다시 생성할 것입니다.

### 간단한 템플릿 요청 추가하기

리소스 내부에 'name:' 과 같이 하드코딩 방식으로 값을 넣는 것은 사실 그다지 좋은 습관은 아닙니다. 이번에는 컨피그맵의 'names:' 필드를 Helm의 release 이름으로 설정해 보겠습니다. Helm의 release 이름 또한 고유하기 때문에, 쿠버네티스에서 리소스 이름이 겹칠 일은 없을 것입니다.

**TIP:** 'names:' 필드는 63개의 문자로 제한되어 있는데, 이는 DNS 시스템의 한계 때문입니다. 이 이유 때문에 release 이름은 사실상 53개의 문자로 제한됩니다.

'configmap.yaml' 파일을 적절하게 바꿔보겠습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```

가장 큰 변화는 'names:' 항목의 값입니다. 위의 예시에서는 '{{ .Release.Name }}-configmap' 으로 바뀌었군요.

> 템플릿의 지시자, 즉 .Release.Name 과 같은 값은 '{{' 와 '}}' 블록 안에서 사용되어야 합니다.

템플릿 지시자인 `{{ .Release.Name }}` 는 release의 이름을 템플릿 내부로 가져옵니다. 템플릿으로 전달되는 이러한 값들은 말하자면 _'네임스페이스에 속하는 오브젝트'_ 라고도 표현할 수가 있겠습니다. 즉, 온점 ('.') 이 각 네임스페이스의 요소를 구분짓는 셈이죠. `{{ .Release.Name }}` 는 Release 네임스페이스의 Name이라는 값이라고 생각해보면 쉬울지도 모르겠습니다. 

'Release' 앞에 붙어 있는 온점은 차트의 범위 중 가장 최상위 네임스페이스에서 시작했음을 의미합니다 (이에 대해서는 나중에 좀 더 자세히 다뤄보도록 하죠). 어쨌든, '.Release.Name' 이라는 값은 "가장 최상위 네임스페이스에서 시작해, 'Release' 라는 오브젝트를 찾아서 'Name' 이라는 값을 사용한다" 라고 해석할 수 있겠습니다. 

'Release' 오브젝트는 사실 Helm에 자체적으로 내장되어 있는 오브젝트 중 하나입니다. 이에 대해서도 나중에 좀 더 깊게 다뤄보도록 하죠. 그러나 지금은 `{{ .Release.Name }}` 가 'Tiller가 release에 할당하는 랜덤한 이름' 을 의미한다는 것만 이해하고 넘어가도 됩니다.

자, 이제 리소스를 설치한 뒤 템플릿 지시자의 결과물을 바로 확인해보도록 하겠습니다.

```console
$ helm install ./mychart
NAME: clunky-serval
LAST DEPLOYED: Tue Nov  1 17:45:37 2016
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                      DATA      AGE
clunky-serval-configmap   1         1m
```

'RESOURCES' 부분에서 컨피그맵의 이름이 `clunky-serval-configmap` 로 바뀐 것을 눈치채셨나요?

생성된 리소스의 YAML 파일 전체를 확인하고 싶다면 'helm get manifest clunky-serval' 명령어를 사용할 수 있습니다.

이번 가이드에서는 템플릿을 사용하는 가장 기본적인 방법인 '{{' 과 '}}' 로 둘러싸인 템플릿 지시자에 대해서 알아보았습니다. 다음 파트에서는 템플릿의 사용 방법에 대해 좀 더 깊게 알아볼 것입니다. 그러나 다음 파트로 넘어가기 전, 템플릿을 좀 더 빠르게 빌드할 수 있는 팁이 있습니다. 템플릿 렌더링을 테스트해야 한다면 (그렇지만 release 생성은 하고 싶지 않다면) `helm install --debug --dry-run ./mychart` 과 같이 Dry Run을 할 수도 있습니다. 이는 Tiller 서버로 차트를 보내 렌더링은 하지만, 차트를 설치하지는 않으며 단순히 렌더링된 템플릿만을 반환합니다.

```console
$ helm install --debug --dry-run ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
NAME:   goodly-guppy
TARGET NAMESPACE:   default
CHART:  mychart 0.1.0
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: goodly-guppy-configmap
data:
  myvalue: "Hello World"

```

'--dry-run' 은 여러분만의 Helm 차트 템플릿을 작성할 때 디버깅을 도와주는 매우 유용한 기능입니다. '--dry-run' 이 문제 없이 잘 동작했다고 해서 차트가 문제없이 잘 동작할 것이라는 생각은 하지 않는게 좋습니다. 어디까지나 템플릿 렌더링에 문제가 없다는 뜻이며, 생성된 템플릿을 쿠버네티스가 오류 없이 잘 생성할지는 모르는 일이니까요.

다음 가이드에서는 위에서 사용해본 기본 차트를 계속 사용해 볼 것입니다. Helm에 자체적으로 내장된 오브젝트들을 사용해 봄으로써, 템플릿 언어를 좀 더 자세히 알아보도록 하죠.



이 문서의 원본 리포지터리 커밋 : c99a3c6 (on 13 Feb 2019)


