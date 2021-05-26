# auto.k8s
vagrant, ansible, kubespray

### 사용방법 ###

1. 신규노드 추가 방법
   A. Vagrantfile의 common 스크립스 수정
      - common 스크립트에서 /etc/hosts 파일에 추가하고자 하는 노드의 hostname과 ip를 추가할 것
      - config.vm.define을 사용하여 신규 노드 정의하고, nodes_setup 스크립트가 실행되도록 추가할 것
   B. vagrant reload [신규노드명] ansible --provision 실행
