## 원격으로 해당 팟에서 실행하기 : -- 가 명령어를 갈라주는 핵심.
kubectl exex [pod-name] -- curl -s ip:port/logic

## DNS 찾기 : 내부에서 호출해야하는 cluster ip 전용. 
kube exec calendar-app-68bb5f68f7-xc967 -- cat /etc/resolv.conf
