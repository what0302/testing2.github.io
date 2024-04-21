# 웹쉘 헌팅 *(Hunting Webshells)* 

##### 링크: [Hunting Webshells: Linux and Windows Commands][webshelllink]
[webshelllink]: https://library.mosse-institute.com/articles/2022/06/hunting-webshells-linux-and-windows-commands/hunting-webshells-linux-and-windows-commands.html "Go webshell"
##### 작성자: 강하늘 사원
##### 작성일자: 2024년 4월 7일


## 웹쉘(Webshell)의 정의
- 공격자가 서버에 원격으로 접근하기 위해 사용하는 악성 스크립트임.
  + 예를 들면 웹쉘을 이용해 임의의 명령을 실행하고, 파일을 업로드 및 다운로드 하거나 리버스 쉘(Reverse Shell) 연결을 설정하는 데에도 사용할 수 있음.
 
## 소개
- 로그 분석은 웹쉘\(Apache, IIS 등)을 탐색하는 데 사용되는 기술 중 하나임.
- 웹쉘 로그 분석에 많은 시간을 투자하지 않고, Log Parser Studio 도구를 사용하여 이전 섹션에서 언급한 도구와 기본 내장 OS 명령 및/또는 스크립팅 언어를 포함한 IIS 웹 로그를 분석하여 웹 서버의 디렉토리에서 숨겨진 웹쉘을 찾을 것임.

</br>

- 실제 도구로 넘어가기 전에 웹쉘을 찾기 위한 몇가지 기본적인 Linux 웹 서버 명령을 살펴보겠음.
- _알아둬야 할 것은 이 방법이 웹쉘을 찾기 위한 유일한 방법은 아님._
- _웹쉘을 찾는 방법들 중 몇가지만 다룰 것임._



</br>

- 다음 절에서는 특정 리눅스 웹 서버에서 4개의 웹 셸을 찾는 방법과 도구를 살펴봄.
  * 두 개의 파일은 난독화되지 않고 다른 두 개의 파일은 난독화됨.
  * 또한, 두 개의 파일에는 .php 파일 확장자가 있고 두 개의 웹쉘에는 .txt 파일 확장자가 있음.
- 이는 완료 및 성공적인 탐지를 위해 수행됨.

![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/947a1829-1c63-490a-a051-397cf133e05c)

![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/8d2bcf47-41c6-4519-ac31-053a5dfca952)

![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/270f4d93-97f0-4ae9-9f75-774818276718)

![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/3dfd07bd-418f-4a9a-937b-56911603abe1)

- 이전 사진에서 다음 웹쉘이 서버에 있는 것을 볼 수 있음.
    * locus7s.php (/var/www/html/)
    * unknown.txt (/var/www/html/js/)
    * ss8.txt (/var/www/html/images/)
    * unknown2.php (/var/www/html/css/)

## 리눅스 명령
- 다음 명령은 지난 24시간 이내에 웹 서버에 업로드된 새 파일을 찾을 것임.
- 사용자 환경의 웹 서버에서 매일 웹쉘 검사를 수행하는 경우 유용함.

  `find . -type f -name '*.php' -mtime -1`

  
  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/2feeb249-4d50-49f7-8818-5aa60030e658)

- 앞의 명령을 사용하여 Linux는 24시간 이내의 모든 PHP 파일('*.php')을 현재 디렉터리(.) 내에서 검색하도록 지시함.
- \-mtime 파일 속성은 파일이 마지막으로 수정한 시간과 날짜를 저장함(속성 자체가 아닌 내용만 저장).

</br>

- 이 정보는 다음 명령을 사용하여 볼 수 있음:

  `ls -l`


- txt 파일이나 해당 문제에 대한 모든 파일에 대해 동일한 작업을 수행할 수 있음:

  `find . -type f -name '*.txt' -mtime -1`


  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/faf28069-3d26-4a8c-8501-1bfacc5f3edd)


- 앞의 명령은 파일 내에서 아무것도 검사하지 않음.
- 검색된 위치에서 파일 내용의 마지막 수정 시간만 확인함.

</br>

- 파일 내에서 의심스러운 코드를 찾아내는 몇 가지 리눅스 명령어를 살펴보겠음.



- 파일 내에서 의심스러운 코드를 찾기 위해 xargs 및 grep을 사용하여 기존 Linux 명령을 수정함.

</br>

- 간략한 설명: </br>
    xargs - 표준 입력을 기반으로 명령줄을 생성하고 실행함. </br>
    grep - 패턴과 일치하는 행을 표시함.



- 이 명령은 PHP eval() 함수의 인스턴스를 검색함.
- 다음 명령을 실행하여 PHP 웹쉘을 검색할 것임.

  `find . -type f -name '*.php' | xargs grep -l "eval *("`


  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/cf212861-640d-4f98-9d72-715e3c62b505)


- php 파일이 아닌 txt 파일을 검색하면 이전과 동일한 결과를 얻을 수 있음:
  
  `find . -type f -name '*.txt' | xargs grep -l "eval *("`


  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/c41f4479-005d-4468-a145-2578c7df0b38)
- 이 Linux 명령을 한 가지 더 변경한 다음 다른 PHP 함수를 찾아보겠음.
- 이번에는 base64_decode() 함수를 찾아보겠음.


- 웹 서버에 있을 수 있는 PHP 파일 내에서 이 함수를 검색하는 동안 결과를 살펴보겠음:

  `find . -type f -name '*.php' | xargs grep -l "base64_decode*("`


  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/a4b9d85a-ba51-4e2e-a5f9-03bcedca047c)


