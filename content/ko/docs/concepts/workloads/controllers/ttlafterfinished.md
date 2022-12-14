---
# reviewers:
# - janetkuo
title: 완료된 잡 자동 정리
content_type: concept
weight: 70
---

<!-- overview -->

{{< feature-state for_k8s_version="v1.23" state="stable" >}}

완료-이후-TTL(TTL-after-finished) {{<glossary_tooltip text="컨트롤러" term_id="controller">}}는 
실행이 완료된 리소스 오브젝트의 수명을 제한하는 
TTL (time to live) 메커니즘을 제공한다. 
TTL 컨트롤러는 {{< glossary_tooltip text="잡" term_id="job" >}}만을 제어한다.

<!-- body -->

## 완료-이후-TTL 컨트롤러

완료-이후-TTL 컨트롤러는 잡만을 지원한다. 클러스터 운영자는
[예시](/ko/docs/concepts/workloads/controllers/job/#완료된-잡을-자동으로-정리)
와 같이 `.spec.ttlSecondsAfterFinished` 필드를 명시하여
완료된 잡(`완료` 또는 `실패`)을 자동으로 정리하기 위해 이 기능을 사용할 수 있다.
잡의 작업이 완료된 TTL 초(sec) 후 (다른 말로는, TTL이 만료되었을 때),
완료-이후-TTL 컨트롤러는 해당 잡이 정리될 수 있다고 가정한다.
완료-이후-TTL 컨트롤러가 잡을 정리할때 잡을 연속적으로 삭제한다. 이는
의존하는 오브젝트도 해당 잡과 함께 삭제되는 것을 의미한다. 잡이 삭제되면 완료자(finalizers)와
같은 라이프 사이클 보증이 적용 된다.

TTL 초(sec)는 언제든지 설정이 가능하다. 여기에 잡 필드 중
`.spec.ttlSecondsAfterFinished` 를 설정하는 몇 가지 예시가 있다.

* 작업이 완료된 다음, 일정 시간 후에 자동으로 잡이 정리될 수 있도록
  잡 메니페스트에 이 필드를 지정한다.
* 이미 완료된 기존 잡에 이 새 기능을 적용하기 위해서 이 필드를
  설정한다.
* [어드미션 웹후크 변형](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
  을 사용해서
  잡 생성시 이 필드를 동적으로 설정 한다. 클러스터 관리자는 이것을
  사용해서 완료된 잡에 대해 TTL 정책을 적용할 수 있다.
* 잡이 완료된 이후에
  [어드미션 웹후크 변형](/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhooks)
  을 사용해서 이 필드를 동적으로 설정하고, 잡의 상태,
  레이블 등에 따라 다른 TTL 값을 선택한다.

## 경고

### TTL 초(sec) 업데이트

TTL 기간은, 예를 들어 잡의 `.spec.ttlSecondsAfterFinished` 필드는
잡을 생성하거나 완료한 후에 수정할 수 있다. 그러나, 잡을
삭제할 수 있게 되면(TTL이 만료된 경우) 시스템은 TTL을 연장하기
위한 업데이트가 성공적인 API 응답을 리턴하더라도
작업이 유지되도록 보장하지 않는다.

### 시간 차이(Skew)

완료-이후-TTL 컨트롤러는 쿠버네티스 잡에
저장된 타임스탬프를 사용해서 TTL의 만료 여부를 결정하기 때문에, 이 기능은 클러스터 간의
시간 차이에 민감하며, 시간 차이에 의해서 완료-이후-TTL 컨트롤러가 잘못된 시간에 잡
오브젝트를 정리하게 될 수 있다.

시계가 항상 정확한 것은 아니지만, 그 차이는
아주 작아야 한다. 0이 아닌 TTL을 설정할때는 이 위험에 대해 유의해야 한다.



## {{% heading "whatsnext" %}}

* [자동으로 잡 정리](/ko/docs/concepts/workloads/controllers/job/#완료된-잡을-자동으로-정리)

* [디자인 문서](https://github.com/kubernetes/enhancements/blob/master/keps/sig-apps/592-ttl-after-finish/README.md)
