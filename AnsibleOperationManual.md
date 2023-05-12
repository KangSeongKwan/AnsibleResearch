# 1. Ansible 활용 방안
- Ansible을 활용하면 다수의 서버에 대해 일정한 작업을 자동화하고, 특정 패치에 관한 배포를 자동화 할 수 있는 등 다양한 활용 사례가 있다.
- 삼성 SDS에서 작성한 Ansible 활용 사례를 보면, AWS와 같이 보안이 취약해질 수 있는 Cloud 환경에서 외부 위협을 방어하는데 사용할 수 있다고 한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/f87bdeb9-d082-41ee-b9ee-554dc25c9b32)
- 아래 글은 SDS 기술문서에서 제안한 Cloud 환경에서 Ansible을 이용해서 보안을 강화할 수 있는 가장 간단한 작업의 사례이다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/85ed5081-9943-4f48-bf2a-748eac839bcc)

# 2. Ansible 운용
### 2-1. 운용 전 알아야 할 것
- 이 사례에서는 관리 대상의 서버에 존재하는 Database 파일(sql파일)을 Controller 서버에 백업하는 방법에 대해서 다루었다.
- 우선, 다루기 전에 Controller Server에 Ansible을 설치한 후 어떤 것이 존재하는지 알아야 한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/c44d0457-5522-44d7-9d43-f9f104137a06)
- 위와 같이 ansible 설정파일이 들어 있는 /etc/ansible에 진입하면 다음과 같이 roles, ansible.cfg, hosts와 그 외의 것이 들어있다.
- 이 중 설치 직후 확인 시 기본으로 탑재되어 있는 것은 roles 디렉터리, ansible.cfg 설정 파일, hosts 설정 파일이다.
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/185d97d1-3414-41b1-a946-7b464913de0c)
- Ansible 설치 매뉴얼 문서의 절차대로 설치가 완료되었다면, 일종의 환경 설정을 진행해야 한다. 
- Controller Server, Managed Node에 ssh-keygen 명령어를 입력하여 둘 다 ssh key를 생성하고 ssh-copy-id 명령어로 생성한 키를 등록해야 한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/9b270dec-0b65-4265-a09c-a398e6d92465)
- key 생성 시 위 화면처럼 출력되지 않으면 ssh-copy-id를 통해 key를 자동으로 등록할 수 없다.(보통 OS버전이 낮으면 많이 발생한다.)
- ssh key 등록에 성공하면 PW를 물어보지 않고 ssh로 로그인 할 수 있음을 확인할 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/54ba818b-903b-4769-b1ef-4e2b9892c7a5)
- 키 생성 후 자동 등록이 되지 않는다면 대상 서버 authorized_key 파일에 직접 등록해야 한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/445c93ce-f39c-4d7c-bc97-7184f55bfeb2)
- ssh key를 직접 등록하는 방법은 별도의 외부 문서를 참고하기 바란다.
- 여기까지 완료해야 본격적으로 Ansible을 운용할 수 있다.

### 2-2. Ansible Inventory 설정
- 이번 절부터 진행하는 작업은 모두 Controller Server에서 하는 작업에 해당한다.
- Inventory는 관리 대상의 목록을 의미한다고 했다. 이 목록은 각각의 개별 노드가 될 수 있고, 여러 노드를 비슷한 성질에 따라 묶은 그룹이 될 수가 있다.
- 이를 사용하려면 그 전에 /etc/hosts 파일에 alias 등을 등록해야 IP주소가 아닌 alias를 지정해서 편하게 유지보수 할 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/8cc93820-d4cd-44fe-9116-93123042a821)
- network servicer restart 후 /etc/ansible/hosts 또는 별도의 ini파일에 관리대상 노드를 등록한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/a282cd37-6a2e-4bc2-afc3-21d59e9203ea)
- 별도의 Inventory 파일을 사용하고 싶다면 다음과 같이 /etc/ansible/ansible.cfg를 수정해서 Default Inventory를 등록하면 명령어의 길이를 줄일 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/a0f5f060-1baf-48fa-b806-256418045fd9)

