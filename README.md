# k8s_test_preparation
## Core Concepts
25. Pods
- pod 를 command 로 생성하는 방법? kubectl run podname --image=nginx

## Scheduling 에서 틀렸던 부분
51. scheduling 

- node select 할 때는 nodeName 이라는 인자를 넣어서 nodeselect 를 할 수 있다. 
- 그런데.. kubectl edit pod 해서 pod 안에서 nodeName 을 바꾸면 에러 나는데...?? 


54. Labels and Selectors
- pod 중 특정 label 을 고르는 법 : kubectl get pods -l env=prod,bu=finance 


57. Tains and Tolerations
- taint 는 모기약이고, tolerations 는 모기의 면역력이다. 다만, 특정 면역력이 있는 모기는 taint 가 없는 곳에도 갈 수 있으니, 조심해라
- tains 를 node 에 다는 법? kubectl taint nodes node-name key=value:taint-effect (NoSchedule (없으면 못들어오게하는거) | PreferNoSchedule (무조건은 아니지만...) | NoExecute(더이상 못들어오고, 이미 있는거중 tolerations 있는거만 남기기))
- kubectl taint nodes node1 key1=value1:NoSchedule- 이렇게 마지막에 - 를 넣어서 taint 지울 수도 있다.
- tolerations 를 pod 에 다는 법? kubectl tolerations nodes node1 app=blue:NoSchudule 인 경우 yaml 파일에 spec > tolerations > [key:"app" operator:"Equal" value:"blue" effect:"Noschudule"] 하기!
- ![Screenshot](image/tolerations.png)
- node 에 달린 taint 를 command 로 보는법? kubectl describe node node1 | grep Taint

60. Node Selectors & Affinity
- node 에 달린 label 정보를 pod 의 yaml 내에 넣어서 특정 node 에 할당 가능 (spec > nodeSelector > key:value)
- node 에 label 다는 법? kubectl label nodes node-name label-key=label-value
- 그럼 특정 label 이 아닌, 이거 아니면 이거 이렇게 해야 하는 경우? 혹은 이건 안되! 해야하는 경우? 아래처럼 **Affinity** 사용!
- ![Screenshot](image/affinity.png)
- 위의 그림처럼 하면, operator 가 in 인 경우는 values 중 하나라도 해당하면 그 node 에 들어갈 수 있다
- operator 가 Exists 인 경우는 key 만 보고, 그게 있으면 되는거고 values 는 따지지 않는다.
- node 의 label 보는 법 : kubectl get nodes --show-labels
- node 에 label 다는 법 : kubectl label nodes your-node-name disktype=ssd
- ![Screenshot](image/node_affinity.png)

65. Resource Requirements and Limits
- k8s 에서는 1 vCPU 와 512 Mi 의 memory 가 default 로 설정된다. 이게 싫으면 manually 수정해줘야 한다. 단, cpu 는 limit 에 정한대로 작동하지만, memory 는 그 이상도 쓸 수 있다. 계속 많이 쓰면 terminate 된다.
- ![Screenshot](image/resource_limit.png)

70. Daemonset
- 모든 ns 에 대해 확인해보려면 kubectl get pod --all-namespaces

73. Static Pods 
- kubelet 이 kube-apiserver 통해 어떤 pod 를 자기 node 에 넣으면 좋을지 알아냄. (kube-scheduler 통해 etcd 저장된 정보 통함). 만약 master node 가 없다면 kubelet 이 captin 역할을 할 수 있을까? 어떻게 할까?? yaml 만 가지고 어떻게 할까? pod definition 을 읽기 위해 /etc/kubernetes/manifests 에 yaml 넣으면 됨. (kubelet.service 의 configuration 에서 --pod-manifest-path 를 수정해서 yaml 다른 곳에 넣어도 동작하도록 수정 가능. 혹은 config=kubeconfig.yaml 이라 적고, kubeconfig.yaml 에 staticPodPath: /etc/kubernetes/manifests 를 수정해도 됨.) 정기적으로 여기 읽어서 pod 만듬. 에러 나면 kubelet 이 처리함. 이렇게 master 의 기능 없이 kubelet 이 만든걸 **static POD** 라 함. 단 여기에 replicaset 관련이나 deployment yaml 넣는다고 그것들이 만들어지진 않는다. kubelet 은 pod 단위만 관리. 
- static pod 가 만들어지면 docker ps 를 통해 static pod 확인 가능. (kube api-server 가 없으니까 docker 로 확인해야 함) (kubectl get pod 해도 나오긴 함)
- 어디에 쓰나? master node 에서 etcd 나 api-server, kube-scheduler 같은거 다 static pod 이다.
- 그럼 daemonset 과 뭐가 다른가? static pod : created by the kubelet, deploy control plane components as Static Pods 인데 daemonset : created by kube-API server, deploy monitoring agents, Logging Agents on nodes 가 다르다. 둘다 Ignored by the Kube-scheduler 는 같은 특징이다.
- 특정 node 안에 있는 kubelet 의 config 파일 문제
- ![Screenshot](image/kubelet_node01.png)

