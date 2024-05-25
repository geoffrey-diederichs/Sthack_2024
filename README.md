# Code Lyoko

```md
Auteur : ghozt

Reverse, medium

Jeremie a réussi à prendre le contrôle du code de XANA pendant un court instant et a réussi à y placer une porte dérobée. Il vous demande de récupérer la main sur XANA au plus vite !

nc 51.20.123.75 1337
```

[Cet executable](./8f612a248da7de80738a208bed1d4d19a27621adf12a2f0adcf55a05af9f06a0) est fournit. Tentons de le lancer :

```console
$ ./8f612a248da7de80738a208bed1d4d19a27621adf12a2f0adcf55a05af9f06a0
              C0d3_Ly0k0                
                                        
                  &&@&                  
                 %@&@@@                 
                 /@%%@(                 
                 /@  @(                 
              *&@@@@@@@@%,              
          @@&             .@@&          
       &@,       ./../.       %@/       
     /@,    .@@*        *@@     #@.     
    %@    ,@/              (@    .@,    
   ,@    /@     *@@&&@@      @,   (@    
   .*   %@#    #@(@@@@.@     %@@   /    
    (@&  ,@@    .@&#@@*@@     @@,  @@.   
     @@    @&                &@   .@&    
      @#    *@%            %@.    @@     
      /%@. .   *@@@&..&@@@,   . #@*/     
         /@@                 ,@@.        
            .@@@@*.&&&#.(@@@@            
           ,@ *   @@@@@%   ( @           
          .@ @.    &  %    (# @          
         .@.@,    .@  @.    ##/@         
        .@.@,      @  @      %@#@        
       ,*@,       @  @       %@&        
                  @%@@                  
                  @%@@                  
                  ,%&,                  
######@@@@@@@ X.A.N.A is watching you @@@~####']-[}
XANA Management Console 
> a
```

Il prend en entrée un charactère et s'interrompt, analysons le code source avec Ghidra.

## Analyse statique

```C
void main(void)
{
  int user_input;
  
  setvbuf_init();
  signal(14,handle_sigalrm);
  signal(8,handle_sigfpe);
  alarm(30);
  menu_art();
  menu_msg();
  print_msg("XANA Management Console \n",30);
  do {
     print_msg(&INPUT_SYMBOL,30);
     user_input = getchar();
     handle_input((int)(char)user_input);
  } while( true );
}
```