### 2-3. yaml 파일 작성
- 2-1.의 인벤토리 설정 이후 yaml 파일을 작성해야 하는데(yml이라고도 함) 이를 Playbook을 작성한다고 한다.
- Playbook을 작성하고 해당 파일을 실행하면 hosts 키워드에 지정된 실행 대상에 대해서 tasks에 지정된 작업을 실행하게 된다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/8b91556a-af3b-4ad2-b66d-1c7131747320)
- yaml 작성은 간격 준수도 엄격히 해야 하고 디버깅이 매우 까다롭다.
- name에 작업 이름, hosts에 해당 Playbook을 실행할 호스트 이름, tasks에 실행할 작업을 명시한다.
- 이때 ansible에서 제공해주는 모듈들을 사용할 수 있는데, 내용이 방대하므로 필요한 모듈이 있을 때 마다 구글링 또는 공식 문서를 참고하는 것을 권장한다.

### 2-4. ansible 실행
- Playbook에 저장된 작업을 실행하고 싶다면 Controller Server에서 anisble-playbook [playbook 이름]을 명시해준 후 명령어를 실행하면 된다.
- 이때 roles를 지정해 여러 Playbook을 동시에 실행할 수 있는데, 이는 추후에 내용을 추가할 수 있도록 하겠음.
- 실행 예시는 아래와 같다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/e4421b04-e954-4441-8b45-33d184ec0283)
- 이는 정상적인 실행이 아닌데, n2 관리 대상의 Python 버전이 2.6 미만이기 때문에 발생한다.
- 정상 실행 내용은 다음과 같다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/bcad8480-b792-47e8-8f6e-8a414c21df08)
- 또한 대상 서버에 들어있던 db백업 파일이 Controller Server에 정상적으로 들어가 있는 것을 확인할 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/e9256f85-4d4d-4802-9ae9-dc4277ee20a2)
- 실행 시 주의 해야 할 것이 있는데 anisble-playbook 명령 실행 시 /etc/ansible/ansible.cfg 설정 파일 내부에 inventory 값을 지정하지 않으면 default Inventory로 /etc/ansible/hosts 파일에 저장되어 있는 내용을 참조하게 된다.
- 이를 피하려면 설정 파일 값을 수정하거나 –i 옵션을 사용하여 별도로 생성한 인벤토리 파일을 지정해주어야 한다. 아래는 인벤토리를 지정해서 실행하는 예시이다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/e30c3d79-521b-4720-99c3-ebc26447077c)
- 참고로 아래 내용은 dbBackup을 실습하면서 사용했던 스크립트이다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/6a521fe8-805f-4ef3-93de-683dbc7f21a2)

### 2-5. db_backup
- sftp를 이용하는 방식이고 2-3.의 마지막 항목에 있는 스크립트를 실행하는 방식이다.
- 구동 절차 : sftp 사용하여 백업 스크립트를 타겟 서버에 업로드 > 타겟 서버에서 스크립트 실행 > 컨트롤러 서버에 db 백업한 파일들 업로드
- 2-3.에서 작성한 yaml 내용을 조금 더 보강하여 다음과 같이 작성한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/d634b640-277b-4477-9f40-7ce080fa1b0a)
- 주의해야 할 점은 remote_path는 타겟 서버에서 만들어져 있는 디렉터리여야 한다.
- remote_path 디렉터리가 생성되지 않았다면 다음과 같은 에러가 발생한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/a5da4777-ad37-46b7-ada4-8d2101905031)
- 만약 타겟 서버에 /root/script 디렉터리가 없다면, ansible Ad-hoc 명령을 실행하여 디렉터리를 일괄적으로 생성하는 방법을 사용하면 된다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/6b48d3a1-4a58-489d-a885-b0d9dcbb4b19)
- 그 다음 실행하면 다음과 같이 정상적으로 결과가 잘 나오는 것을 볼 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/a06129df-01e0-445a-a367-df859f862637)
- 정상 백업이 완료되면 Controller Server에 다음과 같이 백업이 올라와 있는 것을 확인할 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/bfb5c7e4-ba79-4a55-9cca-50e490d6dbee)

