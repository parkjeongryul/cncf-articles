---
layout: post
title: "kubernetes scheduler simulator"
subtitle: "custom scheduler 개발 환경 구축하는 방법을 소개합니다"
date: 2022-10-24 23:45:13 -0400
background: '/img/posts/06.jpg'
---

# kubernetes scheduler simulator란?

쿠버네티스의 스케줄러는 다양한 설정을 지원하기 때문에, 대부분의 경우 Custom 스케줄러를 직접 구현할 필요가 없습니다.
> 쿠버네티스의 스케줄러 플러그인 예시
<img width="720" alt="image" src="https://user-images.githubusercontent.com/25222969/198247597-7f9bf896-e39b-4ab5-8424-af3d38a73e5b.png">

<br>

그러나 스케줄러가 다른 컴포넌트들과 어떻게 상호작용 하는지 알기 위해서 또는 쿠버네티스 기본 스케줄러의 예상하지 못한 동작을 피하기 위해서 직접 스케줄러를 개발하고자하는 경우가 있습니다.

쿠버네티스 환경에서 스케줄링의 결과를 자세하기 확인하기 위해서는 control plane에 접속하여 log를 살펴볼 수 밖에 없는데요.

좀 더 쉽게 스케줄링의 결과를 확인하기 위해서 `kubernetes scheduler simulator`를 사용할 수 있습니다.

`kube-scheduler-simulator` 는 web UI형태로 실제 스케줄러의 동작을 시뮬레이션 할 수 있습니다.

<br>

# Setup kubernetes scheduler simulator

 `kube-scheduler-simulator`를 이용한 스케줄러 개발 환경 구축 방법에 대해서 소개합니다.
> 로컬 환경 실행 기준
<img width="720" alt="image" src="https://user-images.githubusercontent.com/25222969/198226456-f287e590-648c-4d28-8474-790cf89fcf60.png">

위 명령어로 실행했다면 http://localhost:3000 에 접속하여 nodes와 pods를 추가합니다.

기본 동작 방식은 매우 간단하고 직관적입니다. pods들은 모든 nodes에 고르게 스케줄링됩니다.

<img width="720" alt="image" src="https://user-images.githubusercontent.com/25222969/198226528-8bdcbec5-c9ba-40ce-99a8-b5665e49597b.png">

<br>

