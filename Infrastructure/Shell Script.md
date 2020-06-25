# Shell Script

## 이론

### 변수

- 환경변수
  - set 명령을 사용하면 출력되는 요소들이 환경변수
  - 사용 시, 환경변수 이름 앞에 '$' 를 붙임
- 사용자 변수
  - 스크립트 안에 자체 변수를 설정하고 사용
  - 변수명은 최대 20 글자,숫자,언더바로 이루어짐(대소문자 구분)
  - 사용 시, 환경변수 이름 앞에 '$' 를 붙임
- 명령 치환
  - 명령의 출력으로부터 정보를 추출하고 이를 변수에 할당할 수 있음
  - 역따옴표 문자(``), $() 형식으로 실행
- 계산
  - expr, $[] 형식으로 실행
- bc
  - 부동소수점 연산을 위한 방법
  - $(echo "options; expression" | bc) 형식
    - 예시 : var=$(echo "scale=4; $var1 / $var2" | bc)
      - var1과 var2를 나누고 소수점 4자리 까지 표현

### 명령문

- 숫자 비교
  - n1 -eq n2 : 같은 지 검사
  - n1 -ge n2 : 크거나 같은 지 검사
  - n1 -gt n2 : 큰 지 검사
  - n1 -le n2 : 작거나 같은 지 검사
  - n1 -lt n2 : 작은 지 검사
  - n1 -ne n2 : 같지 않은 지 검사
- 문자열 비교
  - str1 = str2 : 같은 지 검사
  - str1 != str2 : 같지 않은 지 검사
  - str1 < str2 : 작은 지 검사
  - str1 > str2 : 큰 지 검사
  - -n str1 : 길이가 0보다 큰 지 검사
  - -z str1 : 길이가 0인지 검사
- 파일 비교
  - -d file : 파일이 존재하고 디렉토리인지 검사
  - -e file : 파일이 존재하는지 검사
  - -f file : 파일이 존재하고 파일인지 검사

- if-then

  - 문법

    ```
    if [ condition ]
    then
    	commands
    elif [ condition ]
    then
    	commands
    else
    	commands
    fi
    ```

- case

  - 문법

    ```
    case var in
    pattern1 | pattern2)
    	commands1;;
    
    pattern3)
    	commands2;;
    
    *)
    	default commands;;
    esac
    ```

- for

  - 문법

    ```
    for var in list
    do
    	commands
    done
    ```

  - 목록의 복잡한 값 읽기

    - 개별 데이터 값 사이에 빈 칸이 있다면 겹따옴표로 감싸기
    - 홑따옴표를 사용하는 값을 겹따옴표로 감싸기

  - IFS

    - set에 존재하며, 필드 구분자를 정의
    - 스크립트 내부에서 새로 정의하면 해당 IFS를 사용
      - 예시 : IFS=$'\n'

- while

  - 문법

    ```
    while [ condition ]
    do
    	commands
    done
    ```

- break

  - 어떤 유형의 루프든 빠져나올 수 있음
  - 하나의 루프만 빠져 나옴
    - N 개의 루프 빠져 나오려면 "break N"

- continue

  - 루프의 시작으로 되돌림
  - 하나의 루프만 되돌릴 수 있음
    - N 개의 루프 되돌리려면 "continue N"

- select

  - 사용자의 입력을 받아서 처리

  - 입력 값은 메뉴 목록의 숫자 중 하나

  - 사용자의 입력을 받기 위해서 PS3 프롬프트 사용

  - 문법

    ```
    PS3="~~~~"
    
    select var in list
    
    do
    	command
    done
    ```


### awk

- 데이터를 조작하고 리포트를 생성하기 위해 사용하는 언어

- 리눅스에서 사용하는 awk는 GNU 버전의 gawk로 심볼릭 링크되어 있음

- 파일 또는 파이프를 통해 입력 라인을 얻어와 $0 라는 내부 변수에 라인을 입력

  - 각 라인을 레코드라고 부르고, newline에 의해 구분

