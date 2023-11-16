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

Nje skanim me i thelle me `nmap -sCV 192.168.10.6` na jep nje rezultat me te mire mbi sistemet dhe versionet qe ndodhen ne keto porta

### :black_medium_square:	Tentojme te logohemi me FTP

Nje gabim i shpeshte qe ndodh gjate konfigurimit te ftp eshte qe lejohet logimi si anonymous.
Logimi deshtoi qe do te thote qe ftp kerkon nje username dhe password valid.

### :black_medium_square:	Kontrollojme porten 80 https
Perpara se te tentojme brute forcing i hedhim nje sy portit 80 http ku po hostohet nje website
Pas njefare kohe kerkimi zbulova nje mesazh te enkoduar me base64 ne source codin e faqes

Perdorim komanden per ta dekoduar nga base64:

`echo "RW51bWVyYXRlIG1lIHdpdGggZGlyZWN0b3J5LWxpc3QtbG93ZXJjYXNlLTIuMy1tZWRpdW0udHh0" | base64 -d`

**"Enumerate me with directory-list-lowercase-2.3-medium.txt"**

Mesazhi na udhezon qe te kryejme nje directory brute forcing me wordlisten e permendur.

Perdorim gobuster per te kryer nje directory brute force me komanden:

`gobuster dir -u http://192.168.10.6 -w /opt/useful/SecLists/Web-Content/directory-list-lowercase-2.3-medium.txt`

Rezultati na kthen keto 4 directory te fshehura:

***/images***

***/css***

***/js***

***/requests***

***/libs***

Ne direktorin _/requests_ shohim nje file interesant me emrin _urgent.txt_. Aty ndodhet nje tekst drejtuar developersave per te ndryshuar
passwordin e nje useri. Username i tij eshte nje planet dhe passwordi ne listen rockyou-10.

## :skull_and_crossbones: FOOTHOLD/EXPLOIT 	

Me keto te dhena ne mund te ndertojme nje list me usernamet e planeteve dhe te perodrim rockyou-10 per te kryer nje sulm brute forcing me hydran.

Pasi kemi krijuar listen me 10 planetet e sistemit diellor perdorim komanden:

`hydra -L planet.txt -P /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou-10.txt ssh://192.168.10.6 -t 4`

### **SUKSES! USER PWNED!!!!**

:technologist: username: **uranus**

:closed_lock_with_key:	password: **butterfly**

## :syringe:	PRIVILEGE ESCALATION

Tashme mund te logohemi me: ssh **uranus**@192.168.10.6

:pirate_flag:	FLAG #1 `cat user.txt`

Hapi i radhes eshte te eskalojme privilegjet per te kapur root. Ne kerkim e siper gjeta dicka interesante ne _.bash_history_

`ls -la` na jep nje liste me filet e fshehura ne home direcotry e uranus.

`cat .bash_history` duke investiguar komandat e superuserit gjeta nje tjeter hint te koduar me base64

`echo "cm9vdCBwYXNzd29yZCBpbiBhIDMtZGlnaXQgY29kZQ==" | base64 -d`
na jep si mesazh ***root password in a 3-digit code***

Kjo do te thote qe superuseri root ka nje password 3-shifror. Duke llogaritur se jane vetem 1000 kombinime te mundshme me numrat 3-shifror eshte e lehte te bejme brute forcing.

Shkruajme nje script ne python i cili gjeneron nje text file me numrat nga _000, 001, 002... deri ne 999_

Pasi gjenerojme listen e passwordeve perodrim kete [shell script](https://github.com/carlospolop/su-bruteforce/blob/master/suBF.sh) per root brute forcing.

Komanda per brute forcing:
`./suBF.sh -u root -w numrat.txt -t 10 -s 0.003`

## GAME OVER

**su root 666**
`cat /root/root.txt`
