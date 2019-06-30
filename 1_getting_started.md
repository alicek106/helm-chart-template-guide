# ��Ʈ ���ø� �ۼ� �����ϱ�

�̹����� ���ο� ��Ʈ�� �ϳ� ������ ��, ù ��° ���ø��� �߰��غ��ڽ��ϴ�. �̹��� �����ϴ� ��Ʈ�� �� ���̵� ���ݿ� ���� ��� ���� ���̹Ƿ�, �������� ���� �����صδ� ���� ��õ�մϴ� :D

## ��Ʈ �����ϱ�

������ [Charts Guide](https://github.com/helm/helm/blob/master/docs/charts.md)���� �����ߴ� ��ó��, Helm ��Ʈ�� �Ʒ��� ���� ������ �����Ǿ� �ֽ��ϴ�.

```
mychart/
  Chart.yaml
  values.yaml
  charts/
  templates/
  ...
```

'templates/' �� ���ø� ������ ����Ǿ� �ִ� ���͸��Դϴ�. Tiller�� ��Ʈ�� �޾Ƶ鿩 ó���� ��, Tiller�� 'templates/' ���͸� �ȿ� �ִ� ��� ������ Helm�� ���ø� ������ �������� �����մϴ�. �׸��� ���� Tiller�� �� ���ø��� ������ ������� �ٽ� ���� ����, ������Ƽ���� ������ ������ ���ҽ��� �����մϴ�.

'values.yaml' ���ϵ� ���ø��� ����ϱ� ���ؼ��� �ʼ����� �����Դϴ�. �� ������ ��Ʈ�� ���Ǵ� _�⺻ ����_ �� �����ϰ� �ֽ��ϴ�. �� ������ ����ڰ� 'helm install' �̳� 'helm upgrade' ���ɾ ����� �� --set �ɼ��� ���� ���� �������� ����� ���� �ֽ��ϴ�.

'Chart.yaml' ������ ��Ʈ�� ���� ���� (description) �� �����ϰ� �ֽ��ϴ�. �������� ���ø� ���ο����� �� ������ ���뿡 ������ ���� �ֽ��ϴ�. 

'charts/' ���͸��� �ٸ� ��Ʈ���� �����ϰ� _���� ����_ �ֽ��ϴ�. (���� ��� _subchart_, �� ���� ��Ʈ��� ���� �͵��� �̿� �ش�˴ϴ�. ��Ʈ�� �����ϸ� subchart�� �Բ� �����˴ϴ�) �� ���̵带 ��� �д� ����, �̷��� ��Ʈ�� ���ø����� ���ø� ������ �� ��� �����ϴ����� ������ �� ���� ���Դϴ�.

## A Starter Chart

'mychart' ��� �̸��� ������ ��Ʈ�� ���� �ϳ� ���� ��, 'mychart' ��Ʈ �ȿ� ���ø��� ������ ���ڽ��ϴ�.

```console
$ helm create mychart
Creating mychart
```

���ݺ��� �츮�� �� 'mychart' ���͸� �ȿ��� �۾��� ���Դϴ�.

### 'mychart/templates/' ���͸� �Ⱦ��

`mychart/templates/` ���͸� ���θ� Ȯ���� ����, �� ���� ������ �̹� �����մϴ�.

- `NOTES.txt`: �������� ��Ʈ�� ������ �� �ִ� 'help text' �Դϴ�. �� ���Ͽ��� 'helm install' ���ɾ ���� ������ ��Ʈ�� ������Ƽ���� �������� �� �͹̳ο� ��µǴ� ������ ��� �ֽ��ϴ�.
- `deployment.yaml`: ������Ƽ�� ���÷��̸�Ʈ�� �����ϱ� ���� �⺻ �Ŵ��佺Ʈ �����Դϴ�.
- `service.yaml`: ���÷��̸�Ʈ�� �����ϱ� ���� ���񽺸� �����ϴ� �⺻ �Ŵ��佺Ʈ �����Դϴ�.
- `_helpers.tpl`: ��Ʈ ���ݿ� ���� ������ �� �ִ� ���ø� ���� (Helper) �� �����صδ� �����Դϴ�.

�׸��� �������� �������� �ؾ��� ����.. _�� ���ϵ��� ���� ����� ���Դϴ�!_ �׷��� �ؾ߸� �ƿ� ó������ Ʃ�丮���� �����ϸ� ���ø� ��� ����� �н��� �� �ֱ� �����Դϴ�. ������ ���̵带 ������ �����鼭 �����и��� 'NOTES.txt' ���ϰ� '_helpers.tpl' ������ ������ ���Դϴ�.

```console
$ rm -rf mychart/templates/*.*
```

������, �������� ���� � (production) �ܰ� �� ��Ʈ�� �ۼ��ϰ� �ִٸ�, ��Ʈ���� �⺻ �������� �����ϴ� ���� �����ϴ�. ���� ���, Github�� ���� �� ��Ʈ�� �������� �ϴ� �͵� �������� �ʰ���.

## ù ��° ���ø� �ۼ��غ���

�츮�� ������ �� ù ��° ���ø��� 'ConfigMap' �Դϴ�. ������Ƽ������ ���Ǳ׸��� ���� �����͸� �����ϱ� �뵵�� ���˴ϴ�. ���Ǳ׸��� �ſ� �⺻���� ���ҽ��̱� ������, ���ø��� ó�� �����ϱ�δ� ������ �� �����ϴ� :D

`mychart/templates/configmap.yaml` ��� ������ ���Ӱ� ������ ������ �ۼ��� ���ڽ��ϴ�.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Hello World"
```

**TIP:** ���ø� ������ ���̾� (�Ǵ� Ȯ����) �� ������ �̸� ���� ���� (Naming Pattern) �� �������� �ʽ��ϴ�. �׷�����, YAML ���Ͽ� ���ؼ��� '.yaml' ��, ���� ���Ͽ� ���ؼ��� '.tpl' �� �����մϴ�.

���� YAML ������ ���Ǳ׸��� �����ϱ� ���� �ּ����� �ʵ常�� �����ϰ� �ִ�, �⺻ ���뿡 �Ұ��մϴ�. �� ������ `templates/` ���͸� ���ο� ��ġ�� �ֱ� ������, �� ������ ���ø� ������ ���� ������Ƽ���� ���۵� ���Դϴ�.

�׷����� ���ø� ������ �ƴ� �ܼ��� (plain) YAML ������ 'templates/' ���͸��� �ξ ũ�� ����� �����ϴ�. Tiller�� �� ���ø��� �о���� ��, ������ ���� �� ������ �ִ� �״�� ������Ƽ���� ������ ���Դϴ�.

����Ե� �� �ܼ��� ���ø������ε� ��Ʈ ��ġ�� �����մϴ�. �Ʒ��� ���� ��Ʈ�� ��ġ�� ���ڽ��ϴ�.

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

���� ��� �������, ���Ǳ׸��� ������ ���� Ȯ���� �� �ֽ��ϴ�. Helm�� �̿��� release�� ���¸� Ȯ���� �� ������, ������ ���ø��� �ε�Ǿ����� �� �� �ֽ��ϴ�.

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

'helm get manifest' ���ɾ�� release �̸� (���� ��쿡�� 'full-coral') �� ���ڷ� �޾Ƽ�, ������ ������ ���ε�� ������Ƽ�� ���ҽ��� ��� ������ ����մϴ�. �� ������ YAML ������ ������ �ǹ��ϴ� '---' �� �����ϰ�, �� ���� �ּ� ('# Source:.. ') �� �ش� YAML ������ ��� ���ø� ���Ϸκ��� ������ �������� �ǹ��մϴ�.

���� ������� �̷���, �� YAML �����ʹ� �츮�� 'configmap.yaml' ���Ͽ� �����ߴ� �����Ͱ� �´� �� �����ϴ�.

�̹����� `helm delete full-coral` ���ɾ ���� release�� ������ �� �Ѿ�ڽ��ϴ�. �ڿ��� �ٽ� ������ ���Դϴ�.

### ������ ���ø� ��û �߰��ϱ�

���ҽ� ���ο� 'name:' �� ���� �ϵ��ڵ� ������� ���� �ִ� ���� ��� �״��� ���� ������ �ƴմϴ�. �̹����� ���Ǳ׸��� 'names:' �ʵ带 Helm�� release �̸����� ������ ���ڽ��ϴ�. Helm�� release �̸� ���� �����ϱ� ������, ������Ƽ������ ���ҽ� �̸��� ��ĥ ���� ���� ���Դϴ�.

**TIP:** 'names:' �ʵ�� 63���� ���ڷ� ���ѵǾ� �ִµ�, �̴� DNS �ý����� �Ѱ� �����Դϴ�. �� ���� ������ release �̸��� ��ǻ� 53���� ���ڷ� ���ѵ˴ϴ�.

'configmap.yaml' ������ �����ϰ� �ٲ㺸�ڽ��ϴ�.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```

���� ū ��ȭ�� 'names:' �׸��� ���Դϴ�. ���� ���ÿ����� '{{ .Release.Name }}-configmap' ���� �ٲ������.

> ���ø��� ������, �� .Release.Name �� ���� ���� '{{' �� '}}' ���� �ȿ��� ���Ǿ�� �մϴ�.

���ø� �������� `{{ .Release.Name }}` �� release�� �̸��� ���ø� ���η� �����ɴϴ�. ���ø����� ���޵Ǵ� �̷��� ������ �����ڸ� _'���ӽ����̽��� ���ϴ� ������Ʈ'_ ����� ǥ���� ���� �ְڽ��ϴ�. ��, ���� ('.') �� �� ���ӽ����̽��� ��Ҹ� �������� ������. `{{ .Release.Name }}` �� Release ���ӽ����̽��� Name�̶�� ���̶�� �����غ��� �������� �𸣰ڽ��ϴ�. 

'Release' �տ� �پ� �ִ� ������ ��Ʈ�� ���� �� ���� �ֻ��� ���ӽ����̽����� ���������� �ǹ��մϴ� (�̿� ���ؼ��� ���߿� �� �� �ڼ��� �ٷﺸ���� ����). ��·��, '.Release.Name' �̶�� ���� "���� �ֻ��� ���ӽ����̽����� ������, 'Release' ��� ������Ʈ�� ã�Ƽ� 'Name' �̶�� ���� ����Ѵ�" ��� �ؼ��� �� �ְڽ��ϴ�. 

'Release' ������Ʈ�� ��� Helm�� ��ü������ ����Ǿ� �ִ� ������Ʈ �� �ϳ��Դϴ�. �̿� ���ؼ��� ���߿� �� �� ���� �ٷﺸ���� ����. �׷��� ������ `{{ .Release.Name }}` �� 'Tiller�� release�� �Ҵ��ϴ� ������ �̸�' �� �ǹ��Ѵٴ� �͸� �����ϰ� �Ѿ�� �˴ϴ�.

��, ���� ���ҽ��� ��ġ�� �� ���ø� �������� ������� �ٷ� Ȯ���غ����� �ϰڽ��ϴ�.

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

'RESOURCES' �κп��� ���Ǳ׸��� �̸��� `clunky-serval-configmap` �� �ٲ� ���� ��ġä�̳���?

������ ���ҽ��� YAML ���� ��ü�� Ȯ���ϰ� �ʹٸ� 'helm get manifest clunky-serval' ���ɾ ����� �� �ֽ��ϴ�.

�̹� ���̵忡���� ���ø��� ����ϴ� ���� �⺻���� ����� '{{' �� '}}' �� �ѷ����� ���ø� �����ڿ� ���ؼ� �˾ƺ��ҽ��ϴ�. ���� ��Ʈ������ ���ø��� ��� ����� ���� �� �� ���� �˾ƺ� ���Դϴ�. �׷��� ���� ��Ʈ�� �Ѿ�� ��, ���ø��� �� �� ������ ������ �� �ִ� ���� �ֽ��ϴ�. ���ø� �������� �׽�Ʈ�ؾ� �Ѵٸ� (�׷����� release ������ �ϰ� ���� �ʴٸ�) `helm install --debug --dry-run ./mychart` �� ���� Dry Run�� �� ���� �ֽ��ϴ�. �̴� Tiller ������ ��Ʈ�� ���� �������� ������, ��Ʈ�� ��ġ������ ������ �ܼ��� �������� ���ø����� ��ȯ�մϴ�.

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

'--dry-run' �� �����и��� Helm ��Ʈ ���ø��� �ۼ��� �� ������� �����ִ� �ſ� ������ ����Դϴ�. '--dry-run' �� ���� ���� �� �����ߴٰ� �ؼ� ��Ʈ�� �������� �� ������ ���̶�� ������ ���� �ʴ°� �����ϴ�. �������� ���ø� �������� ������ ���ٴ� ���̸�, ������ ���ø��� ������Ƽ���� ���� ���� �� ���������� �𸣴� ���̴ϱ��.

���� ���̵忡���� ������ ����غ� �⺻ ��Ʈ�� ��� ����� �� ���Դϴ�. Helm�� ��ü������ ����� ������Ʈ���� ����� �����ν�, ���ø� �� �� �� �ڼ��� �˾ƺ����� ����.



�� ������ ���� �������͸� Ŀ�� : c99a3c6 (on 13 Feb 2019)

