# Helm Chart 개발자를 위한 가이드

이 가이드는 Helm 차트 템플릿을 작성하기 위한 템플릿 언어 사용 방법을 소개합니다.

Helm 차트의 템플릿은 쿠버네티스가 해석할 수 있는 YAML 형식의 리소스 매니페스트 파일을 생성합니다. 이 가이드에서는 템플릿이 어떤 구조로 구성되어 있는지, 어떻게 사용되는지, Go 템플릿으로서 어떻게 작성되는지, 그리고 실제 여러분의 환경에서 어떻게 디버깅할 수 있을지를 다룹니다.

이 가이드는 아래 내용을 중점적으로 설명합니다.

- Helm 템플릿 언어
- values 사용 방법
- 템플릿을 사용하기 위한 기술들

이 가이드 [(chart_template_guide)](https://github.com/helm/helm/tree/master/docs/chart_template_guide) 는 Helm 템플릿 언어의 Input, Output에 대해 배우는 것을 목적으로 합니다. [다른 가이드들](https://github.com/helm/helm/tree/master/docs)은 Helm의 개념 소개, 사용 예시 및 Best Practice로 구성되어 있습니다. 필요에 따라 적절히 문서를 정독하는 것을 추천합니다 :D



# 이 저장소에 대하여

이 저장소는 Helm Github 리포지터리에서 제공하고 있는 Docs 중 chart_template_guide를 번역한 것입니다. 쉬운 이해를 위해 적절한 설명 및 의역을 추가했습니다. 본 저장소의 저작권은 제가 아닌 Helm에 있습니다.



이 문서의 원본 리포지터리 커밋 : d5346b1 (on 5 Nov 2016)
