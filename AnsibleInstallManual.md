# 1. Ansible 개요
- Ansible이란 Infrastructure as a Code(IaaC) 방식을 지향하며 인프라 관리를 코드 기반으로 자동화할 수 있는 DevOps Tool의 일종이다.
- 가상화 기술 및 Public Cloud 분야에서 사용하면 유용한 도구라고 한다.
- 다른 IaaC 솔루션들과는 달리 Ansible이 갖는 특징은 다음과 같다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/358a2262-fd11-4aa2-9c64-c90b0b4a45c5)

- 전체 동작 개요는 다음 그림을 통해 확인할 수 있다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/d65046fa-521a-4b3f-9d1b-e32ae10839b4)

# 2. Ansible 설치 및 환경설정
- Ansible을 설치하고 환경설정을 하는 항목들에 대해 작성한다. 
- 우선, Ansible에서 사용하는 용어는 다음 표와 같다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/9e226e02-5bd6-4b07-b73f-32feec3a7cb6)
- Ansible 설치 절차는 Controller Server에 epel-repository를 설치하고 ansible을 설치하면 된
- 또한, Python 2.6버전 이상이 설치되어 있어야 한다.

### 2-1. 설치 과정
- 1. 우선 Controller server에 python을 설치한다.(네트워크에 연결되어 있어 외부 yum으로 설치 가능하다면 편리하게 설치 가능하다.)  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/1b351231-15e8-46b3-b61e-e7c0cea25ea3)
- 2. yum install –y ansible로 ansible을 설치한다.(이 때 epel-repository가 없다면, yum install –y epel-release 명령어를 입력하여 설치하면 된다.)  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/2bc10155-735e-4f93-9806-50fe02676c69)
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/aa66e1f2-cfac-4124-825e-5bcf25fa0ac2)
- 3. 설치가 완료되면 ansible --version을 입력하여 정상 설치 여부를 확인한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/ddc38548-da91-45a2-9238-8891cdc1ea21)
- 4. 관리 대상(원격 서버)에도 2.6버전 이상의 Python이 설치되어 있어야 하므로, 설치를 진행한다. yum으로 자동 설치가 안 되는 환경이라면 아래와 같이 진행해준다.(wget도 안되는 환경 포함)  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/698a31d2-41b0-4a6e-b215-2f7be82356d0)
- tar xvfz Python2.7 명령을 입력하여 압축해제 후 다음과 같이 파일이 존재하는 것을 확인한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/19758c05-36f1-4bcb-b0fc-623fa26ad49d)
- configure 파이썬 소스 빌드에 필요한 환경설정을 진행한다. Prefix 인자에 설치하고자 하는 경로를 넣으면 된다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/93085e8c-f87d-46ea-b16c-63999d95497f)
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/4a82c3be-0ac0-4744-bf10-822d9f9eb142)
- 결과적으로 다음과 같은 Makefile이 만들어져 있다면 성공이다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/0b911682-9206-4d20-a1aa-837152d45aeb)
- make 명령어를 실행하여 Makefile을 컴파일 후 빌드한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/6adbac2a-325c-49f2-bcfe-b6d808456482)
- make install 명령을 입력하여 빌드한 파일을 실행하여 설치한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/f7564c65-7f47-45a5-861f-779a6184e3f3)
- 이후 파이썬 버전을 확인해보면 가끔씩 Python 2.3, 2.4등 Ansible 관리 대상에 권장되는 Python 버전인 최소 2.6을 만족시키지 못할 때가 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/edc386a5-81a9-4c32-9cc4-11cf3a0fe689)
- 이럴 때에는 /usr/bin의 python에 걸려있는 심볼릭 링크가 python 2.6 미만 버전이기 때문이며, 이를 삭제 후 ln –s를 입력하여 심볼릭 링크를 생성하면 2.6버전 이상의 파이썬을 사용할 수 있다.
- 작업 결과는 다음과 같아야 한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/98065c42-bfcb-47e9-bbee-5c619471e95c)
- 위와 같이 관리 대상 서버에도 2.6버전 이상의 Python이 설치되어 있다면 Ansible을 이용할 준비가 완료된 것이다.

### 2-2. ssh-key 설정하기
- 2-1.의 과정에서 준비를 완료하고 난 이후 Controller 서버와 관리 대상 사이에는 ssh 인증서를 공유하는 설정을 해 주는 것을 권장한다.
- 그 이유는 Ansible은 SSH 접속을 통해 관리 대상 서버를 제어하는 방식인데, 이때 각자 서버의 계정, PW정보를 Controller 서버에 일일이 넣어주는 것은 아주 번거로운 일이다.
- 이 때문에 ssh의 공개키, 개인키 방식을 통해 공개키를 등록하면 ansible이 알아서 ssh 접속 및 playbook task 실행까지 해주기 때문에 거의 완전한 자동화가 가능하다.
- 설정 단계는 다음과 같다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/c6ce040e-4e02-4237-a7a2-71a16b6a2460)
- 우선 Controller Server를 대상으로 ssh-key 생성 및 등록을 진행하도록 한다. 
- ssh-key 등록을 위해서 ssh-kegen 명령어를 입력 후 모두 Enter를 입력해준다.
- 생성이 완료되었다면 ssh-copy-id [계정이름]@[등록 대상 서버IP] 명령어를 입력하여 대상서버의 authorized_keys 파일에 자동으로 내용을 등록한다.  
 ![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/cbfa0eab-87a2-472c-bc87-1f7ebf890f5a)
