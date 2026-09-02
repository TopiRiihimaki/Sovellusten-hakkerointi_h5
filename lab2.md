Aloitin aluksi komennolla:
```
strings ./passtr
```

Ja löysin mahdollisen salasanan.
```
PTE1
u+UH
What's the password?
%19s
sala-hakkeri-321
Yes! That's the password. FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}
Sorry, no bonus.
;*3$"
GCC: (Debian 14.2.0-3) 14.2.0
long unsigned int
strcmp
```
Päätin kuitenkin tutkia enemmän, koska en voinut todistaa, että tuota salasanaa käytetään.

Käynistin ohjelman eka:
```
gdb ./passtr
```
ja ajoin komennon
```
info functions

```
ja havaitsin nämä:
```
puts@plt
strcmp@plt
__isoc99_scanf@plt
```

scanf → ohjelma lukee käyttäjän syötteen

strcmp → ohjelma vertailee kahta merkkijonoa

puts → ohjelma tulostaa merkkijonon

Koska main-funktiota ei löytynyt info functions -komennolla, päätin tutkia ohjelman _start-funktiota assembly-koodina.

```
disassemble _start
```
Olennaisin rivi oli kuitenkin tämmöinen:


```

0x0000000000001084 <+20>: lea 0xce(%rip),%rdi # 0x1159 <main>



```

Assembly-koodista löytyi kohta, jossa main-funktion osoitteeksi määritettiin 0x1159. Tämän perusteella pystyin tutkimaan main-funktiota komennolla disassemble main

```
(gdb) disassemble _start
Dump of assembler code for function _start:
   0x0000555555555070 <+0>:	xor    %ebp,%ebp
   0x0000555555555072 <+2>:	mov    %rdx,%r9
   0x0000555555555075 <+5>:	pop    %rsi
   0x0000555555555076 <+6>:	mov    %rsp,%rdx
   0x0000555555555079 <+9>:	and    $0xfffffffffffffff0,%rsp
   0x000055555555507d <+13>:	push   %rax
   0x000055555555507e <+14>:	push   %rsp
   0x000055555555507f <+15>:	xor    %r8d,%r8d
   0x0000555555555082 <+18>:	xor    %ecx,%ecx
   0x0000555555555084 <+20>:	lea    0xce(%rip),%rdi        # 0x555555555159 <main>
   0x000055555555508b <+27>:	call   *0x2f2f(%rip)        # 0x555555557fc0
   0x0000555555555091 <+33>:	hlt
End of assembler dump.

```
main-funktion assemblysta pystyin näkemään, että ohjelma lukee ensin käyttäjän syötteen scanf-funktiolla. Tämän jälkeen syötettä verrataan strcmp-funktiolla. strcmp:hän annettua arvoa testataan, minkä perusteella ohjelma päättää, käytetäänkö oikean vai väärän salasanan "reaktiota".

Laitoin sitten
```
break strcmp
```
ja laitoin siihen "hei"

Sitten laitoin:
```
x/s $rdi
```
sekä
```
x/s $rsi
```
(rdi on se arvo, jonka käyttäjä antaa ja rsi on arvo jota ohjelma odottaa)

Joista sain tulosteeksi:
```
0x7fffffffdd70:    "hei"
0x555555556022:    "sala-hakkeri-321"
```
Laitoin sitten tuon oikean salasanan, josta sain sen lipun.

FLAG{Tero-d75ee66af0a68663f15539ec0f46e3b1}

### Oppimani

- Opin tarkemmin, miten break komentoa käytetään.
- disassemble assembly-koodin tutkimiseen
- info functions ohjelman funktioiden tutkimiseen.
- x/s-komennon käyttäminen merkkijonojen tarkasteluun.
- iten strcmp:n palautusarvo vaikuttaa ohjelmaan.



# Toinen tapa jolla ratkaisin

```

(gdb) break main
Breakpoint 1 at 0x1161: file passtr.c, line 10.
(gdb) run
Starting program: /home/topir/Sovellusten_hakkerointi/Dynaaminen analyysi-20260827/lab2/passtr/passtr 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, main () at passtr.c:10
10		printf("What's the password?\n");
(gdb) n
What's the password?
11		scanf("%19s", password);
(gdb) n
^[[A
12		if (0 == strcmp(password, "sala-hakkeri-321")) {
(gdb) n
15			printf("Sorry, no bonus.\n");
(gdb) n
Sorry, no bonus.
17		return 0;
(gdb) n
18	}
(gdb) n
0x00007ffff7de1ca8 in ?? () from /lib/x86_64-linux-gnu/libc.so.6
(gdb)
```



