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

Il prend en entrée un charactère et s'interrompt. Analysons le code source pour mieux comprendre.

## Analyse statique



## Exploit


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