### 2-6. patch upload
- 패치를 업로드 하는 방식은 dbBackup 스크립트의 방식과는 다르게 Controller Server의 특정 디렉터리에서 패치 파일을 get으로 받아오는 방식이다.
- 스크립트는 아래와 같이 구현한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/75953eba-c0b9-4b04-ad66-1e8f0b21ef61)
- 주의사항은 db_backup시와 마찬가지로 remote_path 디렉터리가 target 서버에 전부 생성되어 있어야 한다는 점이다.
- db_backup시와 동일(디렉터리 이름만 다르게)한 Ad-hoc 명령을 사용하여 생성한다.
- Controller Server에는 patch 파일을 수동으로 업로드 하면 된다.(FileZila 사용)
- yaml 실행 전, patch 압축파일이 존재하는지 부터 확인한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/ddd33c5f-535d-4886-9bce-f987c15f5439)
- yaml은 아래와 같이 작성해준다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/dee837d1-ede2-43cf-8b86-bfd4f52901bc)
- 실행하면 다음과 같이 출력되어야 정상 동작한 것이다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/4b9f46e1-5387-498d-9f1f-cac064594a7a)
- 타겟 서버 중 하나를 확인하여 patch파일이 잘 올라 갔는지 확인한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/74d3d5fb-2554-4c70-b08d-3658e075a0db)
 
### 2-7. Debug Message
- 위에서 보았던 실행 결과의 문제점은 어느 서버에서 어떤 파일을 전송 하였고 어떤 명령을 실행했는지 구체적인 것을 알 수 없다.
- Playbook에서 debug 모듈을 사용하면 메시지를 표기할 수 있고, 표기 방식은 두 가지가 있다.

#### 2-7.1 register 모듈을 불러와 debug 모듈에 변수로 불러오는 방식
- 하나의 task 단위에 register 모듈 사용하면 해당 task의 실행 과정을 확인할 수 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/951329b0-315c-4179-9e86-030befe4db7e)
- 위는 db_backup yaml 파일의 일부를 발췌한 것이며, 수정 사항으로는 task에 register 모듈을 사용하여 결과 변수를 등록하고, debug 모듈에서 이를 사용하는 내용을 추가한 것이다.
- 실행 결과는 다음과 같이 상세한 과정이 나온다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/80125f2f-8113-4e8d-bfe3-67e932be0d75)
- 연구개발 과정에서의 논의 결과 두 방식 중에서 이 방식을 채택하는 것으로 결론이 나왔다.
#### 2-7.2 debug 모듈의 msg 모듈을 사용하여 직접 메시지를 지정하는 방식
- task에 register 모듈을 사용하는 방식이 아닌, debug 모듈 아래에 msg를 직접 작성하는 방식이다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/6a7d033b-e7c4-4ba3-983f-b94858d383e4)
- 실행 결과는 다음과 같이 지정된 메시지 내용만 출력한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/70f6b1b2-addf-4a66-b55b-00f4f5d062e5)
- 가독성을 높여서 사용할 수 있으나, 매번 세세히 정의해야 하는 번거로움이 있다.