- 라인은 공백을 기준으로 각각의 필드로 구분

  - 필드는 $1부터 시작

- 화면에 필드를 출력할 때 print 함수 사용

- 기본 형식

  | 옵션        | 설명                                       |
  | ----------- | ------------------------------------------ |
  | -F          | 필드 경계를 식별할 구분자                  |
  | -f          | 프로그램이 읽어 들일 명령 파일 지정        |
  | -var =value | 프로그램에서 사용할 변수의 기본값 정의     |
  | -mf         | 데이터 필드에서 처리할 필드의 최대 수 지정 |
  | -mr         | 데이터 파일의 최대 레코드 크기 지정        |


- 예시

  ```bash
    $ awk '{print NR, $0}' awkfile
    > 
    1 홍 길동	3324	5/11/96	50354
    2 임 꺽정	5246	15/9/66	287650
    3 이 성계	7654	6/20/58	60000
    4 정 약용	8683	9/40/48	365000
    // NR은 NR 변수를 의미하며, 각 레코드의 번호 의미
    // $0은 라인을 의미
    $ awk '{print $1}' awkfile
    > 
    홍
    임
    이
    정
    // 첫 번째 필드만 출력
  ```

#### begin

- 보고서의 머리말 부분을 만들 때 사용
- gawk가 데이터를 읽기 전에 실행할 프로그램을 지정

#### end

- 보고서의 꼬리말 부분을 만들 때 사용
- gawk가 데이터를 읽은 후에 실행할 프로그램을 지정

#### 정규표현식

- 텍스트를 걸러내는 데 사용하는 정의 패턴 템플릿

- 표{}

  | 기호     | 설명                                     |
  | -------- | ---------------------------------------- |
  | ^        | 행의 시작                                |
  | $        | 행의 끝                                  |
  | *        | 해당 문자가 없거나 있거나 상관없음       |
  | ?        | 해당 문자가 없거나 한번 나타나는 것      |
  | +        | 최소 한 개는 있어야 함                   |
  | [문자들] | 문자들에 포함되는 하나의 문자와 부합     |
  | {m,n}    | 최소 m번, 최대 n번 나타남                |
  | ()       | 괄호 안에 있는 것들은 하나의 문자로 취급 |
  | \|       | 논리 OR 식                               |

### 매개변수

- 쉡 스크립트에 데이터를 전달하는 가장 기본적인 방법

- 스크립트를 실행할 때 커맨드라인에 데이터를 추가할 수 있음

- 매개변수가 필요한 프로그램에 입력하지 않으면 "Syntax error" 발생

- 사용 예제

  ```bash
  vim test.sh
  '''
  #!/bin/bash
  
  total=$[ $1 * $2 ]
  echo "the first parameter is $1"
  echo "the second parameter is $2"
  echo "the total value is $total"
  '''
  
  ./test.sh 2 5
  the first parameter is 2
  the second parameter is 5
  the total value is 7
  ```

## 실습

### /usr/bin에 넣어서 프로그램 실행

#### /bin

- /bin
  - 아주 기본적인 명령어 위치
- /sbin
  - root 권한이 필요한 명령어 위치
- /usr/bin
  - /bin에 있는 명령을 제외한 명령어 위치
- /usr/sbin
  - /sbicat n에 있는 명령을 제외한 명령어 위치

#### bashrc

- /etc/profile
  - 환경변수와 bash가 수행될 때 실행되는 프로그램을 제어하는 전역적인 시스템 설정 관련 파일
- /etc/bashrc
  - 모든 사용자에게 영향을 끼치는 환경 설정 파일
- ~/.bash_profile
  - 환경변수와 bash가 수행될 때 실행되는 프로그램을 제어하는 지역적인 시스템 설정 관련 파일
  - /etc/profile 과는 달리 bash 실행하는 사용자에게만 영향을 줌
  - /etc/profile 수행된 후에 곧바로 수행
- ~/.bashrc
  - alias와 bash가 수행될 때 실행되는 함수를 제어하는 지역적인 시스템 설정 관련 파일
  - 해당 사용자 한정

