ADM3A

A user asked on the Altair-Duino user group if it was possible to patch
a program that used ADM-3A terminal codes to run with VT100. The program
in question is FLYKPRO.COM (which is included in this repository).

The program was written to run on a Kaypro, which has some terminal
emulation in its BIOS. The terminal emulated was the Lear Siegler ADM-3A.
The Operaters Manual for that terminal is adm3a-om.pdf, also in this
repository. The Operators Manual gives all the functions of the terminal.

The ADM-3A is actually close to the VT52. For example cursor position is
ESC = y x and the VT52 is ESC Y y x, where y and x are single characters...
space is value 1.

All we need to do is modify the output stream from the program, translating
ESC = to ESC Y and the other control codes that the ADM-3A supports. For
example - Control-^ homes the cursor, and Control-Z clears the screen.

I determined that there is a bit of memory available in page zero (addresses
00 to FF) that can be used. Specifically -- 14 to 5b is available on my
Altair (DDT uses 38 to 3a so those three bytes need to be skipped).

The CONOUT entry in the BIOS dispatch table is patched to to point at the
interception code.

However, in adm3a.asm, the interception values are hard-coded. This needs
to be patched to support other CP/M systems. The values in the repository
support 63K Hard Disk CP/M 2.2 -- either Mike Douglas or my version
(see repository https://github.com/ratboy666/hd1024). I have been using
this CP/M on my Altair for 4 years, and it is rock stable. adm3a.asm can
be patched for other systems, and this could be automated. Pull requests
are welcome.

Since the code in adm3a.asm is resident BELOW 100H, it cannot be made
into a COM file. The loader at 100H could be made smarter and relocate
the code into its target location. I use DDT to enable this function.

But, first, VT100 is NOT the target. The VT100 has an ESC code which puts
it into VT52 compatible mode. Running VT52.COM sends that sequence.
After this a real VT100 will be in VT52 mode. Hopefully, your terminal
emulator is good enough to support this -- xterm is ok (and that is
what I use). CLS.COM returns to VT100 mode.

After putting the terminal into VT52 mode

DDT ADM3A.HEX
-G

(or G100) will enable ADM-3A to VT52. To disable:

DDT ADM3A.HEX
-G107

The entry at 107H restores the original BIOS table entry, and thus restores
normal terminal operation.


M80 =ADM3A/L
L80

ADM3A.HEX

DDT ADM3A.HEX
-L0
  0000  JMP  ED03
-LED03
  ED03  JMP  EDD9 <- WBOOT
  ED06  JMP  F286 <- CONST
  ED09  JMP  F293 <- CONIN
  ED0C  JMP  F2AC <- CONOUT

PATCH CONOUT (F2AC) TO 0021: THIS IS DONE IN THE SOURCE

(THIS IS ENTERED ALREADY IN ADM3A.HEX -- FOR MY CP/M)

AT LOCATION 100H, WE SET THE BIOS ENTRY FOR CONOUT TO OUR ROUTINE AT
003BH

0100  LXI H,003B
0103  SHLD FF0D
0106  JMP 0000

-G100

A>

AND WE HAVE ADM3A ENABLED. TO DISABLE:

0100 LXI H,FE4C
0103 SHLD FF0D
0106 JMP 0000


