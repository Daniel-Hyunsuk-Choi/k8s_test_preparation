# k8s_test_preparation
## Core Concepts
25. Pods
- pod 를 command 로 생성하는 방법? kubectl run podname --image=nginx
- 생성시 yaml 로 만드는 법 : k run http-go --image=http-go --dry-run -o yaml > http-go.yaml
- 이미 존재하는 pod 를 yaml 로 뽑으려면 k get pod {podname} -o yaml

## Scheduling 에서 틀렸던 부분
51. scheduling 

- node select 할 때는 nodeName 이라는 인자를 넣어서 nodeselect 를 할 수 있다. 
- 그런데.. kubectl edit pod 해서 pod 안에서 nodeName 을 바꾸면 에러 나는데...?? 


54. Labels and Selectors
- pod 중 특정 label 을 고르는 법 : kubectl get pods -l env=prod,bu=finance 
- env 가 있는 애만 검색해라 : k get pod --show-labels -l 'env' 
- env 가 없는 애만 검색해라 : k get pod --show-labels -l '!env'
- env 가 test 아닌애만 검색해라 : k get pod --show-labels -l 'env!=test'
- 보기 원하는 label 이 app 이랑 rel 일때 : k get pod -L app,rel
- 새로운 라벨 추가 : k label pod http-go-v2 test=foo
- 기존 라벨에 추가 : k label pod http-go-ve test=doo --overwrite
- 라벨 삭제 : k label pod http-go-v2 test-


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

- 디플로이먼트 스케일링 : k edit deploy {deploy name} --replicas={number}
- deployment history 관리 : k create -f http-go-deploy.yaml --record=true
- 

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
- ![Screenshot](image/configmap.png)
104. secrets in Application
- imperative : kubectl create secret generic secret-name --from-literal=key=value 아니면 kubectl create secret generic secret-name --from-file=path-to-file
- 위처럼 from-literal 을 쓸때는 그냥 encode 안해서 해도 됨
- declarative : kubectl create -f
- configmap 과 다르게 encoded format 으로 data 를 넣어놔야 함
- encoding 하는 법은 linux 에서 echo -n 'mysql' | base64 이렇게 encoding
- kubectl get secret app-secret -o yaml 하면 yaml 로 볼 수 있으나, encoding 되어 있음
- decoding 은 echo -n "bXlzcWw=" | bask64 --decode 이렇게!

109. multi Container PODs

113. Init Containers

120. OS Upgrades
- pod-eviction-timeout 을 통해 pod 가 죽은지 몇분 지났을때 실제로 죽었다고 인식할지 정할 수 있음 : kube-controller-manager --pod-eviction-timeout=5m0s ...
- kubectl drain node-01 을 통해 node-01 에 있는 pod 들 다른데로 옮길 수 있음
- 이후 kubectl uncordon node-01 을 통해 pod 막는거 치우기 (이후 새로운 pod 할당 가능)
- drain 은 옮겨버리는거지만 kubectl cordon node-02 하면 node-02 에 더이상 못들어 오게도 가능
- 이 명령어들을 Node upgrade 할때 쓸 수 있다

123. K8s SW Versions
- 1.10.3 : major.feature,function.bugfix

125. Cluster Upgrade Process
- kube-apiserver 의 버전이 가장 높고, > controller-manager 와 kube-scheduler 는 그보다 같거나 한개 낮고, > kubelet 과 kube-proxt 는 그보다 같거나 두개까지 낮다. kube-apiserver 보다 다른애들이 높으면 안된다!
- kubeadm 을 통해 업그레이드 하려고 하면, kubeadm upgrade plan 치면 현재 버전, latest stable version 등등의 정보 나옴
- apt-get upgrade -y kubeadm=1.12.0-00 -> kubeadm upgrade apply v1.12.0 -> kubectl get nodes (version 그대로인게 보임) -> apt-get upgrade -y kubelet=1.12.0-00 -> systemctl restart kubelet -> kubectl get nodes 하면 이제서야 마스터만 업그레이드 된게 보임
- 이제 각각의 node 들어가서 drain 해서 pod 옮겨주고, apt-get upgrade -y kubeadm=1.12.0-00 -> apt-get upgrade -y kubelet=1.12.0-00 -> kubeadm upgrade node config --kubeloet-version v1.12.0 -> system restart kubelet 하면 됨. upgrade 끝나면 kubectl undrain 해주기!

-- 여기서부터 134까지는 다시하자... ㅋㅋ