- 이전 명령의 출력에 따르면 이번에 탐지된 파일은 단 한 개뿐이며 암호화되지 않은 파일임을 알 수 있음.

- PHP 기반 웹 셸과 트로이 목마는 PHP 프로그래밍 언어의 eval() 및 base64_decode() 함수를 사용하는 경우가 많음.
- 이 예제에서는 str_rot13() 및 gzinflate와 같은 다른 PHP 함수가 활용됨.

- 악성 PHP 스크립트에 자주 있는 기능을 포함하여 모든 PHP 기능을 검색하는 한 줄의 코드를 실행할 수 있음.

  `find . -type f -name '*.php' | xargs egrep -i "(mail|fsockopen|pfsockopen|exec|system|passthru|eval|base64_decode) *\("`

- 앞서 본 Linux 명령은 *egrep*을 활용하여 서버에 저장된 파일 내에서 다양한 키워드를 검색함.
- 히트를 기록하는 모든 항목은 발생 즉시 터미널에 표시됨.

- 각 기능에 대한 간략한 설명: </br>
   + **mail()** - 스팸을 보내는 데 사용할 수 있음. 
   + **fsockopen()** - 원격 요청을 보내기 위해 네트워크 연결을 여는 데 사용할 수 있음. 
   + **pfsockopen()** - **fsockopen()** 과 동일함. 
   + **exec()** - 명령 실행을 위한 것임. 
   + **system()** - **exec()** 와 함께 사용할 수 있음. 
   + **passthru()** - **exec()** 와 함께 사용할 수 있음.


  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/7ba61a60-2ebe-4fc8-9c8a-fb888f98d932)

- PHP 파일만 찾고 있기 때문에 파일 이름에 PHP 확장자가 포함된 두 웹쉘을 모두 찾을 수 있었음.
- TXT로 저장된 웹셸에 대해 동일한 검색을 실행하면 결과가 동일할까?

    `find . -type f -name '*.txt' | xargs egrep -i
"(mail|fsockopen|pfsockopen|exec|system|passthru|eval|
base64_decode) *\("`

    ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/5ae54d9c-1e84-4580-8db1-a39c0889f9f8)

- Linux 명령은 대부분 예상대로 수행됨.
- 이제 완전히 숨겨져 있고 난독화된 PHP 백도어를 보여줄 것임.
- 코드 내에는 쉽게 눈에 띄는 PHP 함수가 없음.

이 파일에 대한 일부 정보: </br>
다음 디렉토리는 이 파일이 저장된 위치임: /var/www/html/v1/fonts/

### 알 수 없는 PHP 백도어(Unknown PHP Backdoor)
![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/f0600599-1f37-450a-bea6-2daa1f8b1402)
- 단일 라이너인 최종 Linux 명령을 실행할 때 얻은 출력은 다음과 같음.

    `sudo find . -type f -name '*.txt' | xargs egrep -i
"(mail|fsockopen|pfsockopen|exec|system|passthru|eval|
base64_decode) *\("`

![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/63059044-0137-4204-b7fa-96c47eb08b2f)
- 예상했던 대로 명령은 아무것도 찾지 못함.

## Windows 명령 
- PowerShell의 *Get-ChildItem* 및 *Select-String cmdlet*을 사용하면 Windows의 grep 명령과 유사한 작업을 수행할 수 있으므로 파일 시스템에 저장된 파일 내에서 키워드를 검색할 수 있음.
- 이 예에서는 PowerShell의 grep 명령을 한 줄로 구현하는 방법을 살펴보기만 하면 됨:

  `get-childitem -recurse -include "*.php" | select-string  "(mail|fsockopen|pfsockopen|exec\b|system\b|passthru|eval\b|base64_decode)" | % {"$($_.filename):$($_.line)"} | out-gridview`

  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/9929727d-f1f4-417f-8f77-24ffd8e76631)

- PowerShell에서 이 명령을 간단히 설명하면 다음과 같음:
  + **Get-ChildItem**은 CMD의 dir 명령과 Linux의 ls 명령에 해당하는 명령.
  + **-Recurse**는 루트 디렉터리 아래에 있는 모든 디렉터리를 순회함.
  + **-Include** 스위치를 사용하여 파일 확장자를 지정하면 PowerShell이 해당 특정 파일 형식만 검사하도록 지시함.
  + **|** (파이프), 그 사용법은 Linux 및 CMD와 유사함.
  + **Select-String** 명령은 Linux의 **grep** 명령과 동일함.
  + **\b** 문자는 단어 경계를 나타내며, 이는 PowerShell에게 해당 단어에서 정확히 처리를 중지하도록 지시함.
  + 퍼센트 기호 **"%{$($_.filename:$($ .line)"}** 의 출력은 파일 이름과 단어가 발견된 줄을 제공함.
  + **Out-Gridview** 명령을 사용하면 출력이 콘솔 형식이 아닌 그리드 보기로 표시됨.

- 이전 PowerShell 명령의 결과가 아래에 표시되어 있음.

  ![image](https://github.com/ICTIS-Cert-System-Project/ICTIS-Cert-System/assets/164521627/1edb6cba-a544-4242-bc16-01aec4c2ff1a)

- 앞의 프로그램은 실제 웹 로그를 가리키도록 변경될 수 있으며 로그 내의 키워드를 검색하여 악의적인 증거를 식별할 수 있음.
- Linux의 grep 명령은 정확히 같은 방식으로 작동함.