1. shell script 작성

   ```bash
   vim select.sh
   '''
   #!/bin/bash
   
   PS3="Select the menu you want to excute: "
   
   select excute in "Install Docker" "Create Container" "Delete Container" "Finish Program"
   
   do
   	case $excute in
   	"Install Docker")
   		echo "Now, we are install Docker"
   		echo "installing...." ;;
   	"Create Container")
   		echo "Now, we are creating container"
   		echo "creating...." ;;
   	"Delete Container")
   		echo "will be deleted container you selected"
   		echo "deleting...." ;;
   	"Finish Program")
   		exit ;;
   	*)
   		echo "please select the number between 1 - 4" ;;
   	esac
   done
   '''
   ```

2. /usr/bin에 붙이기

   ```
   cp select.sh /usr/bin/dockerman
   ```

3. 결과

   ![image-20200623165737899](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200623165737899.png)

### text 파일에 존재하는 이메일을 이용해 root 계정 판독

1. text 파일 생성

   ```
   | chulsoo | test1@test.com | 01011111111 |
   | youngsoo | test2@test.com | 01022222222 |
   | soonhee | test3@test.com | 01033333333 |
   | root | root@test.com | 01044444444 |
   ```

2. 스크립트 작성

   ```bash
   #!/bin/bash
   
   cat test.txt | while read line
   
   do
           filter=$(echo "$line" | gawk '{print $4}')
           if [ $filter = "root@test.com" ]
           then
                   echo "hello root"
           else
                   echo "let's play in friday night"
           fi
   done
   ```

3. 결과

   ![image-20200623170416751](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200623170416751.png)





---



### 간단한 텍스트 메뉴 만들기

```bash
vim menu1
'''
#!/bin/bash
# simple script menu

function diskspace {
	clear
	df -k
}

function whoseon {
	clear
	who
}

function memusage {
	clear
	cat /proc/meminfo
}

function menu {
	clear
	echo
	echo -e "\t\t\tSys Admin Menu\n"
	echo -e "\t1. Display disk space"
	echo -e "\t2. Display logged on users"
	echo -e "\t3. Display memory usage"
	echo -e "\t\tEnter options: "
	read -n 1 option
}

while [ 1 ]
do 
	menu
	case $option in
	0)
		break ;;
	1)
		diskspace ;;
	2)
		whoseon ;;
	3)
		memusage ;;
	*)
		clear
		echo "Sorry, wrong selection" ;;
	esac
	echo -en "\n\n\t\t\tHit any key to continue"
	read -n 1 line
done
clear
'''
```

결과

![image-20200624161018276](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200624161018276.png)

![image-20200624161031925](C:\Users\User\AppData\Roaming\Typora\typora-user-images\image-20200624161031925.png)



### 간단한 GUI 메뉴 만들기

1. dialog 패키지 설치

   ```
   yum -y install dialog
   ```

   - dialog 지원 위젯 표

     | 위젯      | 설명                                     | 위젯         | 설명                                      |
     | --------- | ---------------------------------------- | ------------ | ----------------------------------------- |
     | calendar  | 날짜 선택하는 달력 제공                  | passwordbox  | 입력받은 텍스트를 숨기는 단일 텍스트 상자 |
     | checklist | 각 항목을 켜거나 끌 수 있는 여러 항목    | passwordform | 숨겨진 텍스트 필드로 구성된 양식          |
     | form      | 입력을 위한 텍스트 필드로 구성된 양식    | menu         | 선택 목록                                 |
     | gauge     | 진행 비율을 나타냄                       | yesno        | Yes, No 버튼이 있는 간단한 메세지 제공    |
     | infobox   | 응답을 기다리지 않고 메세지              | timebox      | 시,분,초 선택 창                          |
     | inputbox  | 텍스트 입력을 위한 단일 텍스트 양식 상자 | msgbox       | 메세지 표시하고 OK 버튼 표시              |
     | inputmenu | 편집 메뉴                                | radiolist    | 하나의 아이템만 선택                      |

2. 