135. Security
- api-server 가 중추적인 역할을 하는데, 누가 여기에 접근 가능하고, 각각 접근 가능한 사람들에게 어떤 역할(RBAC)을 줄 지 잘 정하는게 중요!!
- k8s 에서 user 는 kubectl create user user1 이런거 불가능하지만, **serviceaccount** 는 관리 가능! kubectl craete serviceaccount sa1
- 모든 user access 는 kube-apiserver 에 의해 관리된다! admin 이 kubectl 을 쓰는거나, 개발자가 curl https://kube-server-ip:6443 이렇게 해서 pod 접근하는거나 모두 api--server을 통한다.
- Authentication : who can access?
- Authorization : what can they do? (RBAC (Role Base Access Control))
- kube-apiserver 가 (1) authenticate User 하고 (2) process request 한다
- authenticate 하는법은 : (1) static pwd file, (2) Static Token File, (3) Certificates, (4) Identity Service 이렇게 4가지 있다.
- (1) static pwd file: kube-apiserver.service 에다가 --basic-auth-file=user-details.csv 이렇게 넣고, user-details.csv 에다가 pwd|user|user_id 이렇게 넣으면 된다. (restart kubeapiserver 필요)
- (2) static token file : 위에 방법이랑 같은데, pwd 에다가 encoding 된 token 넣어서 csv 만들면 된다. 그리고, --token-auth-file 이라는 항목을 kube-apiserver.service 에 넣어주면 된다.
- 위의 두가지 방법은 recommended 방법이 아니고, consider volume mount while providing the auth file in a kubeadm setup

140. TLS
- user 가 server 로 보낼때 encrypt 해서 data 를 보내서 중간에 해킹안당하도록 도와주는 기능. server 가 decrypt 하기 위해서는 key 가 가야함. ASYMMETRIC ENCRYPTION (private key-public lock pair 로 decrypt 가능)
- ssh-keygen 하면 id_rsa 랑 id_rsa.pub 생기는데, pub 이 바로 public lock 이다. 이걸 server 에 주고, 서버가 데이터를 암호화해서 나한테 보내면 나는 private key 로 복호화 할 수 있다. (서버에서 cat ~/.ssh/authorized_keys 하면 보인다(단, 복수개 보일 수 있다.))
- 내가 데이터를 보낼때 나의 private key 로 암호화해서 보내려면 서버는 나의 private key 를 가지고 있어야 복호화가 가능한데 (symmetric), 이렇게 하기 위해 서버가 asymetric 으로 pub key 를 만들어서 나한테 주면 나는, 그 pub key 로 내 private key 를 암호화 해서 넘겨주면 서버는 나의 private key 를 갖고 있게 된다. 그래서 나중에 내가 나의 private key 로 암호화해서 넘겨주면 서버는 나의 private key 로 데이터를 복호화 하게 된다(인증 되서 나인걸 앎)
- TLS = SSL 인데, SSL 인증서는 client 와 server 간으 ㅣ통신을 제3자가 보증해주는 전자화된 문서. 이것의 사용으로 해커 위험 줄이고, 접속하려는 서버가 맞는지 확인하고, 악의적 변경 방지 가능
- decrypt 를 위해서는 key 가 필요한데, 양쪽이 같은 key 가지고 있으면, symmetric 이고 다른거 가지고 있으면 asymmetric 
- symmetric 은 key 값을 보내야 하는데, 해킹당하면 data 보장 안됨
- pub key 는 대칭키 방식을 보완! A 키로 암호화를 하면 B 키로 복호화 할 수 있고, B 키로 암호화 하면 A 키로 복호화 할 수 있음. 여기서 pub key 는 타인에게 공개. 그래서 서버에서 나한테 정보를 보낼 때, pub key 로 암호화 하면 나는 비밀키로 decrypt 할 수 있음. 공개키로는 암호화는 할 수 있지만, 복호화는 할 수 없다.
- 공개키를 이용한 인증? 인증이랑 정보를 보낸 사람이 올바른 사람임을 확인하는 것. 이번에는 비공개키로 암호화하고, 공개키를 가지고 있는사람에게 정보 전송. 만약 공개키를 가지고 정보를 복호화에 성공했다면, 그것은 비밀키를 가지고 있는 사람이 보낸 정보라는 사실을 확인(인증)할 수 있게 되는 것! 이것이 인증의 원리. 
- 인증서의 역할은 client 가 접속한 server 가 client 가 의도한 서버가 맞는지 보장하는 역할. 이 역할을 하는 민간기업들이 있는데 이 기업들을 CA(Certificate Authority) 혹은 RC(Root Certificate) 라고 한다. 
- 해커가 router 를 돌려서 우리가 자신의 사이트에 접속하게 하고, 자신의 pub 키로 우리의 private 를 암호화해서 달라고 할때 우리가 그걸 피하기 위해 쓰는게 인증서! pub key 와 함께 온 인증서를 보고 제대로 된 서버에서 왔는지 확인!!
- 인증서는 해커가 만든 웹서버가 제대로 된 서버인지 확인하고, 인증서 부여. 즉 해커는 제대로 된 인증서를 받을 수 없음
- public key 확장자 : *.crt 나 *.pem
- private key 확장자 : *.key, *-key.pem