### 2-8. OS 명령어
- Linux OS 명령어 실행 결과를 모든 관리 대상 서버에서 일제히 확인하고 싶을 때가 있다. 이 때Ansible을 사용하면 아주 편리하다.
- yaml 파일을 다음과 같이 작성한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/e18170c7-3521-468e-82ed-d5cf99fcf868)
- 사용 방법은 아래 사진과 같이 –e 옵션을 사용하여 command 변수에 값을 부여한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/ee11ebe5-7c2f-4812-88cb-ec022908652f)
- 이 때 명령어에 옵션 부여 또는 파이프를 사용하고자 하면 홑따옴표를 붙여 문자열로 인식 시켜야 값이 제대로 전달된다.(ex command=’df –h’, command=’ls –ltr’ 등등)
- 위와 같이 yaml을 작성하는 것이 아닌 Ad-hoc 명령을 사용하는 방법은 아래와 같다.
- 명령 쉘에서 ansible 명령을 입력하고, -m 옵션으로 shell 모듈을, -a 모듈로 shell 명령에 전달할 명령어를 입력한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/da54189c-f539-41c8-b1d2-7c47741c23b9)
- 이를 활용하여 블록을 내릴 수도, 살릴 수도 있다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/474beac4-a35a-4584-b9a7-fd46d116405f)
- 현재 에러가 발생한 타겟 서버들은 ipageon service가 아니라 xener service를 stop 시켜야 하기 때문에 발생하는 에러이기 때문에 무시해도 상관없다.
- OS 명령어의 경우 상황마다 사용하고 싶은 명령어가 유동적으로 변화하는 특징을 지니고 있다.
- 따라서 Playbook을 작성하여 실행할 명령을 매번 지정하는 것 보다는 Ad-Hoc 방식을 채택하는 것이 더 나은 결과물을 도출하는데 도움이 된다.

### 2-9. 계정 패스워드 변경
- 패스워드를 변경할 때에는 Ad-hoc 실행 보다는 Playbook 작성을 권장한다.
- Ad-hoc 명령 방식으로 실행할 경우 아래 실행과 같이 지정해야 하는 옵션이 많다.
- 유저 이름, 패스워드 상시 변경 여부, 패스워드 등을 지정해야 명령어가 동작한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/49443770-881c-4260-9655-619bf34d9d87)
- 위 사진의 결과의 경우 암호화 해시를 반드시 넣을 것을 권장하는 경고 문구인데, 무시하고 그냥 실행하면 패스워드 변경 후 로그인이 안 된다.(서버에 물리적 접근으로 로그인하는 것은 가능)
- 계정 패스워드를 변경할 수 있는 Playbook은 아래와 같이 작성한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/06a7b4d9-0e26-48d7-80cf-985946db3989)

# 3. 기타 참고 사항
### 3-1. 타겟팅 서버 그룹핑 원칙 및 설정 방법
- 타겟팅 서버를 그룹핑 설정 방법은 Inventory 파일 생성 방법에 포함되어 있는 내용에 있는 인벤토리 파일 하나를 가져와 설명한다.  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/09a2e47f-de5f-401c-a745-28fb24d4f5cf)
- 그룹핑 설정은 위와 같이 [그룹이름] 아래에 포함하고 싶은 대상 서버의 이름 또는 IP 주소를 적어주면 [xipServers]라는 그룹 아래에 n1, n2, n3라는 이름의 관리 대상 서버가 포함된다.
- 이 때 이름으로 지정하려면 무조건 /etc/hosts 파일에 해당 이름에 매칭되는 IP주소 정보가 있어야 한다.
- 그룹을 지정하여 실행하려면 playbook에는 hosts 변수에 그룹명을 작성하고, Ad-hoc 명령에는 그룹명을 작성하면 하면 된다.
- 플레이북 그룹 지정  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/866fd83d-2db2-4607-9fe0-17a2ebfca508)
- Ad-hoc 명령어 그룹 지정  
![image](https://github.com/KangSeongKwan/AnsibleResearch/assets/99636945/eb077dc5-6a61-48b1-a214-3df0f55348ad)
- /etc/hosts 파일에서 관리 대상 IP의 이름을 지정할 때 어떤 서버인지 구분하기 쉽게 이름을 명확하게 지어주는 것도 관리 용이성을 추구할 수 있는 방법 중 하나이다.
- 투입된 프로젝트 단위로 Inventory 파일을 분리해 작성하는 것 또한 좋은 방법이 될 수 있다.

