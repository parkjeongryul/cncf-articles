---
layout: post
title: "kubernetes scheduler simulator을 이용한 custom scheduler 개발"
subtitle: "kubernetes scheduler simulator가 무엇인지, 설치 및 환경 구축 방법에 대해서 소개합니다"
date: 2022-10-24 23:45:13 -0400
background: '/img/posts/06.gif'
---

## kubernetes scheduler simulator란?

kubernetes의 기본적인 scheduler는 다양한 설정을 지원하기 때문에, 대부분의 경우 custom scheduler를 구현할 필요가 없습니다.

그러나 scheduler가 다른 컴포넌트들과 어떻게 동작하는지 알기 위해서 혹은 쿠버네티스 기본 스케줄러의 예상하지 못한 동작을 막기 위해서 직접  scheduler를 개발하고자하는 니즈가 있습니다.

kubernetes 환경에서 스케줄링의 결과를 자세하기 확인하기 위해서는 control plane에 접속하여 log를 살펴볼 수 밖에 없습니다. 이것이 `kube-scheduler-simulator` 를 개발한 이유입니다. `kube-scheduler-simulator` 는 web UI를 통해서 scheduler의 동작을 확인할 수 있습니다.

## kubernetes scheduler simulator 설치

`kube-scheduler-simulator` 는

이 아티클에서는 `kube-scheduler-simulator`를 이용한 scheduler 개발 환경 구축 방법에 대해서 소개합니다.

![움짤 되나?](https://user-images.githubusercontent.com/25222969/198219898-f0b33c0a-e2cf-48d1-8c20-0e00cc13d264.gif)


## kubernetes scheduler simulator란?
kubernetes의 기본적인 scheduler는 다양한 설정을 지원하기 때문에, 대부분의 경우 custom scheduler를 구현할 필요가 없습니다.
그러나 scheduler가 다른 컴포넌트들과 어떻게 동작하는지 알기 위해서 혹은 쿠버네티스 기본 스케줄러의 예상하지 못한 동작을 막기 위해서 직접  scheduler를 개발하고자하는 니즈가 있습니다.
kubernetes 환경에서 스케줄링의 결과를 자세하기 확인하기 위해서는 control plane에 접속하여 log를 살펴볼 수 밖에 없습니다. 이것이 `kube-scheduler-simulator` 를 개발한 이유입니다. `kube-scheduler-simulator` 는 web UI를 통해서 scheduler의 동작을 확인할 수 있습니다.

## kubernetes scheduler simulator 설치

`kube-scheduler-simulator` 는
이 아티클에서는 `kube-scheduler-simulator`를 이용한 scheduler 개발 환경 구축 방법에 대해서 소개합니다.

![움짤 되나?](https://user-images.githubusercontent.com/25222969/198219898-f0b33c0a-e2cf-48d1-8c20-0e00cc13d264.gif)