76. Multiple Schedulers
- kube-scheduler 바꾸어서 custom scheduler 만들 수 있다. 여러개 한번에 띄워져 있도록 하기 가능. 
- wget https://...../kube-scheduler 해서 다운 받고, kube-scheduler.service 의 **--scheduler-name** 을 변경해서 생성 가능 (/etc/kubernetes/manifests/kube-scheduler.yaml)
- 여러개 copy 되어 있으면, **--leader-elect** 가 true 인 애가 행동
- 두개가 true 면 **lock-object-name** 인자 넣어서 differentiate 하기
- pod 생성시에 특정 scheduler 쓰려면 yaml 에서 spec>schedulerName 에 넣으면 됨
- kubectl get events 하면 source 칸에서 어떤 scheduelr 썻는지 보임.
- kubectl logs my-custom-scheduler --name-space=kube-system 하면 log 다 보임
- 질문 
- ![Screenshot](image/q1_sched.png)

83. monitoring
- node level monitoring/ pod level monitoring 어떻게 하나? **metrics server**, prometheus, elastic stack, datadog, dynatrace ...
- metrics server : in-memory monitoring solution. k8s 는 api-server 에 따라 움직이는 kubelet 이라는 애로 each node 에서 운영된다. 그리고 kubelet 은 contatiner Advisor(**cAdvisor**) 을 가지고 있는데, 얘가 retrieveing performance of pod 해서 kubelet 한테 줘서, metrics server 가 확인 가능.
- metrics server 다운로드 및 사용법? git clone https://github.com/kubernetes-incubator/metrics-server.git 해서 다운받고, kubectl create -f deploy/1.8+ 해서 띄우기
- 이후 kubectl top node / kubectl top pod 해서 메트릭 볼 수 있다.

86. Managing App Logs
- kubectl logs -f event-pod
- 만약 여러개의 container 가 pod 에 있으면, kubectl logs -f event-pod container1 이런식으로 써야함

92. Rolling Updates and Rollbacks
- rollout 확인 법 : kubectl rollout status deployment/myapp-deployment
- recreate 는 replica 를 0 으로했다가 새로운거를 올리는거라 app down 생김
- rolling update 는 하나씩 바꾸는 거
- commnad 로 deployment image 를 바꾸려면 : kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 (여기 nginx= 할때 nginx 는 container 명이다)
- rollout 취소 하는 법 : kubectl undo deployment/myapp-deployment
- 위에꺼 하고 kubectl get rs 하면 바뀐거 보임

96. Commands
- docker 에서 ENTRYPOINT=sleep 쓰면 해당 명령어 항상 먼저 실행됨. 그래서 docker run ubuntu-sleeper 10 하면 sleep 은 항상 실행 되니까, 10 이 뒤에 붙어서 sleep 10 이 됨
- ENTRYPOINT ['sleep'] 이랑 아래줄에 CMD ['5'] 넣으면 sleep 5 됨.
- k8s yaml 에 쓸때는 spec>continaers> args 에 ['5'] 넣고, 도커 자체의 ENTRYPOINT 에 sleep 넣으면 sleep 5 됨
- 만약 ENTRYPOINT 를 바꾸려면 spec>containers>command : ["sleep2.0"] 이렇게 command 에 넣으면 됨

100. Configure Env Variable sin Applications
- configmap 만들고, pod 에 inject 하면 env 를 넘길 수 있다
- imperative : kubectl create configmap config-name --from-literal=key=value --from-literal .... or kubectl create configmap config-name --from-file=path-to-file
- declarative : kubectl create -f 이고, config-map.yaml 에는 data: > key:value 로 만들기
- pod 에서 부를때는 spec > containers > envFrom > - configMapRef > name 이렇게 함

104. secrets in Application
- imperative : kubectl create secret generic secret-name --from-literal=key=value 아니면 kubectl create secret generic secret-name --from-file=path-to-file
- 위처럼 from-literal 을 쓸때는 그냥 encode 안해서 해도 됨
- declarative : kubectl create -f
- configmap 과 다르게 encoded format 으로 data 를 넣어놔야 함
- encoding 하는 법은 linux 에서 echo -n 'mysql' | base64 이렇게 encoding
- kubectl get secret app-secret -o yaml 하면 yaml 로 볼 수 있으나, encoding 되어 있음
- decoding 은 echo -n "bXlzcWw=" | bask64 --decode 이렇게!

109. multi Container PODs
