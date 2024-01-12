![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/367ace4d-71ea-4a54-9921-4f462d7ee364)1. Ansible 활용 방안
- Ansible을 활용하면 다수의 서버에 대해 일정한 작업을 자동화하고, 특정 패치에 관한 배포를 자동화 할 수 있는 등 다양한 활용 사례가 있다.
- 삼성 SDS에서 작성한 Ansible 활용 사례를 보면, AWS와 같이 보안이 취약해질 수 있는
Cloud 환경에서 외부 위협을 방어하는데 사용할 수 있다고 한다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/c6c8f5f2-632e-406f-9c0c-7896c8ddfd54)
- 아래 글은 SDS 기술문서에서 제안한 Cloud 환경에서 Ansible을 이용해서 보안을 강화할 수 있는 가장 간단한 작업의 사례이다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/cf062cdb-970f-4590-bb3f-197fd7d1eeac)


2. Ansible 운용
2-1. 운용 전 추가 환경설정
- Ansible 설치 매뉴얼 문서의 절차대로 설치가 완료되었다면, 일종의 환경 설정을 진행해야 한다. - 우선 Ansible이 제대로 동작하기 위해서는 Controller Server, Managed Node에 ssh-keygen 명령어를 입력하여 둘 다 ssh key를 생성하고 ssh-copy-id 명령어로 생성한 키를 등록해야 한다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/f5813be2-f56f-436d-af66-dd941b98699b)

- key 생성 시 위 화면처럼 출력되지 않으면 ssh-copy-id를 통해 key를 자동으로 등록할 수 없다.(보통 OS버전이 낮으면 많이 발생한다.)
- ssh key 등록에 성공하면 PW를 물어보지 않고 ssh로 로그인 할 수 있음을 확인할 수 있다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/e55a1cec-dedc-40f0-8389-e4964a2aacbb)
- 키 생성 후 자동 등록이 되지 않는다면 대상 서버 authorized_key 파일에 직접 등록해야 한다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/ba1a9882-6789-463f-a0f4-c925b9f1596d)
- ssh key를 직접 등록하는 방법은 별도의 외부 문서 참고
- 여기까지 완료해야 본격적으로 Ansible을 운용할 수 있다.


2-2. Ansible Inventory 설정
- 이번 절부터 진행하는 작업은 모두 Controller Server에서 하는 작업에 해당한다.
- Inventory는 관리 대상(Target Server 라고도 한다.)의 목록을 의미한다. 이 목록은 각각의 개별 노드가 될 수 있고, 여러 노드를 비슷한 성질에 따라 묶은 그룹이 될 수가 있다.
- 그 전에 /etc/hosts 파일에 alias 등을 등록해야 IP주소가 아닌 alias를 hosts 변수로 지정해서 편하게 유지보수 할 수 있다.
 
- 앤서블 설치 시 기본으로 지정되는 파일인 /etc/ansible/hosts 파일 또는 별도의 ini파일을 생성하여 관리대상 노드를 등록한다.
 
- 아래는 인벤토리 파일명을 –i 옵션으로 지정해서 실행하는 예시이다.
- ansible –i [인벤토리 파일명.ini] [Target Server Node/Group명] –m [ping 모듈] 형태로 Test한 것이다.
 
- 주의 할 점은 ansible/anisble-playbook 명령 실행 시 ansible.cfg 설정 파일의 inventory 값을 지정하지 않으면 default Inventory로 /etc/ansible/hosts 파일에 저장되어 있는 내용을 참조한다.
- ansible.cfg 파일을 다음과 같이 수정하면 Inventory를 지정하면서 실행하지 않아도 된다..
 

2-3. Inventory 작성 시 타겟팅 서버 그룹핑 원칙 및 설정 방법
- 타겟팅 서버를 그룹핑 설정 방법은 Inventory 파일 생성 방법에 포함되어 있는 내용에 있는 
인벤토리 파일(xipServer.ini) 하나를 가져와 설명한다.
 
- 그룹핑 설정은 위와 같이 [그룹이름] 아래에 포함하고 싶은 대상 서버의 이름 또는 IP 주소를 적어주면 [xipServers]라는 그룹 아래에 n1, n2, n3, n4, n5라는 이름의 관리 대상 서버가 포함된다.
- 이 때 이름으로 지정하려면 무조건 /etc/hosts 파일에 해당 이름에 매칭되는 IP주소 정보가 있어야 한다.
- 그룹을 지정하여 실행하려면 playbook에는 hosts 변수에 그룹명을 작성한다.
- Ad-hoc 명령에는 ansible [Target Server Node/Group명] –m [모듈명] –a [Ad-hoc 명령(모듈에 따라 달라짐)] 형식으로 작성하면 된다.
- /etc/hosts 파일에서 관리 대상 IP의 이름을 지정할 때 어떤 서버인지 구분하기 쉽게 이름을 명확하게 지어주는 것도 관리 용이성을 추구할 수 있는 방법 중 하나이다.
- 투입된 프로젝트 단위로 Inventory 파일을 분리해 작성하는 것 또한 좋은 방법이 될 수 있다.

