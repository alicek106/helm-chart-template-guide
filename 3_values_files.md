# Values Files

이전 장에서는 Helm 템플릿이 자체적으로 제공하는 오브젝트들에 대해서 살펴보았습니다. `Values` 또한 자체적으로 내장되어 있는 오브젝트 중 하나입니다. 이 오브젝트는 차트에 전달되는 값들에 접근하기 위한 용도로 사용됩니다. `Values` 의 값들은 총 4가지 방법을 통해 설정될 수 있습니다.

- 차트 내부에 위치한 `values.yaml` 파일
- 서브 차트가 존재할 경우, 상위 차트의 `values.yaml` 파일
- `helm install` 명령어나 `helm upgrade` 명령어에서 `-f` 옵션을 통해 전달된 values 파일 (ex: `helm install -f myvals.yaml ./mychart`)
- `--set` 옵션을 통해 설정된 개별적인 인자들 (`helm install --set foo=bar ./mychart` 와 같이 사용 가능)

위 목록은 값의 우선순위대로 나열한 것입니다. 즉, [1. values.yaml < 2. 상위 차트의 values.yaml < 3. helm install -f 에 지정된 파일 < 4. --set 옵션에 설정되는 값] 순서대로 우선순위를 갖습니다. 따라서 `--set` 옵션이 가장 큰 효력을 가지며, `--set` 을 통해 값을 설정하면 우선순위가 낮은 값들은 모두 덮어 씌워집니다.

어쨌든, Values 파일은 기본적으로 단순한 YAML 파일에 불과합니다. 이번에는 `mychart/values.yaml` 파일을 먼저 수정해본 뒤, ConfigMap의 템플릿도 변경해 보겠습니다.

`values.yaml` 파일에 기본적으로 설정된 값들을 모두 삭제한 뒤, 하나의 값만을 설정해 보겠습니다.

```yaml
favoriteDrink: coffee
```

그 뒤, 템플릿 내부에서 아래와 같이 사용할 수 있습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favoriteDrink }}
```

마지막 줄에서 `favoriteDrink` 라는 값을 `Values`: `{{ .Values.favoriteDrink }}` 라는 속성을 통해 접근했다는 것에 주목합니다.

자, 그렇다면 최종적으로 어떻게 값이 렌더링되는지 확인해 보겠습니다.

```console
$ helm install --dry-run --debug ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
NAME:   geared-marsupi
TARGET NAMESPACE:   default
CHART:  mychart 0.1.0
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: geared-marsupi-configmap
data:
  myvalue: "Hello World"
  drink: coffee
```

`favoriteDrink` 라는 값이 `values.yaml` 파일에 `coffee` 라는 값으로 정의되었기 때문에, 그 값이 템플릿에 잘 들어가 있습니다.
이러한 변수 값은 `helm install` 명령어에서 `--set` 옵션을 통해서도 쉽게 설정할 수 있습니다. 물론, values.yaml 파일 등에서 설정한 값들은 모두 무시됩니다.

```console
$ helm install --dry-run --debug --set favoriteDrink=slurm ./mychart
SERVER: "localhost:44134"
CHART PATH: /Users/mattbutcher/Code/Go/src/k8s.io/helm/_scratch/mychart
NAME:   solid-vulture
TARGET NAMESPACE:   default
CHART:  mychart 0.1.0
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: solid-vulture-configmap
data:
  myvalue: "Hello World"
  drink: slurm
```

`--set` 옵션은 `values.yaml` 파일보다 우선순위를 갖기 때문에, 위의 예시에서 템플릿의 값에는 `drink: slurm` 라는 값이 들어갔습니다.

Values 파일에는 좀 더 구조화된 데이터를 담을 수도 있습니다. 예를 들어, `favorite` 항목을 별도로 정의한 뒤, 하위 항목으로 여러 값들을 지정할 수도 있습니다.

```yaml
favorite:
  drink: coffee
  food: pizza
```

자, 그렇다면 템플릿의 내용을 살짝 바꿔보도록 하죠. .Values.favorite.drink 와 같은 방식으로 계층적인 구조의 데이터를 사용할 수 있습니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  drink: {{ .Values.favorite.drink }}
  food: {{ .Values.favorite.food }}
```

계층화된 데이터를 사용할 때는 가능하다면 values tree (값 구조 형태) 를 얕은 형태, 그러니까 너무 복잡한 구조를 만들지 않는 것이 좋습니다. 

## Deleting a default key

여러분이 기본 값들 중에서 특정 키에 해당하는 값을 삭제해야 한다면, 해당 키의 값을 `null` 로 설정하면 됩니다. 그렇게 하면 Helm은 해당 키의 값을 삭제합니다.

예를 들어, stable/Drupal 라는 차트는 liveness probe를 설정할 수 있도록 옵션을 제공하고 있습니다. 아래는 해당 차트의 기본 값들입니다.

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  initialDelaySeconds: 120
```

만약 여러분이 `--set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt]` 옵션을 통해 livenessProbe 핸들러를 `httpGet` 대신 `exec` 로 바꾸려고 시도할 경우, Helm은 기본 값과 덮어 씌워지는 값을 합칩니다. 즉, 아래와 같은 YAML이 생성됩니다. httpGet과 exec이 공존하고 있습니다.

```yaml
livenessProbe:
  httpGet:
    path: /user/login
    port: http
  exec:
    command:
    - cat
    - docroot/CHANGELOG.txt
  initialDelaySeconds: 120
```

그러나, 쿠버네티스는 1개보다 많은 livenessProbe 핸들러를 선언하는 것을 허용하지 않습니다. 이를 해결하기 위해서, Helm에게 `livenessProbe.httpGet` 의 값을 삭제하라고 임의로 설정할 수가 있습니다. 바로 null로 설정하는 것입니다.

```sh
helm install stable/drupal --set image=my-registry/drupal:0.1.0 --set livenessProbe.exec.command=[cat,docroot/CHANGELOG.txt] --set livenessProbe.httpGet=null
```

지금까지 Helm 템플릿에 내장된 몇 가지 오브젝트들에 대해서 알아보았습니다. 그리고 그러한 값들을 템플릿으로 가져오는 방법도 알아보았죠. 다음 장에서는 템플릿 엔진을 통해 함수와 파이프라인을 사용하는 방법에 대해서 알아보겠습니다.

이 문서의 원본 리포지터리 커밋 : [40bc1b1](https://github.com/helm/helm/commit/40bc1b173d2d20591c0d9bf9d690e06bc0de7c55) (on 4 Jun 2019)