142. TLS in k8s
- (1) server certificates(priv,pub) / (2) client certificates (priv,pub) / root certificates (symantec 같은)
- (1) Server Certificates : kube-api Server 는 apiserver.crt(pub), apiserver.key(priv) 가지고 있고, ETCD Server, KUBELET Server 도 마찬가지로 각각의 .crt 와 .key 가지고 있음
- (2) Client Certificates : kubectl REST API 쓰는 admin. admin.crt 랑 admin.key 있음, KUBE-SCHEDULER 도 client! scheduler.crt, scheduler.key 있음. KUBE-CONTROLLER-MANAGER 도 마찬가지고 KUBE-PROXY 도 마찬가지! 그들 모두 own pub,priv key 있음
- ![Screenshot](image/cert_flow.png)
- apiserver-etcd 통신 필요한애나, apiserver-kubelet 통신에 필요한 pub,priv 있음
- api-server 에 접근하는게 **authentication** 이고, 접근 이후 필요한 자원 조회 같은 권한이 **authorization**. 권한까지 문제가 없으면 admission control.
- API SERVER 에 접근하는 방법 3가지 (1) x509 Client Certs (2) kubectl (3) SA
- (1) x509: k8s api server 6443 접근하려면 권한 필요한데, 설치시 kubeconfig 라는 곳에 인증서 내용 있어서 그거 복사해서 user 가 가지고 있으면 접근 가능. kubeconfig 안에 있는 키들 만드는 법 : 최초 발급 기관 개인키 (CA key) 와 클라이언트 개인키 (Client key) 를 가지고 인증서를 만들기 위한 인증 요청서(CSR) 을 만듬. CA csr 로 바로 CA crt 만들고, Client crt 는 CA key+CA crt+Client csr 로 만들어짐. 이 관계 자주 나오니 숙지할것. 그리고 kubectl 은 kubeconfig 를 다 가지고 있어서 api server 에 접근 되는 가능! 만약 kubectl 에 proxy 를 열면 밖에서 거기로 kubectl 접근 가능하고, 밖에서 접근한 사용자는 client key 와 client crt 없어도 kubectl 이 다 가지고 있어서 apiserver 접근 가능
- --> 마스터 노드의 인증서는 /etc/kubernetes/admin.conf 에 있다.
- (2) kubectl : 외부서버에서 kubectl 를 설치해서 multi cluster 접근하는 방법은 접근 하려는 clutser 의 kubeconfig 가 나한테 있어야 함. 이 kubeconfig 는 clusters 와 users, 그리고 이 둘을 정하는 contexts가 있음. 그래서 kubectl config user-context context-A 하면 context-A 에 적힌 cluster 로 user 값을 가지고 접근 하게 됨.
- (3) SA : k8s cluster 에는 api server 가 있고, ns 만들게 되면, default 라는 SA 가 자동으로 만들어짐. 그리고 이 SA 는 secret 이 하나 달려있는데, CA crt 정보와 token 이 있음. 이상태에서 pod 를 만들면, pod 는 SA 에 달린 secret 의 token 값으로 api Server 로 연결. 결국 이 ns 의 SA 에 달린 Secret 안의 token 값을 알면 외부의 user 는 바로 api server 에 접근 가능.
- k8s 가 자원의 권한을 지원하는 방법중 가장 중요한게 역할기반으로 권한부여하는 **RBAC**!!
- ** Authentication 잠깐 설명하면**
- 1. Cert
- ![Screenshot](image/1crt.png)
- 1-1. k8s 설치시에 k8s API SErver 가 kubeconfig 라는 파일을 넣게 되는데, 여기에 CA crt(발급기관 개인키), Client crt, Client Key(클라이언트 개인 키) 등이 모두 있다. 만약에 외부의 사용자가 이 kubeconfig 를 복사해서 가지면 외부에서 k8s 에 접근 할 수 있게 된다.
- 1-2. CA crt 는 외부에 있는 CA key(발급기관 개인키) + CA csr(인증요청서) 로 만들어져서 k8s API Server 의 kubeconfig 안에 들어온 것이다.
- 1-3. Client crt 는 CA key + CA crt + Client csr 로 만들어 진다. (csr 은 인증요청서 인데, 이건 최초에 자동으로 만들어짐.)
- 1-4. 이렇게 만들어진 kubeconfig 가 kubectl 안에 들어있기 때문에, k8s API Server 에 접근 가능하다.
- 1-5. 만약 kubectl 에 proxy 로 외부에서 접근 가능토록 만들어 놓으면, 외부에서 거기로 kubectl 칠 수 있고, 그러면 k8s API server 접근 가능.
- 2. kubectl
- 2-1. 외부에 kubectl 깔고, 다른 cluster 에 접근하려면, 외부에 깐 kubectl 에 두 cluster 의 kubeconfig 있어야 한다.
- 2-2. 각 kubeconfig 안에는 contexts, clusters, users 가 있어야 하고, 알맞은 인증서 들이 있어야 한다. 
- 2-3. 그러면 외부에서 context 지정으로 원하는 cluster 에 접근 가능하다.
- ![Screenshot](image/2kubectl.png) 
- 3. SA
- 3-1. cluster 에서 ns 를 만들면, 자동으로 default 라는 SA가 만들어짐. (여기는 CA crt 랑 토큰값이 들어있음.) 그리고 secret 이 만들어지는데, SA 가 이 secret 에 연결됨.
- 3-2. pod 를 만들면 sa 에 연결되고, 파드는 토큰값을 통해 k8s API server 에 연결됨.
- 3-3. 결국 외부의 사용자도 ns 의 토큰 값만 알면, 해당 ns 에 접근 가능
- ![Screenshot](image/3sa.png) 
- ** Authorization 잠깐 설명하면**
- 1. RBAC (Role, RoleBinding) (role 과 rolebinding 이라는 obj 기반으로 역할 기반으로 권한 부여하는 기능!!)
- 1-1. ns 만들때 생기는 SA 에 어떻게 권하는을 부여하는지에 따라 ns 만 접근 가능할지, clutser 까지 접근 가능한지 제어 가능
- 1-2. role 을 read 혹은 write 중 특정 애들만 써서 권한 부여할 수 있고, rolebinding 을 하나의 role 들을 sa 에 묶어주는 역할 (여러개에 묶을 수 있음)
- 1-3. 만약 clutser 단위 제어하려면 cluster role 과 cluster rolebinding 쓰면 됨.
- 1-4. 우측 그림처럼 rolebinding 이 clusterrole 에 묶일 수도 있지만, cluster 는 전체 단위로 관리해야 해서, clusterroleBinding 을 쓰는게 더 이상적
- ![Screenshot](image/4rbac.png) 
- 