2-3.1 플레이북 그룹 지정
 
- hosts 변수에는 IP주소 또는 Inventory 파일 내에 정의된 모든 이름(관리 대상 개별 이름, 그룹 이름)이 할당 가능하다.

2-3.2 Ad-hoc 명령어 그룹 지정
 
2-4. Ansible 기본 사용법
- 해당 절에서는 Inventory 작성 후 Ansible의 기본 사용법에 대해 기술한다.
- Ansible 사용법에는 Ad-hoc 명령 작성과 Playbook 작성이 존재한다.
- Ad-hoc 명령을 사용하는 예시는 아래와 같다.
 
- 2-3.에 사용 형태가 기술되어 있지만, ansible [Target Server Node/Group명] –m [모듈명] –a [Ad-hoc 명령(사용하는 모듈에 따라 형태가 달라짐)] 형식으로 사용한다.
- 위 실행 결과는 xip43 그룹에 shell 모듈을 사용하여 df –h 명령을 적용시킨 결과이다.
- 기본 사용법 절에서는 Ad-hoc 명령어 관련 내용만 언급하고, yaml 작성법부터 Playbook 활용법및 사용 예시를 제시한다.

2-4.1 Module
- 모듈이란 프로그램을 구성하는 구성 요소로, 관련된 데이터와 함수를 하나로 묶은 단위이다.
- Ansible에는 command, shell, copy, systemd, service, yum 등 다양한 환경관리 자동화를 위한 모듈들이 존재한다.
- 가장 보편적으로 사용하는 모듈은 shell 모듈이며 파일 리스트 조회, 디스크 용량 및 파티션 목록 조회, 유저 패스워드 변경 등의 기본 shell 명령 동작에 사용한다.
- 그 외 iptables 방화벽 정책 구성, 파일 송수신, 블록 재구동 같은 시스템에 큰 영향을 미칠 수 있는 작업들은 shell 모듈이 아닌 systemd, iptables 등의 모듈을 사용하여 멱등성을 보장한다.
- 멱등성의 개념이 인지는 3-1.의 참고 사항에서 설명한다.
2-4.2 Shell module ad-hoc Sample usecase
- 우선 ls –l(파일 리스트 조회) 명령의 실행 결과는 아래와 같다.
 
- 할당된 파티션 정보 조회(fdisk –l 명령)도 가능하며 사용 결과는 아래와 같다.
 
- 지정된 user의 password를 변경할 수도 있으며 아래와 같은 예시로 사용한다.
 
- 그 외 추가적인 sample usecase는 추가 제작되는 Example PPT를 참조하는 것을 권장한다.
2-5. yaml 파일 작성
- 2-4. 절에서 Ad-hoc 명령 작성 외에도 Playbook을 이용해 Ansible Task를 실행하는 방식이 존재한다고 언급하였다.
- 하나의 Playbook 파일에 여러 Task를 지정할 수 있어서 한 번의 실행으로 여러 Task를 일괄 처리할 수 있다는 장점을 가지고 있어 많이 활용된다.
- Syntax Error에 대한 자체 디버깅을 제공하지 않으며, 문법 검사를 따로 제공해주는 홈페이지 등을 활용하여야 한다.

2-5.1 yaml 문법
- Playbook의 파일 형식은 yaml이고 해당 내용에 정의되는 Tasks를 지정된 hosts에 일괄 실행할 수 있다.
- name에 작업 이름, hosts에 해당 Playbook을 실행할 호스트 이름, tasks에 실행할 작업을 명시한다.
- 아래는 OS Command 실행을 위한 Playbook 이다.
 
- 위와 같이 작성했을 때, {{ command }} 와 같이 작성하면 command는 ansible-playbook 명령어 실행 시 –e 옵션을 통해서 값을 할당 받아야 하는 변수가 된다.
- 실행 예시는 2-6. 절의 Sample Usecase에서 설명한다.
2-6. ansible-playbook 명령
- Playbook으로 작성한 작업을 실행하고 싶다면 Controller Server에서 아래 명령을 실행한다.
- ansible-playbook [playbook_name.(yml/yaml)]
- 값을 받아야 하는 변수가 있을 경우는 아래 문법을 따른다. 
- ansible-playbook [playbook_name.(yml/yaml)] –e “변수명=변수값”
- 이때 roles를 지정해 한 번의 명령으로 여러 Playbook을 동시에 실행할 수 있는데, 이는 추후에 내용을 추가할 예정이다.

