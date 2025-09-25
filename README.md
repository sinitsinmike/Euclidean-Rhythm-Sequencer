# Euclidean-Rhythm-Sequencer
![eucledian](https://github.com/user-attachments/assets/8744ffaa-ce4c-4743-8015-d6ab0818bc85)
![Eucledian2](https://github.com/user-attachments/assets/13ee1653-2738-49db-8298-116511b59ade)


Building-Your-Own-Modular-Synth
Euclidean Rhythm Sequencer for 800 Yen - Building Your Own Modular Synth
Header image
Euclidean Rhythm Sequencer for 800 Yen - Building Your Own Modular Synth

94
HAGIWO
HAGIWO
September 2, 2021 01:14
I made a modular synthesizer called Euclidean rhythm sequencer using Arduino nano, so here are my notes.


background
This is my 30th modular synth project.
Being able to play Euclidean rhythms may be one of the defining features of modular synths.
Most rhythm machines on the market don't have a built-in Euclidean rhythm sequencer. However, there are many Euclidean rhythm sequencers available for Eurorack.

In order to create rhythms that are different from dance music, I reused the hardware of a "6CH Trigger Sequencer" that I had created in the past, and created the Euclidean Rhythm Sequencer.


Production specifications
Eurorack standard 3U 6HP size
Power supply: Standby 37mA (at 5V or 12V) Output 100mA (at 5V or 12V)
Can operate on a single 5V power supply or a single 12V power supply.
OLED display: 128 * 64 size
Rotary encoder: Parameter selection
PUSH button: Parameter change/decision
Trigger output: 6CH (0-5V)
Trigger input: 1CH (0-5V)

Image 2
Two modes
Manual mode : Set the parameters of each channel as desired for playback.
The selectable parameters are as follows:

HITS : Number of times out of 16 steps are output.

OFFSET : Shifts the start of the rhythm by the offset

LIMIT : Returns to step 1 when there is a trigger input the number of times set. For example, if you set it to 5, it will return to step 1 after outputting steps 1 to 5. Can be used in a polyrhythmic way.

MUTE : Eliminates trigger output for the selected CH.

RESET : Pressing the button returns the playback step of all CHs to step 1.

Random mode : Each parameter of each channel switches randomly each time a specified beat is reached. It is not completely random, but there is a tendency for the values ​​selected by each channel.

OCCURRENCE: When the specified number of steps is reached, each parameter will change randomly. The options are "2, 4, 8, 16" and can be seen in the bar at the bottom left of the screen.

Image 1
Production cost
Total price about 800 yen
----------------------------------
Arduino nano 200 yen
OLED (SSD1306) 180 yen
Panel 150 yen
Rotary encoder 80 yen
Others

Image 4
programming
About Euclidean sequences:
Euclidean rhythms are calculated using advanced calculations, but these calculations are not performed in the program. If you limit it to 16 steps, there are only 17 rhythm patterns (0 hits to 16 hits), so the rhythm patterns are stored in a table.

const static byte euc16[17][16] PROGMEM = {//euclidian rythm

 {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0},
 {1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0},
 {1, 0, 0, 1, 0, 1, 0, 0, 1, 0, 0, 1, 0, 1, 0, 0},
 {1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0},
 {1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0},
 {1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1, 0},
 {1, 0, 1, 1, 0, 1, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1},
 {1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1},
 {1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1},
 {1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1},
 {1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1},
 {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0},
 {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1}
};

copy
Exclusive processing of OLED display:
When OLED is displayed on the screen, latency occurs in the trigger output. Therefore, the OLED display is updated immediately after the trigger output ends. This is inconvenient because the screen display is not updated unless there is a trigger input, but you can increase the screen update frequency by enabling comment out on the source.

  //    disp_reflesh = 1;//Enable while debugging.

copy
OLED graphic display:
A polygon is displayed connecting the vertices of the valid steps. 3 hits results in a triangle, and 4 hits results in a square. The program to display this polygon is as follows:

  for (k = 0; k <= 5; k++) { //ch count
   buf_count = 0;
   for (m = 0; m < 16; m++) {
     if (offset_buf[k][m] == 1) {
       line_xbuf[buf_count] = x16[m] + graph_x[k];//store active step
       line_ybuf[buf_count] = y16[m] + graph_y[k];
       buf_count++;
     }
   }

   for (j = 0; j < buf_count - 1; j++) {
     display.drawLine(line_xbuf[j], line_ybuf[j], line_xbuf[j + 1], line_ybuf[j + 1], WHITE);
   }
   display.drawLine(line_xbuf[0], line_ybuf[0], line_xbuf[j], line_ybuf[j], WHITE);
 }
 for (j = 0; j < 16; j++) {//line_buf reset
   line_xbuf[j] = 0;
   line_ybuf[j] = 0;
 }

copy
The coordinates of the vertices are temporarily stored in a buffer for drawing lines.
After drawing the line connecting those coordinates, the buffer is deleted.
This process is repeated for channels 1 to 6.

Memory usage:
The Arduino IDE shows that only 31% of the RAM is being used, but increasing the program's processing load further makes it unstable.
For example, increasing the number of characters displayed on the OLED or using complex conditional if statements often causes the Arduino to stop responding.
Perhaps it was incompatibility with the library? The cause is unknown.

Image 3
Hardware
The circuit is the same as the 6ch trigger sequencer.
By rewriting the software, you can change the functions.

Advertisement: Support open source projects
To continue this open source DIY modular synth project, we are looking for patrons through a service called Patreon.
We would be grateful if you could support us with as little as a cup of coffee.
We also provide content exclusive to Patreons.

Get more from HAGIWO on Patreon
creating DIY eurorack modular synthesizer
www.patreon.com
Source code
It's rough, but I'm releasing it. If you have any problems, please let me know.
It's not mandatory, but if you create a new product based on this source code, I'd appreciate it if you could include a link to this blog (or YouTube).

euclid_pub.ino
12.1 KB
Header image
Euclidean Rhythm Sequencer for 800 Yen - Building Your Own Modular Synth

94
HAGIWO
HAGIWO
September 1, 2021 19:14
I made a modular synthesizer called Euclidean rhythm sequencer using Arduino nano, so here are my notes.


background
This is my 30th modular synth project.
Being able to play Euclidean rhythms may be one of the defining features of modular synths.
Most rhythm machines on the market don't have a built-in Euclidean rhythm sequencer. However, there are many Euclidean rhythm sequencers available for Eurorack.

In order to create rhythms that are different from dance music, I reused the hardware of a "6CH Trigger Sequencer" that I had created in the past, and created the Euclidean Rhythm Sequencer.


Production specifications
Eurorack standard 3U 6HP size
Power supply: Standby 37mA (at 5V or 12V) Output 100mA (at 5V or 12V)
Can operate on a single 5V power supply or a single 12V power supply.
OLED display: 128 * 64 size
Rotary encoder: Parameter selection
PUSH button: Parameter change/decision
Trigger output: 6CH (0-5V)
Trigger input: 1CH (0-5V)

Image 2
Two modes
Manual mode : Set the parameters of each channel as desired for playback.
The selectable parameters are as follows:

HITS : Number of times out of 16 steps are output.

OFFSET : Shifts the start of the rhythm by the offset

LIMIT : Returns to step 1 when there is a trigger input the number of times set. For example, if you set it to 5, it will return to step 1 after outputting steps 1 to 5. Can be used in a polyrhythmic way.

MUTE : Eliminates trigger output for the selected CH.

RESET : Pressing the button returns the playback step of all CHs to step 1.

Random mode : Each parameter of each channel switches randomly each time a specified beat is reached. It is not completely random, but there is a tendency for the values ​​selected by each channel.

OCCURRENCE: When the specified number of steps is reached, each parameter will switch randomly. The options are "2, 4, 8, 16" and can be checked in the bar at the bottom left of the screen.

Image 1
Production cost
Total price about 800 yen
----------------------------------
Arduino nano 200 yen
OLED (SSD1306) 180 yen
Panel 150 yen
Rotary encoder 80 yen
Others

Image 4
programming
About Euclidean sequences:
Euclidean rhythms are calculated using advanced calculations, but these calculations are not performed in the program. If you limit it to 16 steps, there are only 17 rhythm patterns (0 hits to 16 hits), so the rhythm patterns are stored in a table.

const static byte euc16[17][16] PROGMEM = {//euclidian rythm
 {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0},
 {1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0},
 {1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0},
 {1, 0, 0, 1, 0, 1, 0, 0, 1, 0, 0, 1, 0, 1, 0, 0},
 {1, 0, 0, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0},
 {1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0},
 {1, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1, 0},
 {1, 0, 1, 1, 0, 1, 0, 1, 1, 0, 1, 1, 0, 1, 0, 1},
 {1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1},
 {1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1},
 {1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1},
 {1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1},
 {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0},
 {1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1}
};

copy
Exclusive processing of OLED display:
When OLED is displayed on the screen, latency occurs in the trigger output. Therefore, the OLED display is updated immediately after the trigger output ends. This is inconvenient because the screen display is not updated unless there is a trigger input, but you can increase the screen update frequency by enabling comment out on the source.

  //    disp_reflesh = 1;//Enable while debugging.

copy
OLED graphic display:
A polygon is displayed connecting the vertices of the valid steps. 3 hits results in a triangle, and 4 hits results in a square. The program to display this polygon is as follows:

  for (k = 0; k <= 5; k++) { //ch count
   buf_count = 0;
   for (m = 0; m < 16; m++) {
     if (offset_buf[k][m] == 1) {
       line_xbuf[buf_count] = x16[m] + graph_x[k];//store active step
       line_ybuf[buf_count] = y16[m] + graph_y[k];
       buf_count++;
     }
   }

   for (j = 0; j < buf_count - 1; j++) {
     display.drawLine(line_xbuf[j], line_ybuf[j], line_xbuf[j + 1], line_ybuf[j + 1], WHITE);
   }
   display.drawLine(line_xbuf[0], line_ybuf[0], line_xbuf[j], line_ybuf[j], WHITE);
 }
 for (j = 0; j < 16; j++) {//line_buf reset
   line_xbuf[j] = 0;
   line_ybuf[j] = 0;
 }

copy
The coordinates of the vertices are temporarily stored in a buffer for drawing lines.
After drawing the line connecting those coordinates, the buffer is deleted.
This process is repeated for channels 1 to 6.

Memory usage:
The Arduino IDE shows that only 31% of the RAM is being used, but increasing the program's processing load further makes it unstable.
For example, increasing the number of characters displayed on the OLED or using complex conditional if statements often causes the Arduino to stop responding.
Perhaps it was incompatibility with the library? The cause is unknown.

Image 3
Hardware
The circuit is the same as the 6ch trigger sequencer,
but the functionality can be changed by rewriting the software.

Advertisement: Support open source projects
To continue this open source DIY modular synth project, we are looking for patrons through a service called Patreon.
We would be grateful if you could support us with as little as a cup of coffee.
We also provide content exclusive to Patreons.

Get more from HAGIWO on Patreon
creating DIY eurorack modular synthesizer
www.patreon.com
Source code
It's rough, but I'm releasing it. If you have any problems, please let me know.
It's not mandatory, but if you create a new product based on this source code, I'd appreciate it if you could include a link to this blog (or YouTube).