143. TLS in k8s - cert creation
- easyrsa, openssl 등 있지만, openssl 쓸게
- 


146. KubeConfig
- 원래 kubectl get pods --server my-kube --client-key admin.key --client-crtificate admin.crt --certificate-authority ca.crt 이렇게 써야하는데, 이 뒤에 있는 것들을 kubeconfig 파일에 넣은 것! 그래서 pod 조회 기능 가능 한것. ($HOME/.kube/config 에 있다)
- kubeconfig file 은 cluster, context(어떤 user 가 어떤 cluster 쓸지 정함), users(다른 previlege on diff cluster 가짐) 이렇게 3개로 구성!
- ![Screenshot](image/kubeconfig.png) 
- kubeconfig 파일은 실제로 아래와 같이 context 에서 연결하고 위 아래로 list 로 구성
- ![Screenshot](image/kubeconfig2.png) 
- 위 yaml 의 kind 아래에 current-context 에 현재 쓰는 context 명시할 수도 있고, kubectl config view 하면 보이기도 함
- kubectl config view --kubeconfig=my-custom 이런식으로 지정해서 볼 수도 있음
- kubeconfig use-context prod-user@production 이런식으로 해서 바꿀 수도 있다.
- 아래처럼 kubeconfig 에 base64 로 encoding 된 정보를 직접 넣을 수도 있다.
- ![Screenshot](image/kubeconfig3.png)
- kubectl config --kubeconfig=/root/my-kube-config use-context research 이렇게 해서 쓸 config 파일 지정하고, 그안에 context 지정할 수 있음
149. API groups
- k8s 에서는 apiserver 를 거의 거친다!
- core group 에 all core functionalities 가 존재한다. (ex. namespace, pods, nc, events ... )
- named group api 는 더 organized! --> 새로운 기능들은 이걸 통해 사용 될 것.
- 아래와 같다.
- ![Screenshot](image/apis.png)
- apiserver 에 접근할때 그냥 curl 을 쓰면 crt 정보들 명시줘야 하는데, 만약에 kube-proxy 에 해당 내용들 담아 두면 proxy의 port 를 통해 접근하면 바로 apiserver 사용 할 수 있다.
- ![Screenshot](image/connect.png)
150. Authorization
- 우리가 developer 한테는 delete 안주거나, service account 한테는 get 도 못하게 하거나 하는 기능이 authorization
- restrict access to client 하는게 authorization
151. RBAC
- role obj 로 역할 넣을 수 있다.
- role 에는 apiGroups / resources / verbs 넣어야 하는데, core groupd 은 apiGroups 를 빈칸으로 하면 된다.
- 만약 configmap 도 만들게 하려면 아래처럼 role 에 두개 넣으면 된다.
- kubectl create -f 로 만든 role 을 user 에게 연결하려면 아래 그림의 아래 처럼 rolebinding obj 를 만들어주면 된다. usbject 에 user detail 넣어주고, roleref 는 우리가 만든 role 의 detail 을 넣어주는 곳
- ![Screenshot](image/rbac.png)
- kubectl auth can-i create deployment 같이 'auth can-i' command 로 가능 여부를 확인 할 수 있다. kubectl auth can-i create deployment --as dev-user 처럼 dev-user 라는 사람의 권한 여부도 확인 가능하다
- role 을 특정 pod 만 관리 하도록 줄 수도 있다.
- ![Screenshot](image/rbac2.png)
- kubectl describe pod kube-apiserver-controlplane -n kube-system 를 통해서 api-server 의 authorization-mode 확인 가능하다.
153. Cluster Roles and Role Bindings
- ![Screenshot](image/clusterrolebinding.png)
155. Service Account
- service account 는 used by an application to interact with k8s cluster
- 예를들면, prometheus 도 service account 써서 k8s 자원 상황 보게 된다.
- jenkins 도 이거 보고 관리한다.
- k8s dashboard 도 마찬가지인데, has to be authenticated
- 만드는 법: kubectl create  serviceaccount dashboard-sa
- 확인 : kubectl get serviceaccount
- 이거 만들어지면, token 바로 만들어진다. k describe sa dashboard-sa
- 순서 : (1) sa obj 만들면, (2) generate token for sa, (3) secret 만들고, (4) stores that token into secret (5) link to sa
- token 보려면 describe sa 에서 나온 token 이름으로 kubectl describe secret dashboard-sa-token-kbbm 해야함
- 그리고 api-server로 curl 쓸때 header 에 "Authorization: Bearer {token}" 하면 명령어 쓸 수 있다.
- 모든 ns 에는 default 라는 sa 가 자동으로 생기고, 이 안에 있는 token 을 통해 외부에서 접근 가능하다.
- pod 를 만들때 쓰고자 하는 곳의 sa 를 mount 해서 header 에 매번 넣을필요 없게 만들수도 있다. 예시가 dashboard 이다. dashboard 는 만들어질때 describe 해보면 /var/run/secrets/kubernetes.io/serviceaccount from default-token-4alik 이런식으로 자동으로 mount 된다.
- pod 에 spec>serviceAccountName 이라는 항목 넣어서 만든 sa 지정할 수도 있다.
157. Image Security
- private registry 쓰려면, worker node 에서 쓰기 때문에, docker-registry secret 을 만들고, 아래와 같이 pod 안에서 spec>imagePullSecrets 인자로 넘겨야 한다.
- ![Screenshot](image/imagesecurity.png)
- kubectl exec ubuntu-sleeper -- whoami 라는 명령어를 통해 ubuntu-sleeper pod 의 실행자 누군지 알 수 있다.
161. Network policy
- 한 서버 기준에서 자기한테 들어오면 ingress 고, 나가면 egress!
- 한 pod 에 다른 pod 에서 못들어오게 하려면? network policy obj 를 써라!
- 만들어진 network policy obj 를 원하는 pod 에 연결하면 그 pod 에는 'allow ingress traffic from API Pod on Port 3306' 이런식으로 정책을 줄 수 있다. 그러면 matches to specified policy 만 들어올 수 있음
- link 하는법도 label 과 selector 쓴다.  networkpolicy obj 에 podSelector: matchLabels: {}:{} 이런식으로 하고, pod 에 labels: {}:{} 이런식으로 matching 시켜서 link! (Flannel 은 networkpolicy obj 지원하지 않음. calico, kube-router, weave-net 은 지원)
- ![Screenshot](image/networkingpolicy.png)
- 위의 예시에서 ingress 만 쓰면 되는 이유는, ingress 로 들어온 딱 api 에는 결과 보내줄 수 있기 때문!
- 다만, make calls 를 직접 db 에서 할 수는 없다. return 만 줄 수 있을 뿐!
167. docker storage
- 도커가 data 를 file system 에 저장하는 법? 인스톨하면 create folder structure : /var/lib/docker. 여기는 aufs, containers, image, volumes 등이 들어감
- 도커파일안에 있는 한줄 한줄이 docker layer임! 이렇게 layer 구조로 하면, 새로운 dockerfile 을 통해서 image 만들때 용량 효율적으로 사용 가능(reuse the image that was made previously). 환경은 그대로고 코드만 바꾼 경우는 정말 효율적
- /var/lib/docker/volumes 안에 data_vol1 이라는 dir 을 만들면, docker run 할때 -v data_vol1:/app/data 이런식으로 해당 dir 이름만 가지고 마운트 할 수 있음 (일반은 complete path 쓰면 됨)
- --mount 가 요즘꺼고, -v 는 옛날꺼 ㅋㅋ
- -mount type=bind,source=/data/data_vol1,target=/var/lib/mysql 이렇게 type,source,target 이라고 명시적으로 사용
- 이런 mount 행동을 담당하는게 storage driver 인데, ubuntu 는 기본적으로 AUFS 쓰고, fedora는 Device Mapper 쓰고, 이외에 ZFS, BTRFS, Overlay, Overlay2 등도 있음 
168. Volume Driver Plugins in Docker
- 위에꺼는 storage driver 고, local 안쓰고, aws ebs 같은거 쓸때는 docker run 에 --volume-driver rexrays/ebs 이런식으로 명시해줘야 함
169. Container Storage Interface(CSI)
- k8s 가 옛날에는 container runtime 에 docker 만 썼는데, 이제는 rkt, cri-o 같은거도 씀
- 그래서 CRI(Container Runtime Interface) 도 신경써야 함
- 유사하게, CNI(Container Networking Interface) 도 weaveNet, flannel, cilium 등 여러개 있음
- 유사하게 CSI(Container Storage Interface) 도 신경써야 함
171. volume
- ![Screenshot](image/volume1.png) 
- 위의 그림처럼 volumes 에 hostPath 를 정하고, pod 를 띄우면, 어떤 노드에 뜰지 모르기 때문에 데이터가 일관적으로 유지되기 어렵다.
- ![Screenshot](image/volume2.png) 
- 하여 위의 그림처럼 volumes 에 hostPath 를 aws 로 수정하면 외부에 있는 aws storage 를 공통적으로 쓸 수 있게 된다.
- 아래는 pv-pvc 를 사용하는 예제이다. 여기서 중요한 점은, pvc 는 pv 가 어디에 되어있느냐에 따라 pod 와 함께 pv 가 떠있는 곳에 뜬다는 점이다.
- ![Screenshot](image/volume3.png) 
- 아래의 예시와 같은 경우, pv 는 k8s-node1 에 /node-v 를 마운팅 하려고 준비하고 있기 때문에 여기 연결되는 pvc 와 pod는 k8s-node1 에 뜨고, 해당 node 의 /node-v 라는 위치를 마운팅 하게 된다.
- reclaimpolicy 는 아래와 같다.
- ![Screenshot](image/reclaimpolicy.png) 
183. Switching Routing
- ![Screenshot](image/switching.png) 
- ip link 로 host 서버의 네트워크 확인 가능
- 자신의 ip 를 ip link 에 나온 dev 에 연결하는 명령어 ip addr add ... dev ...
- 하나의 node 가 있는데, 해당 노드가 한 network 에 묶여 있을때, 같은 network 에 있는 노드와 통신 가능
- ![Screenshot](image/gateway.png) 
- 그런데, 다른 network 에 묶여 있는 노드한테 트리거 주려면 router 필요. 
- router 가 다른 네트워크의 통신을 gateway 를 통해 가능케 해준다. newtork 가 방이라면, gateway 는 문. router 에 ip route add 로 destination(방) via origin(문) 해주면 origin 의 라우터 ip 를 통해서 destination 으로 갈 수 있게 해준다.
- route 명령어 치면 커널 라우팅 테이블(통신 가능한 애들) 볼 수 있다.
- 구글처럼 외부에 접속할때도, 구글ip(네트워크) via origin 하면 origin 을 통해서 갈 수 있다. (GW)
- default 는 0.0.0.0 인데, any network 다 접근 가능하다는 뜻이다. 즉, default via 192.168.2.0 하면 192.168.2.0 으로 바깥세상 다 갈 수 있는거고, 192.168.2.0 via default 하면 어디서든지 192.168.2.0 에 접근 가능하단 뜻.
- ![Screenshot](image/router.png) 
- A 에서 C 를 갈때, B 를 경유해야 한다면, ip route add C_Network via A_gateway, 이거랑, ip route add A_Network via C_gateway 이렇게 해준다. 그리고 forward packet allow 해줘야 한다.
- echo 1 > /proc/sys/net/ipv4/ip_foward 해주면 /etc/sysctl.conf 에 net.ipv4.ip_forward=1 로 바뀌고, 결과 받을 수 있다.
- ip addr add 한거는 껏다키면 날라가니까 /etc/networks 에 해줄필요 있다.