2-6.1 ansible-playbook Sample usecase
- 2-5. 에서 예시로 제시한 OS Command Sample Playbook을 실행하였다.
 
- 2-5.1 에서도 언급했듯이 –e 옵션을 준 다음, command 변수에 ls –l 값을 할당하여 ls -l이라는 명령어를 전달하여 실행한 결과를 return 받은 것이다.
- yaml 파일에서 사전에 입력 받을 변수를 선언하지 않았다면, -e 옵션을 부여하지 않고 그대로 사용하면 된다.
- 여기서 주의할 점은 Playbook으로 실행하면서 변수값에 특수문자(또는 이스케이프 문자) 또는 파이프라인을 사용할 경우 홑따옴표로 명령어 전체를 묶어줘야 한다.
- 홑따옴표로 명령어를 묶어서 실행했을 경우
 
- 홑따옴표로 묶어서 실행하지 않았을 경우
 
- 위 케이스처럼 홑따옴표로 파이프라인 명령어를 포함한 전체를 묶어주지 않으면 Controller Server가 전혀 다른 결과를 Return 받는 현상이 발생한다.
2-7. 계정 패스워드 변경
- 계정 패스워드 변경 작업은 Ad-hoc 방식 또는 Playbook 방식을 사용할 수 있다.
- 아래 예시는 ad-hoc 방식으로 user 모듈을 사용하여 root계정의 패스워드를 변경하 것이다.
 
- 유저 이름, 패스워드 상시 변경 여부, 패스워드, hash 방식 등을 지정해야 명령어가 동작한다.
- 계정 패스워드를 변경할 수 있는 Playbook은 아래와 같이 작성한다.
 
- 실행은 아래와 2-6.1의 sample usecase와 유사하게 ansible-playbook 명령으로 실행한다.
 
2-8. Debug Message
- 위에서 보았던 실행 결과의 문제점은 어느 서버에서 어떤 파일을 전송 하였고 어떤 명령을 실행했는지 구체적인 것을 알 수 없다.
- Playbook에서 debug 모듈을 사용하면 메시지를 표기할 수 있고, 표기 방식은 두 가지가 있다.

2-8.1 register 모듈을 불러와 debug 모듈에 변수로 불러오는 방식
- 하나의 task 단위에 register 모듈 사용하면 해당 task의 실행 과정을 확인할 수 있다.
 
- 위는 db_backup yaml 파일의 일부를 발췌한 것이며, 수정 사항으로는 task에 register 모듈을 사용하여 결과 변수를 등록하고, debug 모듈에서 이를 사용하는 내용을 추가한 것이다.
- 실행 결과는 다음과 같이 모든 객체들의 값을 출력한다.
 
2-8.2 debug 모듈의 msg 모듈을 사용하여 직접 메시지를 지정하는 방식
- task에 register 모듈을 사용하는 방식이 아닌, debug 모듈 아래에 msg를 직접 작성하는 방식이다.
 
- 실행 결과는 다음과 같이 msg에 지정된 메시지 내용만 출력한다.
 
- 가독성을 높여 사용할 수 있으나, 매번 세세히 정의해야 하는 번거로움이 있다.
2-8.3 stdout_lines를 활용한 깔끔한 디버깅 메시지 작성
- Playbook 실행 시 register 모듈을 사용하면 debug 메시지를 확인할 수 있다.
- 그 중 stdout_lines 객체를 불러오면 명령어 실행 결과를 디버깅 메시지로 확인할 수 있다.
- Playbook은 아래와 같이 register 모듈의 값을 저장하는 변수에 stdout_lines 객체를 덧붙여 호출하면 된다.
 
- stdout_lines 객체는 아래와 같이 명령어 실행 결과를 리턴 한다.
 
- shell 모듈의 register는 stdout_lines를 지원하지만, systemd등의 모듈에서는 stdout_lines 객체를 지원하지 않는다.
- systemd 모듈의 경우 아래와 같이 changed, failed, name, state, status 객체를 지원한다.
 
2-9. db_backup
- sftp 사용하여 백업 스크립트를 Target Server에 업로드 하고 Target Server에서 스크립트 실행 후Controller Server에 DB백업한 파일을 업로드 하는 방식으로 동작한다.
- 스크립트 및 Playbook은 아래와 같이 구현한다.
 

 
- 해당 Playbook을 실행하면 다음과 같이 정상적으로 결과가 잘 나오는 것을 볼 수 있다.
 
- Create a script directory if it does not exist로 정의된 Task는 디렉터리가 존재하지 않는다면 디렉터리를 생성하며 Changed를 Return하고 존재할 경우 OK를 Return하며 다음 Task로 전환한다.
- 정상 백업이 완료되면 Controller Server에 다음과 같이 백업이 올라와 있는 것을 확인할 수 있다.
 
