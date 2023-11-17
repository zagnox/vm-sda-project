# VM-SDA-PROJECT
:pirate_flag:	CTF Game SDA Virtual Machine Step-by-Step

## :mag:	RECONNAISSANCE
### :black_medium_square:	Skanojme IP dhe mbledhim informacion rreth targetit
Lokalizojme IP e vm-sda ne rrjetin tone te brendshem duke skanuar te gjithe range 0/24

Perdorim komanden `nmap -n 192.168.10.0/24`

Nga rezultati i skanimit me nmap kuptojme qe 192.168.10.6 eshte IP e vm-sda. Kjo IP ka hapur portat:

**21/tcp ftp**

**22/tcp ssh**

**80/tcp http**

![Screenshot_2023-11-16_15_53_25](https://github.com/zagnox/vm-sda-project/assets/144890045/e4080f10-fede-40c5-9088-588f9df18de6)

Nje skanim me i thelle me `nmap -sCV 192.168.10.6` na jep nje rezultat me te mire mbi sistemet dhe versionet qe ndodhen ne keto porta

![Screenshot_2023-11-16_16_12_45](https://github.com/zagnox/vm-sda-project/assets/144890045/ea9115ee-cdd1-4415-a1a0-87847e0a423d)

### :black_medium_square:	Tentojme te logohemi me FTP

Nje gabim i shpeshte qe ndodh gjate konfigurimit te ftp eshte qe lejohet logimi si anonymous.
Logimi deshtoi qe do te thote qe ftp kerkon nje username dhe password valid.

![Screenshot_2023-11-16_16_01_33](https://github.com/zagnox/vm-sda-project/assets/144890045/183fab2d-8c8f-40f1-9ae4-d106c120167f)


### :black_medium_square:	Kontrollojme porten 80 https
Perpara se te tentojme brute forcing i hedhim nje sy portit 80 http ku po hostohet nje website.
Pas njefare kohe duke kerkuar zbulova nje mesazh te enkoduar me base64 ne source codin e faqes

![Screenshot_2023-11-16_16_07_23](https://github.com/zagnox/vm-sda-project/assets/144890045/782a74d4-1c60-4919-94c6-714c42520e5e)

Perdorim komanden per ta dekoduar nga base64:

`echo "RW51bWVyYXRlIG1lIHdpdGggZGlyZWN0b3J5LWxpc3QtbG93ZXJjYXNlLTIuMy1tZWRpdW0udHh0" | base64 -d`

![Screenshot_2023-11-16_16_09_02](https://github.com/zagnox/vm-sda-project/assets/144890045/99c09dd8-e226-4548-9a9e-595724de6b4a)

Mesazhi na udhezon qe te kryejme nje directory brute forcing me wordlisten e permendur.

Perdorim gobuster per te kryer nje directory brute force me komanden:

`gobuster dir -u http://192.168.10.6 -w /opt/useful/SecLists/Web-Content/directory-list-lowercase-2.3-medium.txt`

Rezultati na kthen keto 4 directory te fshehura:

***/images***

***/css***

***/js***

***/requests***

***/libs***

![Screenshot_2023-11-16_16_21_35](https://github.com/zagnox/vm-sda-project/assets/144890045/d0bc2230-bc70-4fc2-9874-c04ef9165864)


Ne direktorin _/requests_ shohim nje file interesant me emrin _urgent.txt_. Aty ndodhet nje tekst drejtuar developersave per te ndryshuar
passwordin e nje useri. Username i tij eshte nje planet dhe passwordi ne listen rockyou-10.

![Screenshot_2023-11-16_16_27_00](https://github.com/zagnox/vm-sda-project/assets/144890045/fd043e06-5228-4a04-a5a3-d45722b01560)


## :skull_and_crossbones: FOOTHOLD/EXPLOIT 	

Me keto te dhena ne mund te ndertojme nje liste me usernamet e planeteve dhe te perodrim rockyou-10 per te kryer nje sulm brute forcing me hydran.

Pasi kemi krijuar listen me 10 planetet e sistemit diellor perdorim komanden:

`hydra -L planet.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou-10.txt ssh://192.168.10.6 -t 4`

![Screenshot_2023-11-16_16_59_20](https://github.com/zagnox/vm-sda-project/assets/144890045/69bc020e-99db-4d0c-9e61-3e9533be10e0)


### :smiling_imp:	USER PWNED!!!! :smiling_imp:	

![giphy](https://github.com/zagnox/vm-sda-project/assets/144890045/99addf51-eb2b-4296-b629-4a1ba6ca13f2)


:technologist: username: **uranus**

:closed_lock_with_key:	password: **butterfly**

## :syringe:	PRIVILEGE ESCALATION

Tashme mund te logohemi me: ssh **uranus**@192.168.10.6

:pirate_flag:	FLAG #1 `cat user.txt`

Hapi i radhes eshte te eskalojme privilegjet per te kapur root. Ne kerkim e siper gjeta dicka interesante ne _.bash_history_

`ls -la` na jep nje liste me filet e fshehura ne home direcotry e uranus.

`cat .bash_history` duke investiguar komandat e superuserit gjeta nje tjeter hint te koduar me base64

![Screenshot_2023-11-16_17_54_00_bash](https://github.com/zagnox/vm-sda-project/assets/144890045/453869ce-bb5a-4f53-a96f-f7e391b62e67)

![Screenshot_2023-11-16_17_54_00](https://github.com/zagnox/vm-sda-project/assets/144890045/6f90e00a-532d-47f5-ac28-9406e0bbe597)

`echo "cm9vdCBwYXNzd29yZCBpbiBhIDMtZGlnaXQgY29kZQ==" | base64 -d`
na jep si mesazh ***root password in a 3-digit code***

Kjo do te thote qe superuseri root ka nje password 3-shifror. Duke llogaritur se jane vetem 1000 kombinime te mundshme me numrat 3-shifror eshte e lehte te bejme brute forcing.

Shkruajme nje script ne python i cili gjeneron nje text file me numrat nga _000, 001, 002... deri ne 999_

![Screenshot_2023-11-16_18_14_24](https://github.com/zagnox/vm-sda-project/assets/144890045/7a1c8f1f-1587-4efd-b455-0b37498677fd)

Pasi gjenerojme listen e passwordeve perodrim kete [shell script](https://github.com/carlospolop/su-bruteforce/blob/master/suBF.sh) per root brute forcing.

Komanda per brute forcing:
`./suBF.sh -u root -w numrat.txt -t 10 -s 0.003`

![Screenshot_2023-11-16_18_28_00](https://github.com/zagnox/vm-sda-project/assets/144890045/49420135-dcf7-4be8-b603-b561e8313e44)

## GAME OVER

![Screenshot_2023-11-16_18_28_33](https://github.com/zagnox/vm-sda-project/assets/144890045/1b81e723-0fe2-41a1-b7cf-6425dfa5c8d6)

**su root 666**
`cat /root/root.txt`
