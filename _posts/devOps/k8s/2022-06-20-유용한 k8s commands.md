---
title: 유용한 k8s command (계속 업데이트)
author: OsoriAndOmori
date: 2022-06-20 18:00:00 +0900
categories: [Blogging, DevOps]
tags: [k8s, command]
---

## 원격으로 해당 팟에서 실행하기 : -- 가 명령어를 갈라주는 핵심.
kubectl exex [pod-name] -- curl -s ip:port/logic

## DNS 찾기 : 내부에서 호출해야하는 cluster ip 전용.
kube exec app-68bb5f68f7-xc967 -- cat /etc/resolv.conf