2-9. patch upload
- 패치를 업로드 하는 방식은 db_backup 스크립트의 방식과는 다르게 Target Server가 Controller Server의 특정 디렉터리에서 패치 파일을 get으로 받아오는 방식이다.
- 스크립트는 아래와 같이 구현한다.
 
- 주의사항은 패치파일 이름의 날짜가 스크립트 실행 시점의 날짜와 일치해야 한다는 점이다.
- Controller Server에는 patch 파일을 수동으로 업로드 해야 한다.(FileZila 사용)
- yaml 실행 전에 반드시 Controller Server의 /SI/PKG/patch 디렉터리에 패치파일이 존재하는지 확인해줘야 한다.
- yaml은 아래와 같이 작성한다.
 
- 실행하면 다음과 같이 출력되어야 정상 동작한 것이다.
 
- 타겟 서버 중 하나를 확인하여 patch파일이 잘 올라 갔는지 확인한다.
 

2-11. 블록 재구동
- 데몬 관리를 위해서는 OS 버전별로 어떤 System Daemon을 사용하고 있는지 이해해야 한다.
- Redhat 계열 리눅스인 CentOS의 경우 CentOS 7을 기점으로 System Daemon이 inittab에서 systemd로 교체되면서 데몬 관리 명령어도 달라진다.
- CentOS 6.X 이하의 OS들은 inittab 기반으로 service 명령을, CentOS 7.x 이상의 OS는 systemd 기반으로 systemctl 명령을 사용하여 블록을 관리한다.
- systemd 모듈을 활용한 블록 재구동 Playbook은 아래 예시와 같이 작성한다.
 
- systemd 모듈 활용한 블록 재구동 예시는 아래와 같다.
 
- service 모듈을 활용한 블록 재구동 Playbook은 아래 예시와 같이 작성한다.
 

- service 모듈 활용한 블록 재구동 예시는 아래와 같다.
 

2-12. Packet Capture
- 간혹 서비스 운용 중에 이슈가 발생하여 패킷 내용을 확인할 필요가 있을 때 사용한다.
- 패킷은 호스트에서 네트워크 및 서비스가 주고받는 메시지의 일종이다.
- Playbook은 아래 예시와 같이 작성한다.
 
- 동작은 아래와 같이 패킷을 Target Server에서 캡쳐 및 파일로 저장하고 해당 파일을 Controller 서버로 저장한다.
 
- 주의 할 것은 노란색 박스의 동작에서 Ctrl + C로 pause 모듈을 중지 시킨 이후에
반드시 C를 입력해야 나머지 tcpdump 프로세스 종료 및 파일 저장이 정상적으로 이루어 진다.

- A를 입력할 경우 아래와 같이 Playbook 실행 자체가 중지된다.
 
- 또한 아래와 같이 Target Server의 프로세스가 살아 있는 채로 계속 동작한다.
 
3. 기타 참고 사항
3-1. 참고사항
3-1.1 멱등성
- 전산학 또는 수학에서 사용하는 용어이다.
- 연산을 여러 번 적용 하더라도 결과가 달라지지 않는 성질 또는 연산을 
여러 번 반복 하더라도 한 번만 수행된 것과 같은 성질을 의미한다.
- 메소드가 여러 번 실행되어도 결과는 같으므로 안전하게 사용할 수 있다는 의미이다.
- 변화가 발생하지 않는 동일한 Task를 실행하였을 때 
결과가 항상 Changed로 Return 되는 것은 Ansible 사용 취지에 적합하지 않다.
- 디렉터리 생성으로 테스트했을 때 적절한 모듈을 사용한 ad-hoc 명령은 다음과 같이 동작한다.
 
- 위와 같이 첫 번째 Task에서는 Changed가 true, 두번째 task에서는 false를 Return 한다.
- 그러나 file 모듈이 아닌 shell 모듈을 사용하여 디렉터리를 생성할 경우 아래 경고가 발생하며 계속 changed를 return한다.
 
- System 관련 Task를 실행할 때 해당 부분이 Ansible을 상용에서 운영함에 있어서 큰 문제가 될 수 있기 때문에 반드시 보장되어야 한다.

3-2. 특이사항
- 특정 제품군을 대상으로 서비스 블록을 재구동 하면 iptables가 자동으로 재구성되는 현상 때문에 Task의 Result 값을 Return 받지 못하는 현상이 발생(Access-Control-List 항목에 Controller Server IP를 ACCEPT 정책으로 추가하여 해결)
- Playbook으로 계정 Password 변경 시 아래와 같이 사용하면 CentOS7.x 아래 버전의 OS에서는 Password가 인식이 제대로 안 되는 현상이 있음
 
