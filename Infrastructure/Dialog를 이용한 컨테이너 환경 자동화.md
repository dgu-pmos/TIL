



# Dialog를 이용한 컨테이너 환경 자동화

## dockermgmt

```bash
#!/bin/bash

# temp file 생성 
temp3=$(mktemp -t test3.XXXXXX)

# 도커 패키지 설치 함수
function installdocker {
	# docker 패키지 정보 질의 후, temp.txt에 리다이렉션
	rpm -qa | grep docker > temp.txt
	check=$(echo $?)
	# 이미 설치 되었다면 infobox 출력 후, 메인 메뉴로 이동
	if [ $check -eq 0 ]
	then
		dialog --infobox "이미 설치되어 있습니다" 5 30 ; sleep 1
	# 설치되지 않았다면, yum command를 이용해 필요한 패키지들 설치 및 실행
	else
		yum -y install yum-utils device-mapper-persistent-data lvm2 
		yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
		yum -y install docker-ce
		systemctl start docker
		dialog --infobox "도커-CE 가 설치되었습니다" 3 30 ; sleep 5
	fi
}

# 컨테이너 상태 출력 함수
function statusctn {
	# 컨테이너 리스트 출력 후, statusctn.txt에 리다이렉션
	sudo docker container ls -a > statusctn.txt
	# textbox를 이용해 statusctn.txt 출력
	dialog --textbox statusctn.txt 20 150
}

# 이미지 상태 출력 함수
function statusimg {
	# 이미지 리스트 출력 후, statusimg.txt에 리다이렉션
	sudo docker image ls > statusimg.txt
	# textbox를 이용해 statusimg.txt 출력
	dialog --textbox statusimg.txt 20 150
}

# 컨테이너 설치에 필요한 파라미터 수집 함수
function installctn {
	# yesno를 이용해 설치 여부 확인
	dialog --title "도커 컨테이너 설치 안내" --yesno "본 실습내용은 정~~말 간단한 내용의 컨테이너 설치를 알아보기 위한 내용입니다.  dialog 실습용으로만 확인하세요! 도커 컨테이너 설치를 진행하겠습니다" 10 40
	# 설치 여부 결과 installcheck에 저장
	installcheck=$(echo $?)
	# 만약 사용자가 설치를 원한다면(code 0), 필요한 파라미터 수집 시작 
	if [ $installcheck -eq 0 ]
	then
		# inputbox를 이용해 컨테이너 이름 ctnname에 저장
		dialog --inputbox "컨테이너 이름:" 10 40 2>ctnname.txt
			ctnname="--name $(cat ctnname.txt)"
		# yesno를 이용해 tty 사용 여부 ctnit에 저장 
		dialog --yesno "설치된 컨테이너와의 연결을 위한 tty 를 생성하시겠습니까?" 10 40
			ctnit=$(echo $?)
			# ctnit가 yes 라면 "-it" 옵션 추가
			if [ $ctnit -eq 0 ]
			then
				ctnityes="-it"
			# ctnit가 no 라면 "-d" 옵션 추가
			else
				ctnityes="-d"
			fi
		# yesno를 이용해 포트포워드 사용 여부 portcheck에 저장
		dialog --yesno "호스트의 포트를 이용하여 외부서비스를 진행하시겠습니까?" 10 20
	   		portcheck=$(echo $?)
		 	# portcheck가 yes 라면 imputbox를 이용해 ctnportmap에 포트번호 저장
			if [ $portcheck -eq 0 ]
			then
				dialog --inputbox "호스트포트와 컨테이너포트(예, 8080:80) 형태로 입력하세요" 10 40 2>ctnportmap.txt
				ctnportmap="-p $(cat ctnportmap.txt)"
			fi
		# inputbox를 이용해 이미지명 ctnimg에 저장
		dialog --inputbox "사용하고자 하는 이미지명 작성:" 10 40 2>ctnimg.txt
			if [ -s ctnimg.txt ]
			then
				ctnimg=$(cat ctnimg.txt)
			# 만약 ctnimg.txt가 비어있다면 종료
			else
				dialog --infobox "이미지명을 작성하지 않아 종료됩니다" 5 30 ; sleep 1
				exit
			fi
	else
		exit
	
	fi
}

function installctn_finish {
	# 만약 파라미터 수집이 정상적을 종료하였다면 docker container run 실행
	if [ $? = 0 ]
	then
		dialog --infobox "설치를 진행합니다" 5 30 ; sleep 3
		docker container run $ctnityes $ctnname $ctnportmap $ctnimg
		dialog --infobox "설치가 마무리 되었습니다" 5 30 ; sleep 3
	else
		exit
	fi
}

# 네트워크 상태 출력 함수
function listnetwork {
	# 네트워크 리스트 출력 후, listnetwork.txt에 리다이렉션
	sudo docker network ls > listnetwork.txt
	# textbox를 이용해 listnetwork.txt 출력
	dialog --textbox listnetwork.txt 20 150
}

function removectn {
	# 이 배열은 입력 파라미터를 보관하기 위해 필요함
	menus=()
	# 컨테이너 리스트 출력 후, statusctn.txt에 리다이렉션
	sudo docker container ls -a > statusctn.txt
	# 만약 statusctn.txt에 컨테이너 내용이 하나라도 있다면, 필터링 과정을 거쳐 입력 파라미터 수집
	if [ -n "$(cat statusctn.txt | grep -v CONTAINER)" ]
	then
		# 필터링 과정
		# 1. grep을 이용해 테이블 헤더 제거
		# 2. gawk를 이용해 첫 필드와 마지막 필드 선택
		# 3. listctn에 리다이렉션
		cat statusctn.txt | grep -v CONTAINER | gawk '{print $NF " " $1}' > listctn.txt
		# 입력 파라미터 수집
		# while read 명령문을 이용해 listctn.txt를 라인 별로 읽고 공백을 기준으로 파싱해 line1, line2에 담음  
		while read line1 line2
		do
			# 배열에 요소 삽입
        	menus+=("$line1")
        	menus+=("$line2")
			menus+=("off")
		done < listctn.txt
		# radiolist를 이용해 컨테이너 이름 선택
		dialog --radiolist "Remove container" 20 60 10 "${menus[@]}" 2>remove.txt
		# 만약 cancel 버튼을 입력했다면 메세지 출력과 함께 메인 메뉴로 이동
		if [ $? -eq 1 ]
		then
			dialog --infobox "cancel remove container" 5 30 ; sleep 3
		# 만약 컨테이너 이름이 선택 되었다면 삭제 실행
		elif [ -s remove.txt ]
		then
			sudo docker container rm -f $(cat remove.txt)
			dialog --infobox "remove container finished" 5 30 ; sleep 3  
		# 만약 컨테이너 이름을 선택하지 않았다면 메세지 출력과 함께 메인 메뉴로 이동
		else
			dialog --infobox "please select container" 5 30 ; sleep 3
		fi
	# 만약 컨테이너 리스트가 비어 있다면 메세지 출력과 함께 메인 메뉴로 이동
	else
		dialog --infobox "container list is empty!" 5 30 ; sleep 3
	fi
}

function removeimg {
	# 이 배열은 입력 파라미터를 보관하기 위해 필요함
	menus=()
	# 이미지 리스트 출력 후, statusimg.txt에 리다이렉션
	sudo docker image ls -a > statusimg.txt
	# 만약 statusimg.txt에 이미지 내용이 하나라도 있다면, 필터링 과정을 거쳐 입력 파라미터 수집
	if [ -n "$(cat statusimg.txt | grep -v REPOSITORY)" ]
	then
		# 필터링 과정
		# 1. grep을 이용해 테이블 헤더 제거
		# 2. gawk를 이용해 첫 필드와 마지막 필드 선택
		# 3. listimg에 리다이렉션
		cat statusimg.txt | grep -v REPOSITORY | gawk '{print $1 " " $2}' > listimg.txt
		# 입력 파라미터 수집
		# while read 명령문을 이용해 listimg.txt를 라인 별로 읽고 공백을 기준으로 파싱해 line1, line2에 담음  
		while read line1 line2
		do
			# 배열에 요소 삽입
        	menus+=("$line1")
        	menus+=("$line2")
			menus+=("off")
		done < listimg.txt
		# radiolist를 이용해 이미지 이름 선택
		dialog --radiolist "Remove Image" 20 60 10 "${menus[@]}" 2>remove.txt
		# 만약 cancel 버튼을 입력했다면 메세지 출력과 함께 메인 메뉴로 이동
		if [ $? -eq 1 ]
		then
			dialog --infobox "cancel remove image" 5 30 ; sleep 3
		# 만약 이미지 이름이 선택 되었다면 삭제 실행
		elif [ -n "$(cat remove.txt)" ]
		then
			sudo docker image rm -f $(cat remove.txt)
			sudo docker image prune -y
			dialog --infobox "remove image finished" 5 30 ; sleep 3  
		# 만약 이미지 이름을 선택하지 않았다면 메세지 출력과 함께 메인 메뉴로 이동
		else
			dialog --infobox "please select image" 5 30 ; sleep 3
		fi
	# 만약 이미지 리스트가 비어 있다면 메세지 출력과 함께 메인 메뉴로 이동
	else
		dialog --infobox "statusimg is empty!" 5 30 ; sleep 3
	fi
}

# 무한루프로 메뉴 실행
while [ 1 ]
do
# menu를 이용해 메인 메뉴 출력
dialog --cancel-label "Exit" --menu "Docker Management" 20 30 10 1 "도커CE 설치" 2 "도커 컨테이너 상태  확인" 3 "도커 이미지 상태 확인" 4 "컨테이너 설치" 5 "Check network list" 6 "Remove container" 7 "Remove image" 2>$temp3
# 선택 결과를 selection에 저장하고, case 문으로 실행할 함수 선택
selection=$(cat $temp3)
case $selection in
1)
	installdocker ;;
2)
	statusctn ;;
3)
	statusimg ;;
4)
	installctn
	installctn_finish ;;
5)
	listnetwork ;;
6)
	removectn ;;
7)
	removeimg ;;
*)
	dialog --infobox "잠시후 종료됩니다" 5 30 ; sleep 3
	exit ;;
esac 

done

rm -f $temp1 2 > /dev/null
rm -f $temp2 2 > /dev/null
rm -f $temp3 2 > /dev/null
cp -f *.txt ..
rm -f *.txt
```

## 결과

<img src="https://i.ibb.co/fDtc9LR/image-20200625161422911.png" alt="image-20200625161422911" style="zoom:67%;" />

<img src="https://i.ibb.co/TqDQygx/image-20200625161437464.png" alt="image-20200625161437464" style="zoom:67%;" />

<img src="https://i.ibb.co/kDZqQy2/image-20200625161511245.png" alt="image-20200625161511245" style="zoom:80%;" />