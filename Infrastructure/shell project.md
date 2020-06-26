# Shell_Quiz

```bash
#!/bin/bash

# 응답을 수집해서 $ANSWER에 저장하는 함수
function get_answer {
# 기존에 저장된 값을 unset으로 제거
unset ANSWER
# 사용자가 응답하지 않은 횟수 카운트
ASK_COUNT=0
#  ANSWER 변수에 값이 채워질 때 까지 반복
while [ -z "$ANSWER" ] 
do
	# 카운트 추가 (4일 경우에 프로그램 강제 종료)
	ASK_COUNT=$[ $ASK_COUNT + 1 ]
	case $ASK_COUNT in
	# 2번째 질문인 경우
	2)
		echo
		echo "질문에 답변해주세요 "
		echo ;;
	# 3번째 질문인 경우	
	3)
		echo
		echo "다시한번.. 질문에 답변 부탁드립니다"
		echo ;;
	# 4번째 질문인 경우 프로그램 종료
	4)
		echo
		echo "질문에 대한 답변이 없어"
		echo "프로그램을 종료합니다"
		echo
		exit ;;
	esac
	echo
	# 만약 LINE2에 값이 채워져 있다면
	# 출력할 LINE이 2개이므로 전부 출력
	if [ -n "$LINE2" ]
	then 
		echo $LINE1
		echo -e $LINE2" \c"
	# 그렇지 않으면, LINE1만 출력
	else 
		echo -e $LINE1" \c"
	fi
	# 10초 동안 응답 대기 (응답이 들어오면 ANSWER에 저장)
	read -t 10 ANSWER
done
# 출력한 LINE1,2를 제거
unset LINE1
unset LINE2
}
 
# 응답이 YES인지 아닌지 판별하는 함수 
function process_answer { 
# LINE1,2 출력
echo $LINE1
echo $LINE2 
# 정규표현식을 이용해 'yes'인지 확인
case $ANSWER in
	$(echo $ANSWER | gawk '/[Yy]{1}[Ee]?/')|$(echo $ANSWER | gawk '/[Yy]{1}[Ee]{1}[Ss]?/')) ;;
	# 'yes'가 아니라면 EXIT_LINE 출력하고 프로그램 강제 종료
	*)
 		echo
 		echo $EXIT_LINE1
 		echo $EXIT_LINE2
 		echo
 		exit ;;
	esac
# 프로그램 종료 메세지 EXIT_LINE1,2 제거
unset EXIT_LINE1
unset EXIT_LINE2
} 

# 메인 프로그램 시작!!
echo "Step #1 - 삭제할 계정 선택하기"
echo 
# get_answer를 실행하기 전에 LINE1,2에 값 채움
LINE1="삭제를 원하는 계정의"
LINE2="사용자 명을 작성해 주세요:"
# 사용자명을 얻기 위한 get_answer 함수 실행
get_answer
# 함수로부터 얻은 값을 USER_ACCOUNT에 저장
USER_ACCOUNT=$ANSWER
LINE1="삭제하고자 하는 계정이"
LINE2="$USER_ACCOUNT 가 맞는지요?  [y/n]"
# 삭제 여부를 위한 값을 얻기 위한 get_answer 함수 실행
get_answer
EXIT_LINE1="삭제하고자 하는 계정이"
EXIT_LINE2="$USER_ACCOUNT 가 아니어서 프로그램을 종료합니다"
# 값이 yes인지 판별하기 위한 함수 실행
process_answer
# USER_ACCOUNT에 해당하는 정보를 /etc/passwd로부터 추출하고
# USER_ACCOUNT_RECORD에 저장
USER_ACCOUNT_RECORD=$(cat /etc/passwd | grep -w $USER_ACCOUNT)
# 비정상 종료(사용자 정보가 없다면) 프로그램 종료
if [ $? -eq 1 ] 
then
	echo
	echo "$USER_ACCOUNT 를 찾을 수 없습니다"
	echo "프로그램을 종료합니다"
	echo
	exit
fi
echo
echo "아래의 내용을 확인해주세요 :"
echo $USER_ACCOUNT_RECORD
# 추출된 값이 맞는지 사용자로부터 확인
LINE1="계정정보가 맞는가요?  [y/n]"
get_answer
EXIT_LINE1="삭제하고자 하는 계정이"
EXIT_LINE2="$USER_ACCOUNT 가 아니어서 프로그램을 종료합니다"
process_answer
echo
echo "Step #2 - 해당 사용자가 실행한 프로세스 중지"
echo
# USER_ACCOUNT 소유의 프로세스를 확인하기 위해 ps 명령을 실행하고 
# 종료 상태 코드를 이용해 프로세스 존재 여부 확인
# 출력 값은 필요없으므로 /dev/null로 리다이렉션
ps -u $USER_ACCOUNT >/dev/null 
case $? in
	# 만약 프로세스가 없다면 해당 메세지 출력
	1) 
		echo "해당 계정이 실행중인 프로세스가 없습니다"
		echo ;;	
	# 프로세스가 있다면 프로세스들 중지 여부를 묻고 중지
	0) 
		echo "$USER_ACCOUNT 는 아래의 프로세스를 실행중입니다 : "
		echo
		# 존재하는 프로세스 목록 출력
		ps -u $USER_ACCOUNT --no-heading
 		LINE1="중지할까요? [y/n]"
 		get_answer
 		case $ANSWER in
 				# 만약 사용자가 중지한다고 했다면 ps, xargs 명령을 이용해 프로세스 중지
				$(echo $ANSWER | gawk '/[Yy]{1}[Ee]?/')|$(echo $ANSWER | gawk '/[Yy]{1}[Ee]{1}[Ss]?/'))
 				echo
				echo "해당 프로세스를 중지중입니다 "
				# 헤더를 지우고 USER_ACCOUNT의 프로세스 목록을 출력
				COMMAND_1="ps -u $USER_ACCOUNT --no-heading"
				# xargs는 출력된 결과를 인자값으로 이용해 다른 커멘드에서 활용할 수 있음
				# 앞 명령어의 결과를 다음 명령어의 입력으로 넘기는 파이프와 함께 사용
				# sudo kill -9 {PID}를 병렬적으로 수행해줌
				COMMAND_3="xargs -d \\n /usr/bin/sudo /bin/kill -9"
				$COMMAND_1 | gawk '{print $1}' | $COMMAND_3
				echo
				echo "Process(es) killed." ;;
			*) 
				echo
 				echo "해당 프로세스를 중지하지 않습니다 "
 				echo ;;
 		esac
	esac
echo
echo "Step #3 - 사용자 계정 소유의 파일 검색"
echo
echo " $USER_ACCOUNT 소유의 파일목록을 작성중입니다 "
echo
echo "해당 파일에 대한 백업과 패키징을 추천합니다 "
echo
echo "잠시만 기다려 주세요 "
REPORT_DATE=$(date +%y%m%d)
# 현재 날짜와 삭제할 사용자명을 이용해 파일 이름 생성
REPORT_FILE=$USER_ACCOUNT"_Files_"$REPORT_DATE".rpt"
# find 명령으로 삭제할 사용자의 파일을 REPORT_FILE에 저장하고, stderr 값은 버림
find / -user $USER_ACCOUNT > $REPORT_FILE 2>/dev/null
echo
echo "목록작성이 마무리되었습니다 "
echo "목록명: $REPORT_FILE"
echo "목록파일의 위치 : $(pwd)"
echo
echo
echo "Step #4 - 계정삭제"
echo
LINE1="시스템에서  $USER_ACCOUNT 계정 삭제를 진행합니다? [y/n]"
get_answer
EXIT_LINE1="계정삭제를 취소하여, "
EXIT_LINE2="스크립트를 종료합니다"
process_answer
# process_answer 결과 값이 'yes'라면 userdel 명령 
userdel -r $USER_ACCOUNT 
echo
echo "사용자계정, $USER_ACCOUNT 는 삭제되었습니다"
echo
exit
```