184. DNS
- /etc/hosts 에서 DNS 관리 되는데, 중복 가능하다. 그런데, node 가 많아지면 각각 /etc/hosts 에 가지고 있는게 비효율적이라, DNS 서버를 둔다.
- 이런 경우 이름이 중복되면 local 의 /etc/hosts 가 우선권을 가진다. 만약 DNS 서버가 우선권 가지게 하려면, /etc/nsswitch.conf 에서 hosts: files dns 이렇게 바꿔주면 된다.
- dns 서버는 /etc/resolv.conf 에 적혀있는데, local dns 는 보통 192.168.1.100 이다. 구글같은 일반 사이트를 가진 dns 는 8.8.8.8 인데, resolve.conf 가면 8.8.8.8 있다.
- ![Screenshot](image/domainnames.png) 
- 도메인 이름 규칙 있는데, 마지막은 commercial 이면 com, network 면 net, education 이면 edu같은 식이다.
- www 가 있는 subdomain layer 에서 google 의 기능 나눠진다. maps.google 이나 driver.google 처럼
- ![Screenshot](image/domainnames2.png) 
- 위처럼, 전체를 차장 가려면, .com dns 가서 구글 dns 가서, app.google.com 의 ip 찾아와야 하는데, 이 과정 반복하지 않고, 캐싱됨
- ![Screenshot](image/domainnames3.png) 
- 앞자리 짤라먹고 하고 싶으면 resolve.conf 에 search 로 넣어두기
186. network ns
- 하나의 host 에 red 랑 blue 라는 ns 만들면, host 에서 ip link 하면 red 랑 blue 안에 있는 veth 안보이고, 반대로 red 랑 blue 에서도 host 꺼 안보임
- red 랑 blue 랑 연결하고 싶으면 router 놓는거처럼 해야 함 : ip link add veth-red type veth peer name veth-blue
- red랑 blue 에 문 놓기 : ip link set veth-red netns red, ip link set veth-blue netns blue
- 그리고 veth-red 랑 veth blue 랑 연결해야 함 : ip -n red addr add 192.168.15.1 dev veth-red, ip -n blue addr add 192.168.15.2 dev veth-blue
- 올리기 : ip -n red link set veth-red up, ip -n blue link set veth-blue up
- 보내기 : ip netns exec red ping 192.168.15.2
- 여러개 netns 가 생기면 어떻게 다 연결?? bridge 놓기 : ip link add v-net-0 type bridge
- ip link set dev v-net-0 up
- 그리고 다 v-net-0 에 연결 
- (1) 네트워크 만들기: ip link add veth-red type veth peer name veth-red-br
- (2) 실제 연결 : ip link set veth-red netns red, ip link set veth-red-br master v-net-0
- (3) 문에 ip 달기 : ip -n red addr add 192.168.15.1 dev veth-red
- (4) up : ip -n red link set veth-red up
- host 에서 ping 192.168.15.1 해서 red 로 바로 연결은 안되지만, v-eth-0 통하면 됨. 
- (5) veth 에 host 연결 가능한 ip 배정 : ip addr add 192.168.15.5/2 dev v-net-0
- (6) ping 192.168.15.1 성공!
- host 말고 바깥 세상의 lan 으로 연결하는 법? eth0 통하기
- (1) ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
- (2) ip netns exec blue ping 192.168.1.3 하면 보내지긴 하지만 받진 못함
- (3) iptables 쓰기
- 바깥 8.8.8.8 이랑 연결하려면?
- (1) ip nets exec blue ping 8.8.8.8 안됨!
- (2) ip netns exec blue route 하면 없음
- (3) ip netns exec blue ip route add default via 192.168.15.5 (내 문)
- (4) ip nets exec blue ping 8.8.8.8
===