# Add a minimal scheduler to kube-scheduler-simulator
[mini-kube-scheduler](https://github.com/sanposhiho/mini-kube-scheduler)을 base implementation으로 custom scheduler 개발을 시작할 수 있습니다.

커스텀 스케줄러 로직을 구현해보겠습니다.

`mini-kube-scheduler`라는 기본적인 스케줄러 코드에서 시작하겠습니다.

1. Copy mini-kube-scheduler/minisched (branch : initial-random-scheduler) into kube-scheduler-simulator

2. kube-scheduler-simulator/scheduler/scheduler.go에서 minisched를 사용하도록 변경합니다. (변경 사항은 아래 코드 참고)

3. Examine the behavior change (완전 랜덤한 스케줄링 정책으로 코드를 변경하였습니다)

```golang
Patch license: Apache-2.0 (same as kube-scheduler-simulator)

diff --git a/scheduler/scheduler.go b/scheduler/scheduler.go
index a5d5ca2..8eb931d 100644
--- a/scheduler/scheduler.go
+++ b/scheduler/scheduler.go
@@ -3,6 +3,8 @@ package scheduler
 import (
     "context"

+    "github.com/kubernetes-sigs/kube-scheduler-simulator/minisched"
+
     "golang.org/x/xerrors"
     v1 "k8s.io/api/core/v1"
     clientset "k8s.io/client-go/kubernetes"
@@ -14,7 +16,6 @@ import (
     "k8s.io/kubernetes/pkg/scheduler/apis/config"
     "k8s.io/kubernetes/pkg/scheduler/apis/config/scheme"
     "k8s.io/kubernetes/pkg/scheduler/apis/config/v1beta2"
-    "k8s.io/kubernetes/pkg/scheduler/profile"

     simulatorschedconfig "github.com/kubernetes-sigs/kube-scheduler-simulator/scheduler/config"
     "github.com/kubernetes-sigs/kube-scheduler-simulator/scheduler/plugin"
@@ -59,7 +60,6 @@ func (s *Service) ResetScheduler() error {
 // StartScheduler starts scheduler.
 func (s *Service) StartScheduler(versionedcfg *v1beta2config.KubeSchedulerConfiguration) error {
     clientSet := s.clientset
-    restConfig := s.restclientCfg
     ctx, cancel := context.WithCancel(context.Background())

     informerFactory := scheduler.NewInformerFactory(clientSet, 0)
@@ -71,36 +71,10 @@ func (s *Service) StartScheduler(versionedcfg *v1beta2config.KubeSchedulerConfig

     s.currentSchedulerCfg = versionedcfg.DeepCopy()

-    cfg, err := convertConfigurationForSimulator(versionedcfg)
-    if err != nil {
-        cancel()
-        return xerrors.Errorf("convert scheduler config to apply: %w", err)
-    }
-
-    registry, err := plugin.NewRegistry(informerFactory, clientSet)
-    if err != nil {
-        cancel()
-        return xerrors.Errorf("plugin registry: %w", err)
-    }
-
-    sched, err := scheduler.New(
+    sched := minisched.New(
         clientSet,
         informerFactory,
-        profile.NewRecorderFactory(evtBroadcaster),
-        ctx.Done(),
-        scheduler.WithKubeConfig(restConfig),
-        scheduler.WithProfiles(cfg.Profiles...),
-        scheduler.WithPercentageOfNodesToScore(cfg.PercentageOfNodesToScore),
-        scheduler.WithPodMaxBackoffSeconds(cfg.PodMaxBackoffSeconds),
-        scheduler.WithPodInitialBackoffSeconds(cfg.PodInitialBackoffSeconds),
-        scheduler.WithExtenders(cfg.Extenders...),
-        scheduler.WithParallelism(cfg.Parallelism),
-        scheduler.WithFrameworkOutOfTreeRegistry(registry),
     )
-    if err != nil {
-        cancel()
-        return xerrors.Errorf("create scheduler: %w", err)
-    }

     informerFactory.Start(ctx.Done())
     informerFactory.WaitForCacheSync(ctx.Done())
```


다음과 같이 랜덤하게 pod가 스케줄링 되었습니다.

<img width="720" alt="image" src="https://user-images.githubusercontent.com/25222969/198226752-ee124c49-96c6-4942-a418-b2e8e872a3c2.png">

<br>

# Modify the algorithm

minisched/minisched.go를 수정하여 스케줄링 알고리즘을 구현합니다..

다음과 같이 pod를 첫번째 노드에 무조건 스케줄링되도록 구현해보았습니다.

```golang
Patch license: MIT (same as mini-kube-scheduler)

diff --git a/minisched/minisched.go b/minisched/minisched.go
index 82c0043..ae02597 100644
--- a/minisched/minisched.go
+++ b/minisched/minisched.go
@@ -34,7 +34,7 @@ func (sched *Scheduler) scheduleOne(ctx context.Context) {
        klog.Info("minischeduler: Got Nodes successfully")

        // select node randomly
-       selectedNode := nodes.Items[rand.Intn(len(nodes.Items))]
+       selectedNode := nodes.Items[0]

        if err := sched.Bind(ctx, pod, selectedNode.Name); err != nil {
                klog.Error(err)
```

<img width="720" alt="image" src="https://user-images.githubusercontent.com/25222969/198226912-dbf3ff7c-8061-4f67-8261-a787a49934cc.png">

<br>

# Conclusion
- `kube-scheduler-simulator`를 이용해 real cluster없이, 커스텀 스케줄러를 개발할 수 있습니다. 

- `mini-kube-scheduler/minisched`를 커스텀 스케줄러의 베이스 코드로 사용할 수 있습니다.

# Refereneces
- [https://www.cncf.io/blog/2022/08/24/writing-your-own-scheduler-with-kube-scheduler-simulator](https://www.cncf.io/blog/2022/08/24/writing-your-own-scheduler-with-kube-scheduler-simulator)
- [https://github.com/kubernetes-sigs/kube-scheduler-simulator](https://github.com/kubernetes-sigs/kube-scheduler-simulator)
- [https://github.com/sanposhiho/mini-kube-scheduler/tree/initial-random-scheduler/minisched](https://github.com/sanposhiho/mini-kube-scheduler/tree/initial-random-scheduler/minisched)
