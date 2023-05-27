

##### Stress Test CPU in Linux

Stress testing your CPU is one of the best ways to check your processor's performance capabilities under heavy load and the system's temperature when that happens..

##### Stress test Linux CPU using the Terminal
You'd need two utilities to stress test using a terminal:
s-tui and stress.

sudo apt install s-tui stress

To store the data, all you have to do is append the -c flag while starting the s-tui utility as shown

s-tui -c
s-tui --csv-file <name of file>.csv
s-tui --csv-file Hello.scv