000. inflearn2
- k port-forward http-go 7777:8080 하면 밖의 7777 에 http-go pod 의 8080 을 연결한다.
- k annotate pod http-go test1234=test1234 하고 k get pod -o yaml 로 확인해보면, annotations 가 metadata 에 들어간걸 확인 가능하다.
- k get pod --all 하면 모두 가져오고, k delete pod --all 하면 다 지움
- 
===

001. POD 중급
- ![Screenshot](image/lifecycle.png)
- [1] Phase : Pending
- (1) conditions Initialized : pod 에 사전 세팅이 필요한 경우 initContainer 라는 초기화 script 를 넣을 수 있고, 이 스크립트가 본 컨테이너보다 먼저 실행되고 성공하면 initialized 가 true 가 됨. 아니면 False
- (2) conditions PodScheduled : pod 가 어디에 올라갈지 node scheduling 되면 PodScheduled 가 True 가 됨
- (3) conditions containerReady : 이 다음에 image download 되는데, 위의 두 단계 동안, container 는 waiting 이고, 이유는 containerCreating 임. 다 down 되고, 정상기동 되면
- [2] Phase : Running
- 다운까지 되면, Running 이 되는데, 만약 container 가 CrashLoopBackOff 에 걸리면, running 이지만, 내부에 containerReady: False, Ready: False 가 된다. 잘 돌아가면, 둘다 True 로 변경됨
- [3] Phase : Failed / Succeeded
- 만약 작업중인 container 가 문제 생기면 failed 고, 혹은 작업 끝나서 terminated 되면 succeeded 가 됨
- phase pending 중에 failed 로 갈 수도 있고, pending 이나 running 일때 통신 장애 생기면 unknown 갔다가 지속되면 failed 갈 수도 있다.
- ![Screenshot](image/readinessliveness.png)
- readiness 필요 경우: 사용자가 서비스에 들어올때 pod 가 재실행 될때 잠깐의 순간에 못들어오게 막으려면 readinessprobe 쓰기
- pod 가 장애가 났는지 계속 health check 를 하려면 livenessprobe 달아줘야 함
- 둘은 사용 목적이 다르지, 방법은 같음. httpget, exec, tcpSocket 으로 해당  app 의 상태 확인
- ![Screenshot](image/readinessliveness2.png)
- 이 셋중 하나는 꼭 정해야하고, 우측의 설정값은 안넣으면 default 로 됨
- 아래는 readinessprobe 의 예제이다. /readiness/ready.txt 가 안생기면 준비 완료라고 안보고, 생기면 3회 보고, 성공하면, containerReady 와 Ready 를 True 로 바꾸고, endpoint 도 addresses 인식하면서 서비스에서 연결시킴
- ![Screenshot](image/readinessliveness3.png)
- 아래는 livenessprobe 의 예제이다. /health 라는 api 에 계속 명령 날리면서 500 에러를 3번 받으면 문제있다고 인지하고, pod 를 restart 시킨다.
- ![Screenshot](image/readinessliveness4.png)

