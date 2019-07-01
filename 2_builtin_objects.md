# Helm에 자체적으로 내장되어 있는 오브젝트 (값) 들

템플릿이 템플릿 엔진으로 전달되어 렌더링될 때, 오브젝트라고 하는 값도 함께 템플릿 엔진으로 전달됩니다. 그리고 여러분의 코드 또한 오브젝트로서 전달될 수도 있습니다 (이를 위해 'with' 나 'range' 구문을 사용하는 예시도 뒤쪽에서 살펴볼 것입니다). 여러분의 템플릿에서 새로운 오브젝트를 생성하는 방법으로는 'list' 함수와 같이 몇 가지 방법이 있습니다만, 이에 대해서도 뒤에서 다시 살펴볼 것입니다.

오브젝트는 단순히 하나의 값을 가질수도 있습니다. 또는 오브젝트 자체가 다른 오브젝트나 함수를 포함하고 있을 수도 있죠. 예를 들어, 이전 가이드에서 사용해보았던 `Release` 오브젝트는 `Release.Name`과 같은 여러 개의 오브젝트를 포함하고 있으며, `Files` 오브젝트는 몇 가지 함수를 포함하고 있기도 합니다.

이전 가이드에서, 템플릿에 release 이름을 집어넣기 위해 `{{.Release.Name}}` 를 사용했던 것을 기억할 것입니다. `Release`는 템플릿의 가장 최상위 레벨에서 접근할 수 있는 대표적인 오브젝트 중 하나입니다.

- **'Release'** : release에 대한 정보들을 포함하고 있습니다. `Release` 오브젝트는 다음과 같은 오브젝트를 포함하고 있습니다.
  - **'Release.Name'** : release의 이름입니다.
  - **'Release.Time**' : release가 생성된 시간입니다.
  - **'Release.Namespace'** : release가 생성될 네임스페이스입니다. (매니페스트 파일에서 덮어 씌워질 수 있습니다)
  - **'Release.Service'** : 서비스를 생성하는 주체의 이름입니다. (항상 `Tiller` 로 설정됩니다. Helm 3부터는 어떻게 될 지 모르겠네요)
  - **'Release.Revision' **: 이 Release의 리비전 번호입니다. 이 값은 1부터 시작해서 `helm upgrade` 명령어를 사용할 때마다 1씩 올라갑니다.
  - **'Release.IsUpgrade'** : 현재 실행하는 명령이 helm upgrade나 rollback 이라면 `true` 로 설정됩니다.
  - **'Release.IsInstall'** : 현재 실행하는 명령이 helm install 이라면 `true` 로 설정됩니다.

- **'Values'** : Values는 `values.yaml` 파일을 통해 템플릿으로 전달되는 값입니다. values.yaml 파일은 사용자가 직접 수정할 수 있습니다. 기본적으로는 `Values` 는 비어있는 값으로 초기화됩니다.
- **'Chart'** : `Chart.yaml` 파일의 내용을 가리킵니다. `Chart.yaml` 파일에 저장되어 있는 데이터는 `Chart` 를 통해 접근할 수 있습니다. 예를 들어, `{{.Chart.Name}}-{{.Chart.Version}}` 는 'mychart-0.1.0' 를 출력할 것입니다.
  - `Chart`에서 사용 가능한 값은 [Charts Guide](https://github.com/helm/helm/blob/master/docs/charts.md#the-chartyaml-file) 에서 확인할 수 있습니다.
- **'Files'** : `Files`는 차트 내부에 위치한 일반적인 파일에 접근할 수 있는 오브젝트입니다. 예를 들어 템플릿에 접근하기 위해 `Files`를 쓸 수는 없으나, 차트 안의 config.ini과 같은 파일에 접근할 수 있는 기능을 제공합니다. 더 많은 설명을 읽고 싶다면 _Accessing Files_ 가이드를 참고하기 바랍니다.
  - **'Files.Get'** : 파일 내용을 이름으로서 가져올 수 있는 함수입니다. (ex. `.Files.Get config.ini`)
  - **'Files.GetBytes'** : 파일의 내용을 문자열 대신 바이트 배열으로 가져옵니다. 일반적으로 생각하면 별로 쓸모는 없지만, 이미지 파일같은 것을 가져올 때는 꽤 유용합니다.
- **'Capabilities'** : 이 오브젝트는 쿠버네티스 클러스터가 지원할 수 있는 기능들에 대한 정보를 제공합니다.
  - **'Capabilities.APIVersions'** : API 버전들의 집합입니다.
  - **'Capabilities.APIVersions.Has $version'** : 특정 버전 (ex. `batch/v1`) 이나 리소스 (`apps/v1/Deployment`) 가 클러스터 내부에서 사용 가능한지를 나타냅니다.
  - **'Capabilities.KubeVersion'** : 쿠버네티스 버전을 나타냅니다. 이는 'Major', 'Minor', 'GitVersion', 'GitCommit', 'GitTreeState', 'BuildDate', 'GoVersion', 'Compiler', 'Platform'과 같은 값들을 포함하고 있습니다.
- **'Template'** : 현재 실행되고 있는 템플릿에 대한 정보를 제공합니다.
  - **'Name'** : 현재 탬플릿의 경로 명입니다. 단, 차트 디렉터리를 기준으로 경로가 반환됩니다. (ex. `mychart/templates/mytemplate.yaml`)
  - **'BasePath'** : 현재 차트의 템플릿 디렉터리의 위치입니다. (ex. `mychart/templates`)

위 값들은 최상위 레벨의 오브젝트로서 사용 가능합니다. 뒤에서 다시 살펴보겠지만, 이러한 오브젝트들이 _어디에서든지 사용 가능하다_ 라는 것을 의미하지는 않습니다.

Helm에 내장된 값들은 Go 언어의 네이밍 컨벤션을 준수하기 위해 항상 대문자로서 시작합니다. 여러분만의 값을 만들 때에는 여러분의 팀에 적합한 네이밍 컨벤션을 사용하면 됩니다. [Helm Charts](https://github.com/helm/charts) 팀은 Helm에 내장된 값들과 로컬 값을 구분하기 위해 항상 소문자로 시작하는 이름을 사용합니다. 



이 문서의 원본 리포지터리 커밋 : [3bd6e9f](https://github.com/helm/helm/commit/3bd6e9fcf0de8827fa949c9440dd341bf548fc23) (on 20 Jun 2019)