La fonction `main` commence par définir quelles fonctions s'occuperont des signaux 14 et 8. D'après [le man](https://man7.org/linux/man-pages/man7/signal.7.html) de `signal` les code 14 et 8 correpondent respectivement à `SIGALRM` et `SIGFPE`. Le premier nous intéresse peu, car la fonction `handle_sigalrm` ne fait qu'afficher un message avant de relancer le programme (à moins que la fonction `system` soit exploitable plus tard) :

```C
void handle_sigalrm(int signal)
{
  if (signal == 14) {
     print_msg("Too late... Back to the past !\n",0x1e);
     system("/home/ctf/chall");
  }
  return;
}
```

Le second est plus intéressant, car il affiche le contenu de la variable d'environnement `BCKDR` (probablement utile pour le chal) :

```C
void handle_sigfpe(int signal)
{
  char *bckdr;
  size_t length;
  int index;
  
  if (signal == 8) {
     bckdr = getenv("BCKDR");
     length = strlen(bckdr);
     for (index = 0; (ulong)(long)index < length; index = index + 1) {
       printf("%d ",(ulong)(byte)(bckdr[index] ^ 127));
     }
     putchar(10);
     exit(1);
  }
  return;
}
```

`SIGFPE` signalant une expection arithmétique, il faudra trouver dans le programme un moyen de réaliser une mauvaise opération arithmétique.

La fonction `main` affiche ensuite le menu, récupère une entrée d'un charactère et l'envoie à la fonction `handle_input` :

```C
void handle_input(char user_input)
{
  if (user_input == 's') {
     calculate_mob_army();
     exit(42);
  }
  if (user_input < 't') {
     if (user_input == 'h') {
       print_help_msg();
       return;
     }
     if (user_input == 'q') {
       print_msg("Exiting session ....",60);
       exit(1337);
     }
  }
  exit(1337);
}
```

Rien d'intéressant si l'utilisateur saisie `h` ou `q`, mais en saisissant `s` nous arrivons sur cette fonction :

```C
void calculate_mob_army(void)
{
  long in_FS_OFFSET;
  int user_input;
  undefined4 local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  print_msg("Sending mobs...\n",10);
  puts("How much mobs: \n");
  puts("> \n");
  __isoc99_scanf(&DECIMAL_INPUT,&user_input);
  local_14 = 1000;
  ARMY_STRENGTH = 1000 / user_input;
  printf("Total army strength : %d \n",(ulong)ARMY_STRENGTH,1000 % (long)user_input & 0xffffffff);
  print_msg("Reconfiguring mob army...\n ",60);
  puts("Done ! Attack launched !");
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
     __stack_chk_fail();
  }
  return;
}
```

Cette fonction réalise une opération arithmétique à partir de l'entrée de l'utilisatuer, et sauvegarde le résultat dans la variable globale `ARMY_STRENGTH`. Nous pouvons facilement obtenir un `SIGFPE` avec cette ligne de code :

```C
  ARMY_STRENGTH = 1000 / user_input;
```

Enfin, cette varibale d'environnement `BCKDR` semble intéressante, mais n'est pas réutilisé dans le code que nous avons trouvé. En cherchant des références à cette variable dans le code nous retrouvons cette fonction :

```C
void pas_envie_de_t_analyser(void)
{
  ulong __len;
  size_t sVar1;
  undefined *__src;
  code *pcVar2;
  undefined *puVar3;
  long in_FS_OFFSET;
  undefined auStack_108 [8];
  ulong local_100;
  char *local_f8;
  ulong local_f0;
  undefined8 local_e8;
  undefined *local_e0;
  code *local_d8;
  code *local_d0;
  undefined8 local_c8;
  undefined8 local_c0;
  undefined8 local_b8;
  undefined8 local_b0;
  undefined8 local_a8;
  undefined8 local_a0;
  undefined8 local_98;
  undefined8 local_90;
  undefined8 local_88;
  undefined8 local_80;
  undefined8 local_78;
  undefined8 local_70;
  undefined8 local_68;
  undefined8 local_60;
  undefined8 local_58;
  undefined8 local_50;
  undefined local_48;
  long local_40;
  
  local_40 = *(long *)(in_FS_OFFSET + 0x28);
  local_c8 = 0xe809a6fd21ac5409;
  local_c0 = 0x6963df4161741a59;
  local_b8 = 0x469e424604e6174;
  local_b0 = 0x650f54f9216c6541;
  local_a8 = 0x216c65416ecd696c;
  local_a0 = 0xb34521a4ec099e45;
  local_98 = 0x5534b14c7ae66373;
  local_90 = 0x9624a6be29b29624;
  local_88 = 0x29971c6c9cc229bd;
  local_80 = 0x54099e8b9693a286;
  local_78 = 0x46d32d179e45219a;
  local_70 = 0x361c1a434a2f0816;
  local_68 = 0x604ef82c52063a15;
  local_60 = 0x69696a41617455d4;
  local_58 = 0x3618520a5a3e5c31;
  local_50 = 0x651250135c1a0d77;
  local_48 = 10;
  local_f8 = getenv("BCKDR");
  local_f0 = 0x81;
  local_e8 = 0x80;
  for (puVar3 = auStack_108; puVar3 != auStack_108; puVar3 = puVar3 + -0x1000) {
     *(undefined8 *)(puVar3 + -8) = *(undefined8 *)(puVar3 + -8);
  }
  *(undefined8 *)(puVar3 + -8) = *(undefined8 *)(puVar3 + -8);
  __len = local_f0;
  local_e0 = puVar3 + -0x90;
  for (local_100 = 0; local_100 < local_f0; local_100 = local_100 + 1) {
     puVar3[local_100 - 0x90] = *(byte *)((long)&local_c8 + local_100) ^ local_f8[local_100 % 6];
  }
  *(undefined8 *)(puVar3 + -0x98) = 0x10168f;
  pcVar2 = (code *)mmap((void *)0x0,__len,7,0x22,-1,0);
  __src = local_e0;
  sVar1 = local_f0;
  local_d8 = pcVar2;
  if (pcVar2 == (code *)0xffffffffffffffff) {
     *(undefined8 *)(puVar3 + -0x98) = 0x1016af;
     perror("mmap");
  }
  else {
     *(undefined8 *)(puVar3 + -0x98) = 0x1016d1;
     memcpy(pcVar2,__src,sVar1);
     pcVar2 = local_d8;
     local_d0 = local_d8;
     *(undefined8 *)(puVar3 + -0x98) = 0x1016ed;
     (*pcVar2)();
     pcVar2 = local_d8;
     sVar1 = local_f0;
     *(undefined8 *)(puVar3 + -0x98) = 0x101706;
     munmap(pcVar2,sVar1);
  }
  if (local_40 == *(long *)(in_FS_OFFSET + 0x28)) {
     return;
  }
  __stack_chk_fail();
}
```

Et en cherchant les références vers cette fonction nous trouvons :

```C
void __libc_csu_fini(void)
{
  if (ARMY_STRENGTH < 1) {
     pas_envie_de_t_analyser();
  }
  return;
}
```

On peut donc exécuter `pas_envie_de_t_analyser` lorsque le programme s'interrompt en ayant précedemment attribué une valeur négative à `ARMY_STRENGHT` dans `calculate_mob_army`. Essayons maintenant d'analyser dynamiquement `pas_envie_de_t_analyser`.

## Analyse dynamique

La fonction `pas_envie_de_t_analyser` requiert la variable d'environnement `BCKDR` :

```C
local_f8 = getenv("BCKDR");
```

Commençons par récupérer `BCKDR` :

```console
$ nc 51.20.123.75 1337
              C0d3_Ly0k0                
                                        
                  &&@&                  
                 %@&@@@                 
                 /@%%@(                 
                 /@  @(                 
              *&@@@@@@@@%,              
          @@&             .@@&          
       &@,       ./../.       %@/       
     /@,    .@@*        *@@     #@.     
    %@    ,@/              (@    .@,    
   ,@    /@     *@@&&@@      @,   (@    
   .*   %@#    #@(@@@@.@     %@@   /    
    (@&  ,@@    .@&#@@*@@     @@,  @@.   
     @@    @&                &@   .@&    
      @#    *@%            %@.    @@     
      /%@. .   *@@@&..&@@@,   . #@*/     
         /@@                 ,@@.        
            .@@@@*.&&&#.(@@@@            
           ,@ *   @@@@@%   ( @           
          .@ @.    &  %    (# @          
         .@.@,    .@  @.    ##/@         
        .@.@,      @  @      %@#@        
       ,*@,       @  @       %@&        
                  @%@@                  
                  @%@@                  
                  ,%&,                  
######@@@@@@@ X.A.N.A is watching you @@@~####']-[}
XANA Management Console 
> s
Sending mobs...
How much mobs: 

> 

0
62 26 19 22 11 30
```

Comme on peut le voir dans le code source de `handle_sigfpe`, la variable `BCKDR` est xoré :

```C
     bckdr = getenv("BCKDR");
     length = strlen(bckdr);
     for (index = 0; (ulong)(long)index < length; index = index + 1) {
       printf("%d ",(ulong)(byte)(bckdr[index] ^ 127));
     }
```

Déchiffrons cette variable avec Python :

```Python
>>> var = "62 26 19 22 11 30"
>>> for i in var.split(" "):
...     print(chr(int(i)^127), end="")
... 
Aelita
```

Nous pouvons maintenant exécuter `pas_envie_de_t_analyser` :

```gdb
gef➤  file 8f612a248da7de80738a208bed1d4d19a27621adf12a2f0adcf55a05af9f06a0 
Reading symbols from 8f612a248da7de80738a208bed1d4d19a27621adf12a2f0adcf55a05af9f06a0...
(No debugging symbols found in 8f612a248da7de80738a208bed1d4d19a27621adf12a2f0adcf55a05af9f06a0)
gef➤  set env BCKDR=Aelita
gef➤  r
Starting program: /home/coucou/Documents/Sthack_2024/8f612a248da7de80738a208bed1d4d19a27621adf12a2f0adcf55a05af9f06a0 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
              C0d3_Ly0k0                
                                        
                  &&@&                  
                 %@&@@@                 
                 /@%%@(                 
                 /@  @(                 
              *&@@@@@@@@%,              
          @@&             .@@&          
       &@,       ./../.       %@/       
     /@,    .@@*        *@@     #@.     
    %@    ,@/              (@    .@,    
   ,@    /@     *@@&&@@      @,   (@    
   .*   %@#    #@(@@@@.@     %@@   /    
    (@&  ,@@    .@&#@@*@@     @@,  @@.   
     @@    @&                &@   .@&    
      @#    *@%            %@.    @@     
      /%@. .   *@@@&..&@@@,   . #@*/     
         /@@                 ,@@.        
            .@@@@*.&&&#.(@@@@            
           ,@ *   @@@@@%   ( @           
          .@ @.    &  %    (# @          
         .@.@,    .@  @.    ##/@         
        .@.@,      @  @      %@#@        
       ,*@,       @  @       %@&        
                  @%@@                  
                  @%@@                  
                  ,%&,                  
######@@@@@@@ X.A.N.A is watching you @@@~####']-[}
XANA Management Console 
> s
Sending mobs...
How much mobs: 

> 

-1
Total army strength : -1000 
Reconfiguring mob army...
 Done ! Attack launched !
test
[Inferior 1 (process 9883) exited normally]
```

Le programme redemande une entrée à l'utilisateur. En explorant le code, on comprend que cette entrée est demandé dans un shellcode executé ligne 83 :

```C
(*pcVar2)();
```

Que l'on peut maintenant retrouver dans GDB :

```gdb
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x5555555556d8                  mov    QWORD PTR [rbp-0xc8], rax
   0x5555555556df                  mov    rdx, QWORD PTR [rbp-0xc8]
   0x5555555556e6                  mov    eax, 0x0
●→ 0x5555555556eb                  call   rdx
   0x5555555556ed                  mov    rdx, QWORD PTR [rbp-0xe8]
   0x5555555556f4                  mov    rax, QWORD PTR [rbp-0xd0]
   0x5555555556fb                  mov    rsi, rdx
   0x5555555556fe                  mov    rdi, rax
   0x555555555701                  call   0x555555555240 <munmap@plt>
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── arguments (guessed) ────
*0x7ffff7fc2000 (
   $rdi = 0x00007ffff7fc2000 → 0x8d48c78948c03148,
   $rsi = 0x00007fffffffd6d0 → 0x8d48c78948c03148,
   $rdx = 0x00007ffff7fc2000 → 0x8d48c78948c03148
)
```

```gdb
gef➤  x/34i 0x7ffff7fc2000
   0x7ffff7fc2000:	xor    rax,rax
   0x7ffff7fc2003:	mov    rdi,rax
   0x7ffff7fc2006:	lea    rsi,[rip+0x73]        # 0x7ffff7fc2080
   0x7ffff7fc200d:	mov    edx,0xf
   0x7ffff7fc2012:	syscall
   0x7ffff7fc2014:	lea    rbx,[rip+0x65]        # 0x7ffff7fc2080
   0x7ffff7fc201b:	lea    rsi,[rip+0x4e]        # 0x7ffff7fc2070
   0x7ffff7fc2022:	mov    ecx,0xf
   0x7ffff7fc2027:	xor    rdi,rdi
   0x7ffff7fc202a:	mov    rax,rcx
   0x7ffff7fc202d:	xor    rdx,rdx
   0x7ffff7fc2030:	xor    al,BYTE PTR [rsi]
   0x7ffff7fc2032:	mov    dl,BYTE PTR [rbx]
   0x7ffff7fc2034:	cmp    al,dl
   0x7ffff7fc2036:	jne    0x7ffff7fc2068
   0x7ffff7fc2038:	inc    rsi
   0x7ffff7fc203b:	inc    rbx
   0x7ffff7fc203e:	dec    rcx
   0x7ffff7fc2041:	cmp    rcx,0x0
   0x7ffff7fc2045:	jne    0x7ffff7fc202a
   0x7ffff7fc2047:	mov    rdi,0xffffffffffffffff
   0x7ffff7fc204e:	xor    rsi,rsi
   0x7ffff7fc2051:	xor    rdi,rdi
   0x7ffff7fc2054:	push   rsi
   0x7ffff7fc2055:	movabs rdi,0x68732f2f6e69622f
   0x7ffff7fc205f:	push   rdi
   0x7ffff7fc2060:	push   rsp
   0x7ffff7fc2061:	pop    rdi
   0x7ffff7fc2062:	push   0x3b
   0x7ffff7fc2064:	pop    rax
   0x7ffff7fc2065:	cdq
   0x7ffff7fc2066:	syscall
   0x7ffff7fc2068:	mov    eax,0x3c
   0x7ffff7fc206d:	syscall
```

En explorant dynamiquement ce shellcode, on comprend que notre entrée est récupéré avec le premier syscall, puis comparé à quelque chose dans ces instructions :

```gdb
   0x7ffff7fc2030:	xor    al,BYTE PTR [rsi]
   0x7ffff7fc2032:	mov    dl,BYTE PTR [rbx]
   0x7ffff7fc2034:	cmp    al,dl
```

Si les bytes comparés sont différents, on jump vers un syscall interrompant le shellcode :

```gdb
   0x7ffff7fc2036:	jne    0x7ffff7fc2068
```
```gdb
   0x7ffff7fc2068:	mov    eax,0x3c
   0x7ffff7fc206d:	syscall
```

Sinon, l'opération se répète avec les bytes suivants. Pour récupérer le string auquel notre saisie est comparé, on peut simplement modifier cette instruction `0x7ffff7fc2036:	jne    0x7ffff7fc2068` en un `je` et récupérer le string byte par byte lorsqu'il est en mémoire : `0x7ffff7fc2034:	cmp    al,dl`. Cette étape pourrait être scripté, mais le mot de passe n'est pas très long on l'obtient rapidement :

```gdb
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
 → 0x7ffff7fc2000                  xor    rax, rax
   0x7ffff7fc2003                  mov    rdi, rax
   0x7ffff7fc2006                  lea    rsi, [rip+0x73]        # 0x7ffff7fc2080
   0x7ffff7fc200d                  mov    edx, 0xf
   0x7ffff7fc2012                  syscall 
   0x7ffff7fc2014                  lea    rbx, [rip+0x65]        # 0x7ffff7fc2080
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "8f612a248da7de8", stopped 0x7ffff7fc2000 in ?? (), reason: SINGLE STEP
```
```gdb
gef➤  set {char}0x7ffff7fc2036=0x74
gef➤  br *0x7ffff7fc2034
Breakpoint 10 at 0x7ffff7fc2034
gef➤  c
Continuing.
AAAA
```

```gdb
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x7ffff7fc202d                  xor    rdx, rdx
   0x7ffff7fc2030                  xor    al, BYTE PTR [rsi]
   0x7ffff7fc2032                  mov    dl, BYTE PTR [rbx]
●→ 0x7ffff7fc2034                  cmp    al, dl
   0x7ffff7fc2036                  je     0x7ffff7fc2068
   0x7ffff7fc2038                  inc    rsi
   0x7ffff7fc203b                  inc    rbx
   0x7ffff7fc203e                  dec    rcx
   0x7ffff7fc2041                  cmp    rcx, 0x0
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "8f612a248da7de8", stopped 0x7ffff7fc2034 in ?? (), reason: BREAKPOINT
```
```gdb
gef➤  p $rax
$10 = 0x4a
gef➤  c
Continuing.
```
```gdb
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x7ffff7fc202d                  xor    rdx, rdx
   0x7ffff7fc2030                  xor    al, BYTE PTR [rsi]
   0x7ffff7fc2032                  mov    dl, BYTE PTR [rbx]
●→ 0x7ffff7fc2034                  cmp    al, dl
   0x7ffff7fc2036                  je     0x7ffff7fc2068
   0x7ffff7fc2038                  inc    rsi
   0x7ffff7fc203b                  inc    rbx
   0x7ffff7fc203e                  dec    rcx
   0x7ffff7fc2041                  cmp    rcx, 0x0
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "8f612a248da7de8", stopped 0x7ffff7fc2034 in ?? (), reason: BREAKPOINT
```
```gdb
gef➤  p $rax
$11 = 0x33
```

En répetant cette opération autant que nécessaire on obtient :

```python
>>> print(solu)
[74, 51, 114, 51, 109, 49, 101, 95, 49, 110, 115, 49, 100, 51, 82]
>>> for i in solu:
...     print(chr(int(i)), end="")
... 
J3r3m1e_1ns1d3R
```

## Exploit

Testons le mot de passe :

```console
$ nc 51.20.123.75 1337
              C0d3_Ly0k0                
                                        
                  &&@&                  
                 %@&@@@                 
                 /@%%@(                 
                 /@  @(                 
              *&@@@@@@@@%,              
          @@&             .@@&          
       &@,       ./../.       %@/       
     /@,    .@@*        *@@     #@.     
    %@    ,@/              (@    .@,    
   ,@    /@     *@@&&@@      @,   (@    
   .*   %@#    #@(@@@@.@     %@@   /    
    (@&  ,@@    .@&#@@*@@     @@,  @@.   
     @@    @&                &@   .@&    
      @#    *@%            %@.    @@     
      /%@. .   *@@@&..&@@@,   . #@*/     
         /@@                 ,@@.        
            .@@@@*.&&&#.(@@@@            
           ,@ *   @@@@@%   ( @           
          .@ @.    &  %    (# @          
         .@.@,    .@  @.    ##/@         
        .@.@,      @  @      %@#@        
       ,*@,       @  @       %@&        
                  @%@@                  
                  @%@@                  
                  ,%&,                  
######@@@@@@@ X.A.N.A is watching you @@@~####']-[}
XANA Management Console 
> s
Sending mobs...
How much mobs: 

> 

-1
Total army strength : -1000 
Reconfiguring mob army...
 Done ! Attack launched !
J3r3m1e_1ns1d3R
whoami
ctf
ls
chall
flag.txt
cat flag.txt
STHACK{X@n4_PwN3d_F0r3v3r} 
```