002. volume 중급
- storage class 사용 시 
- (1) 설치하고, deployment 내에 spec.containers.env 에 DISABLE_SCHEDULER_WEBHOOK 을 TRUE 로 수정해야 함 --> 이걸 안하면, PV 만들때 StorageClassName 에 "" 만 넣으면 pv 못찾고 에러남!
- (2) 관리 계정을 위한 secret 생성하고, 
- (3) storageOS 설치
- (4) 외부에서 접근 가능토록, k edit svc storageos -n storageos 해서 spec 에 externalIPs 와 MasterIP 추가
- 그럼 아까 만든 secret 안에 id /pw 로 접속 가능
- (5) default storageclass 추가 (is-default-class:"true" 되어 있는 obj 추가)
- 그럼 k get storageclasses.storage.k8s.io 로 디폴트 확인 가능
- 

===

교육 중 
- container 를 사용하면, os 가 두번 깔리는게 아니라, 컨테이너 내부에서는 host os 의 커널을 사용함. os 가 실제로 깔리는건 아니고, apt, yum 이런걸 쓸 수 있게 해주는 정도임
- 격리 기술을 위해 chroot 를 사용. chroot 를 쓰면 /root 가 내가 원한 곳으로 설정되고 그 이상 위로 못올라감.
- 네임스페이스는 커널이 가지고 있는 기능인데, process 에게 os 안에 있는 객체를 구분하기 위한 ns 가 있는데, 대부분은 하나의 ns 를 공유. 시스템의 모든 process를 하나의 ns 를 공유하기 때문에, 다 보임. 우리의 목적은 나누는거라서 ns 사용
- ns : 동일한 sys 에서 수행되지만, 독립된 공간처럼 격리된 환경을 제공하는 linux kernel 의 경량 프로세스 가상화 기술
- hypervisor 는 호스트의 HW 자원을 가상화
- NET ns? NET 은 통신이 목표고, ns 는 격리가 목푠데? -> 각 ns 별로 개별적인 interfase IP 구성을 가짐. 중복 포트 바인딩을 통한 net 충돌 방지
- 도커를 하면 내부와 로컬의 veth 가 연결되고, 이게 bridge network(docker0) 에 연결되고, 물리 랜카드에 연결 eth0. bridge 왜 있냐고? veth 가 가상일뿐이고, docker 마다 생기는데, 이게 다 물리 랜카드에 1:n 으로 묶일 수 없다.
- ![Screenshot](image/dockernet.png)
- 컨테이너는 프로세스일 뿐이다. 가상화되고 격리되어 동작하는 프로세스일 뿐이다.
- 도커가 생기면 process 가 생긴건데, 그 process 는 chroot /var/lib/docker/overlay2/난수/diff 여기이다. 여기 아래에 컨테이너 내에서 만든 파일들, 디렉토리들 만들어지지만, 휘발성이다.
- swap 은 당분간 사용하지 않을 메모리에(RAM) 프로세스를 swap(disk) 에 넣어서 잠시 보관하여 ram 효율 높이기 위해 쓰는건데, io 가 생김. 그런데 k8s 는 빠른 연산을 위해 모든 memory 를 다 쓰길 원해서 swap off 해야 함.
- 그리고, volume mount 하면 /var/lib/docker/volumes/{지정한vol이름}/_data 에 생긴다!
- 즉, docker run -it -v temp111:/app centos 이렇게 하면, temp111 이라는 폴더가 /var/lib/docker/volumes 아래에 생긴다. 만약에 현재 자신의 위치에 temp111 이 있더라도, 저 위치에 생기고, 만약 현 위치의 temp111 을 mount 하고 싶다면, 절대경로로 써줘야한다.
- static pod 는 kubelet 이 직접 실행하는 pod, 각각의 노드에서 kubelet 에 의해 실행
- pod 들을 삭제할 때, apiserver 를 통해서 실행되지 않은 스태틱pod 는 삭제 불가
- /etc/kubernetes/manifests 아래에 있는 yaml 은 바로 실행됨. 수정해도 바로 반영 됨. 여기가 static pod 의 기본 경로. 여기에 일반 pod 의 yaml 갖다 놓으면 자동으로 뜨고, 지워지지 않음
- 저 경로는 자동으로 뜬다는 내용은 /var/lib/kubelet/config.yaml 에 staticPodPath 에 적혀있음
- docker container prune -> 사용하지 않는 모든 컨테이너 삭제
- docker system df -> 우리의 도커 관련 시스템들이 얼마나 용량을 소모하고 있는지 보여주는 명령어!
- ![Screenshot](image/dockersysdf.png)
- Dockerfile 은 IaC(Infrastructure as Code)라고 부름. 원하는 인프라를 코드로 정리해 놓았다는 뜻
- control-plane = master node = 관제노드 : etcd 라는 명세들이 담긴게 여기 있음
- kubeadm 을 가지고 클러스터 구성할때 초기에는 고가용성(이중화) 지원하지 않았지만, 이제는 지원함
- k8s 쓰려면 bridge 모듈을 enable 해야함 (netfilter 라는 kernel engine 이 제어할 수 있도록 해야함 -> modprobe br_netfilter 해서 br_netfilter 가 나와야 함. 그러면 netkernel 이 bridge 를 제어 가능하다는 뜻. )
- ![Screenshot](image/sysctl.png)
- GPG : 쿠버는 원래 구글 사내에서 쓰던건데, 공용으로 나온거라서 받을때 pkg 위변조 안됐는지 확인하기 위해 gpg 키 받고, pkg 가져와야 함.
- 

=== mock exam
- k run nginx-pod --image=nginx:alpine
- k run messaging --image=redis:alpine -l tier=msg
- k get nodes -o json > /opt...
- k expose pod messaging --name messaging-service --port 6379 --target-port 6379
- k describe svc message-service
- k create deploy hr-web-app --image=kodecloud/webapp-color; k scale deploy hr-web-app --replicas=2
- k run static-busybox --image=busybox --command sleep 1000 --dry-run-client -o yaml > static.yaml; mv static.yaml /etc/kubernetes/manifests/static.yaml
- k run finance --image=redis:alpine --dry-run -o yaml > pod.yaml; namespace 추가
- k get pod orange -o yaml > orange.yaml; vi orange.yaml 해서 typo 고치기
- k expose deployment hr-web-app --name hr-web-app-service --type NodePort --port 8080 --target-port 8080 --dry-run=client -o yaml > pod.yaml; nodeport: 30082; k craete pod.yaml
- k get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/node_os.txt
- k explain pv --recursive | less 하면 pv 에 대해 알려주고, /hostpath 로 검색해서 인자 알 수 있음
- 