- 등록이 완료된 결과물은 다음과 같이 Password 입력 없이 ssh접속이 진행된다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/add69dc8-772e-446a-a2b4-1bbe84664899)

- 관리 대상 Server에서도 위와 같은 절차로 키 생성 및 등록을 진행해준다. 
- 관리 대상의 서버가 CentOS 4.x 버전이거나 일부 5.x 버전일 경우 ssh-key를 생성한 후 자동으로 등록하는 명령어가 먹히지 않을 가능성이 높다.
- 그럴 때에는 id_rsa.pub의 내용을 출력한 후 그 내용을 직접 대상 서버의 authorized_keys 파일에 등록하는 방법이 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/3bb532eb-6585-4341-b8a9-eb4cb21c662f)
- 위와 같이 ssh로 대상 서버에 로그인 한 후 authorized_keys 파일 및 .ssh 디렉터리의 권한을 바꿔준다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/ec5f4d2f-de98-4360-b676-1108af34c9ac)
- 이후 authorized_keys 파일에 복사한 public key를 등록한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/0c20ed75-d6ab-40b0-8445-8387a29f2a47)
- 등록이 완료되었다면, ssh 접속을 다시 시도한 후 완료되는 절차를 확인한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/baf9426b-d4df-441f-88ae-1134e85f3078)
- 여기까지 하면 Ansible 운영 매뉴얼을 참고하여 환경설정을 진행해주면 된다.

# 3. EPEL Repo 사용하지 않고 Ansible 설치
- 앤서블 패키지 자체는 EPEL Repository에 포함되어 있기 때문에 epel-release를 설치 후 install 해주는 것이 기본적인 방법이다.
- 하지만 epel은 10000개가 넘는 패키지를 가지고 있다. 오픈된 네트워크이고 외부 Repository를 가져올 수 있는 환경이라면 2절의 절차를 그대로 실행해도 된다.
- 하지만 폐쇄망인 경우(외부 yum repo 사용 불가) 10000개가 넘는 패키지를 전부 가지고 다녀야 한다.
- 또한 의도치 않은 업데이트가 진행되는데 yum은 기본적으로 EPEL에 현재 사용하는 패키지의 최신 버전이 들어있으면 자동으로 업데이트를 진행하기 때문이다.
- yum에서 repo를 무력화해서 업데이트를 방지하는 것도 가능하기는 하다. 하지만 잊어버린 경우 커널 단의 시스템이 업데이트가 되면서 버전이 꼬이면서 안될 수도 있다.
- 그래서 Ansible을 폐쇄망에 설치할 수 있는 방법은 2가지가 있는데 하나는 rpm을 파일로 가지고 다니면서 설치, 다른 하나는 pip를 이용한 설치이다.

### 3-1. rpm 파일로 설치
- 수많은 시도 끝에 성공했다. 과정을 돌아보면 여러 상황을 접하고 에러를 해결해보려는 시도를 할 수 있어 유의미한 시도였다.
- 우선, ansible 관련 rpm 파일을 다운로드 받아 이를 tar로 묶어줬다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/2fb4c018-8521-49fe-b437-9d502314680d)
- 이후 서버로 파일을 옮긴 후 압축해제 한 결과는 다음과 같다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/e6726486-473c-4224-adad-41d6dfc940c7)
- 이후 ans.repo.(yum 패키지에서 참조하는 repository 파일이다.)라는 파일을 작성하고 yum clean all 명령을 입력한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/174d3644-3f0e-44b6-a72b-d3fa94cdcd45)
- 그 다음 yum install을 시도한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/a73575f3-9246-491c-b67b-3ac558f3f067)
- 위와 같은 에러가 발생하며 설치를 더 이상 진행할 수가 없는데, 그 이유는 /var/www/html/ans 디렉터리 아래에 repodata/repomd.xml이라는 파일을 열어 볼 수가 없다는 것이다.
- 해결 방법은 createrepo를 통해 repomd.xml을 생성한 후 패키지를 설치함으로써 해결하였다.
- 이 오류를 해결하는 과정을 통해 yum은 패키지를 설치하며 repository의 repodata/repomd.xml 파일을 참조함을 알 수 있었다.
- createrepo 명령으로 repomd.xml을 생성한다. 명령이 동작하지 않는다면 패키지 설치를 별도로 진행하면 된다.(local yum or 외부 yum)  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/a2d68276-dd80-4478-bdda-010c6f64f3e7)
- ansible 설치를 시도한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/84ad3de2-ee23-4fa4-bc0c-f400ea0e8cfa)
- 버전을 확인해보면 아주 완벽하게 설치가 잘 된 것을 알 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/3d4af1a9-66ba-4781-91b1-1991fd3249f8)

# 4. 맺음말
- Ansible의 구체적인 운영 방법에 관한 문서는 Ansible 운영 매뉴얼을 참고하길 바란다.
- 만들면서 느낀 점이 있다면 다른 Tool 또는 패키지 및 관련 SW등은 설치 과정에서만 핵심적인 부분을 담아서 매뉴얼을 만들어도 최소 10페이지가 넘는 워드 문서가 나온다.
- 하지만 Ansible은 Controller에만 Ansible 패키지를 설치하고 나머지는 Python만 설치되어 있으면 자동화를 통한 관리가 가능하다는 점에서 설치 절차가 간편하다는 것을 느꼈다.
- Ansible은 거의 완전한 자동화를 지원한다고 한다. 
- 이유는 앤서블이 OS 설치단계에서 부터 자동화를 지원하지는 않기 때문이다.

