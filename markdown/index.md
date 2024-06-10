---
title: 'Controlling an LED Strip Through AWS'
author: '**Mitul Mathur and Austin Bunuan** (website template by Ryan Tsang)'
date: '*EEC172 SQ24*'

toc-title: 'Table of Contents'
abstract-title: '<h2>Description</h2>'
abstract: 'LED strips controllers have been in use for many years, with most 
models utilizing an IR sensor to detect input from a remote to change the 
LED strips color. Our device, however, is configured to accept input not only 
from a standard IR remote but also from an accelerometer, where a minigame is 
displayed on an OLED screen. This approach allows the user to change the LED 
color using either method. The chosen input—whether from the IR remote or the 
accelerometer --is transmitted to an AWS IoT shadow to change the desired color 
change on the LED strip. 
<br/><br/>
Our source code can be found 
<!-- replace this link -->
<a href="https://github.com/FunBunyan/control-led-through-aws">
  here</a>.

<h2>Video Demo</h2>
<div style="text-align:center;margin:auto;max-width:560px">
  <div style="padding-bottom:56.25%;position:relative;height:0;">
    <iframe width="560" height="315" src="https://www.youtube.com/embed/UGESU_c0wPQ?si=CoIdTuwe5fL0UWG3" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
  </div>
</div>
'
---

<!-- EDIT METADATA ABOVE FOR CONTENTS TO APPEAR ABOVE THE TABLE OF CONTENTS -->
<!-- ALL CONTENT THAT FOLLOWS WILL APPEAR IN AND AFTER THE TABLE OF CONTENTS -->

# Design

## System Architecture

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 400px;">
    We used two CC3200 microcontrollers for the system architecture, with one 
    sending updates to change the desired color of the LED strip and the other 
    receiving updates to update the color of the LED strip (our master versus 
    slave board). For our master board, a user can control the Adafruit OLED 
    screen using a remote or the onboard CC3200 accelerometer to send a color to
     our device shadow through AWS. Through WiFi, our slave board checks the 
    device shadow state every so often and sets the LED strip to the desired 
    color. See Figure 1 for the diagram.

  </div>
</div>
<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
    <div style="display:inline-block;vertical-align:top;flex:0 0 400px;">
        <div class="fig">
        <img src="./media/Architecture.jpg" style="width:100%;height:auto;" />
        <span class="caption">Figure 1: Architecture Diagram</span>
        </div>
    </div>
</div>

## Functional Specification

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
  <div style="display:inline-block;vertical-align:top;flex:1 0 300px;">
    <p>
    We used the following state logic to control the LED strip, see Figure 2 
    below. On the master board, upon startup, the user is greeted with a main 
    menu screen, giving them the choice to control the OLED with a remote or 
    accelerometer. After choosing a mode, the user is greeted with a different 
    screen depending on what mode they chose. When the user is ready to send a 
    color to AWS, they can either press SW3 (if they are in the accelerometer 
    control mode) or “Mute” (if they are in the remote control mode) to send a 
    color to our desired shadow state through AWS. 
    </p>
    <p>
    On the slave board, we used a cycle to control the LED strip. It starts by 
    waiting 15 seconds before connecting to AWS and obtaining the desired color 
    from our shadow. Depending on the color specified, the slave board sets 
    signals to MOSFETS controlling our LED strip to change the color of the LED 
    strip. Next, the reported color in our device shadow is updated, where the 
    cycle starts over again.
  </div>
</div>
<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
    <div style="display:inline-block;vertical-align:top;flex:0 0 500px">
        <div class="fig">
        <img src="./media/State_Diagram.jpg" style="width:100%;height:auto;" />
        <span class="caption">Figure 2: Functional Specification Diagram</span>
        </div>
    </div>
</div>

# Implementation
For our implementation, we decided to use two CC3200 boards where one board 
would send updates to AWS and the other would check AWS for updates.  Neither 
board is directly connected to the other, where both exclusively use AWS to 
communicate. Our master board uses an IR sensor, an Adafruit OLED board, and 
the onboard CC3200 accelerometer. Our slave board uses three MOSFETs to control 
the LED strip, one to connect each color (red, blue, and green).

## AWS IoT Core

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 200px'>
    Doing the initial setup, we set up a device shadow on AWS, downloaded 
    certificates and flashed them to our CC3200 boards, attached policies to 
    that certificate, and converted the certificates from a .pem format to a 
    .der format using OpenSSL version 1.1.1. For our master CC3200 board, we 
    update the desired section of our shadow, specifically a field called 
    “color”. In our slave device, to update the LED strip, we use a GET request 
    to obtain the JSON data, read the desired color, and then update the 
    reported section to the new color, if the read was successful. 
    Upon an unsuccessful read, the LED strip blinks 3 times to let the user 
    know the data was not retrieved properly.
  </div>
</div>

## Adafruit OLED board

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    For our master device, we use the Adafruit OLED board as a UI and menu 
    screen for the user. The SPI communication protocol to communicate with the 
    CC3200. For our project, there are three different “screens”: the main menu 
    screen, a screen if the user selects to control LED lights with the 
    accelerometer, and another to control the lights with a remote control. The 
    main menu screen prints how the user can enter either screen: to use remote 
    control mode a number button can be pressed on the remote; to enter 
    accelerometer mode SW3 or SW2 can be pressed on the CC3200. The user also 
    gets feedback printed on the OLED screen after they select a color to send 
    in either control mode. The user can also navigate back to the main menu 
    screen if the user wants to control the LED strip using a different mode.
  </div>
  <div style='display: inline-block; vertical-align: top;flex:0 0 400px'>
    <div class="fig">
      <img src="./media/Adafruit_OLED.png" style="width:auto;height:2in" />
      <span class="caption">Figure 3: Adafruit OLED Circuit Diagram</span>
    </div>
  </div>
</div>

## IR Mode

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    One way to control the LED strip was with IR signals from a remote. The 
    Vishay IR sensor is connected to Pin 50 on the CC3200, where every time a 
    signal is detected interrupts are enabled to read the signal. Before taking 
    input from the remote, the Adafruit OLED screen prints out a list of valid 
    colors and instructions on how to send data to AWS. Using an IR sensor in 
    combination with the OLED screen, we decoded signals from the remote 
    controller as text input for the OLED screen. These signals include: “0” is 
    a space character, “1” goes back to the main menu screen, “Last” deletes the
     last inputted character, “Mute” sends the color (if valid) to AWS, and any 
    other number button types text. It is possible to cycle through different 
    letters for one number button by pressing it quickly. After the user 
    presses the “Mute” button, it will print a message to the user. If the 
    text was an invalid color, the message printed is “Invalid color, try 
    again”. Otherwise, this means the color was valid and the message printed 
    is “Sending color… Sent!”. In either case, after the message is printed, 
    the message is cleared and the user can type input with the remote once 
    more.
  </div>
  <div style='display: inline-block; vertical-align: top;flex:0 0 400px'>
    <div class="fig">
      <img src="./media/IR_reciever.png" style="width:auto;height:2in" />
      <span class="caption">Figure 4: IR Receiver Wiring Diagram</span>
    </div>
  </div>
</div>

## Accelerometer Mode

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    The other way to control the LED strip was by using the onboard 
    accelerometer on the CC3200. Note that I2C was used to access the data from 
    the CC3200 processor. When entering accelerometer mode, 8 rectangles are 
    drawn of different colors (2 rows, 4 columns, 32 x 40 pixels long) along 
    with a message that tells the user to send a color using buttons SW3 and SW2
     on the CC3200. Afterward, the user can tilt the CC3200 board to control a 
    small gray square (4 pixels wide) on the screen where the rectangles are 
    (the square the user controls is bound to the area where the colored 
    rectangles are). The speed of the square the user controls depends on how 
    much they tilt the accelerometer, where the larger the tilt the faster the 
    square moves. To select a color, the user moves the square into a colored 
    rectangle and presses SW3. When the user presses SW3, the coordinates of 
    the gray square are checked, and will send the selected color to AWS. The 
    user gets feedback when SW3 is pressed, where a message confirms they 
    pressed the SW3 button and the color that was selected. To navigate back to 
    the main menu, the user can press SW2.
  </div>
</div>

## Getting Desired State

<div style="display:flex;flex-wrap:wrap;justify-content:space-between;">
  <div style='display: inline-block; vertical-align: top;flex:1 0 300px;'>
    For our slave board, we used a simple cycle to update the LED strip. 
    Every 15 seconds, this CC3200 board connects to AWS and sends a GET request 
    to our shadow state. We then parsed the GET response to find the JSON body, 
    then used the cjson library to extract the “color” field from the JSON 
    object. We stored the value of the “color” field in a string and compared 
    it to our known colors. If this string is a valid color, the three GPIO 
    signals outputting to our MOSFETs are adjusted accordingly to low or high. 
    For example, “RED” would set the GPIO signal controlling the red MOSFET to 
    high and setting the blue and green GPIO signals to low. If the board 
    cannot read the JSON object, it will cause the LED strip to blink 3 times 
    and sets the LED strip color to off (or “black”). Once the color is 
    updated, the slave board sends a POST request to AWS to update the reported 
    color state and waits another 15 seconds to start the cycle over.
  </div>
  <div style='display: inline-block; vertical-align: top;flex:1 0 400px'>
    <div class="fig">
      <img src="./media/MOSFET.png" style="width:auto;height:3in" />
      <span class="caption">Figure 5: MOSFET and LED strip Circuit Diagram</span>
    </div>
  </div>
</div>

# Challenges

## UI and Pixel Alignment

The largest challenge we faced was setting up the UI for the Adafruit OLED 
board. Initially, we only tested our slave and master boards for functionality 
and thought the UI design part would be much easier. However, we quickly 
realized that we needed to do a lot of pixel math to make everything look nice. 
After experimenting with the limits of what the Adafruit library could do, we 
opted for a simple but clean UI. This was frustrating, as it took a lot of 
trial, error, and time to get this to work. 

For example, one of these issues was printing words, where words would get cut
off and move to the next line. If this happened, we needed to add spaces 
between words to force a word to the next line. If the word was too big the 
spacing looked weird. To solve this, we selected words that could fit in one 
line of the OLED board and limited the extra space.


## Accelerometer - Controlling the gray square

Setting up the underlying logic to control the ball was easy: move the gray 
square in a direction that matches the direction of the accelerometer and have 
bound checks to be sure the gray square didn’t go offscreen or into the text 
below. To show that the gray square was “moving,” we kept the previous 
coordinates of the gray square and drew over it with the color of the rectangle 
the gray square was previously in, then drew a new square, incrementing or 
decrementing the old value accordingly. We realized, however, that if the gray 
square crossed the line separating each colored rectangle, parts of one colored 
rectangle would be overwritten by another. (Figure 6 below). This was because 
the stored coordinates of the gray square start at the top left corner of the 
square, and when the old square was drawn over it would see that the 
coordinates were still in the previous square, causing one rectangle’s color to 
be overridden. To fix this problem, since all colored rectangles are 32 by 40 
pixels long, the gray square can only move in multiples of 4.

<div style="display:flex;flex-wrap:wrap;justify-content:space-evenly;">
    <div style="display:inline-block;vertical-align:top;flex:0 0 400px;">
        <div class="fig">
        <img src="./media/Overlap.png" style="width:100%;height:auto;" />
        <span class="caption">Figure 6: Rectangle being overridden</span>
        </div>
    </div>
</div>

# Future Work

If we had more time, one of the improvements that we would make is adding a 
fade effect to the lights when receiving a new color. This could be done with 
pulse width modulation with the GPIO pins. Another would be adding different 
modes or patterns that could control the LED strip. Lastly, the most ambitious 
idea we would want to implement would be using an AWS lambda function and the 
Amazon Alexa App to control lights using voice control. 


# Finalized BOM

<!-- you can convert google sheet cells to html for free using a converter
  like https://tabletomarkdown.com/convert-spreadsheet-to-html/ -->

| Item                        | Quantity | Cost (1 unit)               | Where to Find                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------------------- | -------- | --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 100 Ohm resistor            | 1        | $0.55                       | [Amazon](https://www.amazon.com/Projects-10EP512100R-100-Resistors-Pack/dp/B0185FJZRK/ref=sr_1_3?dib=eyJ2IjoiMSJ9.sg5HHu4hbSXqU6GTsPk0qZIRGYaVB6ekA53OO372djsRAg9ens9db7QrXYwYbDp5wnRMKpEm8r_wUnDTuN4L4mImbJQ_rMMEh6plC_AVZ-9WWea43JtbeZkqEVTBtTVG17McdV2BrFJ0QPVrfM4l26IEhfafZzZBg9Ft4kgwuupB73NZWHQRx3eDX3xwez4UJOuB4nPYyQgwlNMmHgbRtHRAeV4H07LYpFHytvRjobI.SIXmWih4XSQ6m2iTmpXfm8kucvY2n6uPUxUPMicTlIM&dib_tag=se&keywords=100+ohm+resistor&qid=1716322236&sr=8-3)                                                                                                                                     |
| 100 uF Capacitor            | 1        | $4.60                       | [DigiKey](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/860020381030/5728762)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| 12 V DC Power Connector     | 1        | $0.50                       | [Amazon](https://www.amazon.com/Power-Connector-Female-Adapter-Camera/dp/B07C61434H/ref=sr_1_7?crid=22G53T67SAJSZ&dib=eyJ2IjoiMSJ9.FqAQaYOVtdp3GCxuf_suUsNDxAcBO8pLnyLl9EuKB1eCK5aAwdaSm6pb0ETR9LOUaHxaVKpebL750nM7wWQV8XQF7mGq8tJum4DMkwOQEX8l117903gfEILQWYRIJEFMtez68ue-C2L4iBCzmLsAqbzYxIgM5LQvAlZ2jzoJJts3WtjubLDgYg3NePmVh86SEhcFQqJAXiLupzCYF4eIAQk7tyg8_cFqmY9T1TSU3jo3h2VuFcwGb7mRSQVQB8Lq3XZ_EQizeB2JAAQFf6xTl1EGyYXiOE-9Smf_JvfgfuY.eDRYJ7zPjgqhQ2gX3W9KhEcgOeP2GivtUctH1-_gJKw&dib_tag=se&keywords=dc+wire+adapter&qid=1716370589&s=industrial&sprefix=dc+wire+ada%2Cindustrial%2C158&sr=1-7) |
| Adafruit SSD1351 OLED Board | 1        | $38.95                      | [Amazon](https://www.amazon.com/Adafruit-OLED-Breakout-Board-microSD/dp/B00XW2ME84)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| DC Power Adapter            | 1        | Free (Comes with LED strip) | [Amazon](https://www.amazon.com/Daybetter-Lights-32-8ft-Remote-Changing/dp/B08D99TWQJ/ref=sr_1_9?crid=389XVSVDL6TIV&dib=eyJ2IjoiMSJ9.6pZHq71t9htVUcz1LgS5kZ-SgxI3pkg75l4cnjxc32qZQke3siLm9rd09MghO5RD9R6isj49yJhzjkCx1gbUjW283TesApvz1F4idRWASkxIHpsiY_9jdmzChGWOYRQB1DuaFGnSU6ujCgFHnEeNc2g7DnV3D2DjWFFDL2oqii-GTGnL48xlQK_z-Y92nTb3E912O46-JMeoUJ_CxjkNxcUTUkglQif3bU1GJ9zMc7h-0P4Rb40rmMAON0vThRALvl7evpg9FKtyBWfe4j5yPvCK8oqR4ZSgR74KtNMLCRU.tjRi8_R88IHUE3xIgCll0GW0xStZnja4C4EQyeKb03k&dib_tag=se&keywords=LED%2Bstrip&qid=1716369993&sprefix=led%2Bstrip%2Caps%2C163&sr=8-9&th=1)                  |
| IR Sensor                   | 1        | $1.40                       | [Mouser](https://www.mouser.com/ProductDetail/Vishay-Semiconductors/TSOP31140?qs=FL6C5OO8pQ68QKD80z1xdg%3D%3D&utm_source=netcomponents&utm_medium=aggregator&utm_campaign=TSOP31140&utm_content=Vishay)                                                                                                                                                                                                                                                                                                                                                                                                   |
| Jumper wires                | 1        | $1.00                       | [Amazon](https://www.amazon.com/EDGELEC-Breadboard-Optional-Assorted-Multicolored/dp/B07GD2BWPY)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| LED strip                   | 1        | $10.00                      | [Amazon](https://www.amazon.com/Daybetter-Lights-32-8ft-Remote-Changing/dp/B08D99TWQJ/ref=sr_1_9?crid=389XVSVDL6TIV&dib=eyJ2IjoiMSJ9.6pZHq71t9htVUcz1LgS5kZ-SgxI3pkg75l4cnjxc32qZQke3siLm9rd09MghO5RD9R6isj49yJhzjkCx1gbUjW283TesApvz1F4idRWASkxIHpsiY_9jdmzChGWOYRQB1DuaFGnSU6ujCgFHnEeNc2g7DnV3D2DjWFFDL2oqii-GTGnL48xlQK_z-Y92nTb3E912O46-JMeoUJ_CxjkNxcUTUkglQif3bU1GJ9zMc7h-0P4Rb40rmMAON0vThRALvl7evpg9FKtyBWfe4j5yPvCK8oqR4ZSgR74KtNMLCRU.tjRi8_R88IHUE3xIgCll0GW0xStZnja4C4EQyeKb03k&dib_tag=se&keywords=LED%2Bstrip&qid=1716369993&sprefix=led%2Bstrip%2Caps%2C163&sr=8-9&th=1)                  |
| MOSFET                      | 3        | $1.00                       | [Amazon](https://www.amazon.com/Bridgold-N-Channel-Transistor-International-Rectifier/dp/B07MW1N4Q5/ref=sr_1_3?c=ts&dib=eyJ2IjoiMSJ9.UtcJgo37Wa6pr8fpKO2Wqlb-yQ8QAVGGH2IL2N6gPXcz4WBR1jg0u5qV0yM5CamTCXr0OQWaEp4xUgZ9tL-czmvLGVMz3INZ47wLCANLXeSkpGINcE-PIVfSzUq60xTYOWH4-sT73QvM266Oy2kgULgoAo-6TYBpY6iUoTkMCegySaWQY13P26u3fOv49w5j2pAgPq8jnL2fVeUB9e68qdNQWYyHiG2hBuzEMRvniEM1kjhX3DRurGyfh8_vx7G30ysAuPnN_9JjF85w33oC35CkNcOx41DHtcyUTHoL9qg.0okQepDECHf8ioMHQEcsJy18hTilHDXPZh_9qZ6Y6ms&dib_tag=se&keywords=MOSFET+Transistors&qid=1716370154&s=industrial&sr=1-3&ts_id=306919011)                   |
| TI CC3200 Microcontroller   | 2        | $55.00                      | [Ti.com](https://www.ti.com/tool/CC3200-LAUNCHXL#order-start-development)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
# References    

- Industries, Adafruit. “OLED Breakout Board - 16-Bit Color 1.5" W/Microsd Holder.” Adafruit Industries Blog RSS, www.adafruit.com/product/1431
- “Starfield Class 2 CA Certificate.” StarfieldClass2CA.crt.der, https://canvas.ucdavis.edu/courses/897392/files/folder/lab/LAB4?preview=24265625
- Ti CC3200 Launchpad DataSheet, www.ti.com/lit/ds/symlink/cc2630.pdf

## Code References
- Tsang, “Lab 4 Blank project.” lab4-blank.zip, https://canvas.ucdavis.edu/courses/897392/files/folder/lab/LAB4?preview=24264023
- Mathur, Bunuan, “Lab2Part4 Code”, lab2_MitulMathur_AustinBunuan.zip (submitted to Canvas)
- Mathur, Bunuan, “Lab4Part3 Code”, lab4_MitulMathur_AustinBunuan.zip (submitted to Canvas)
- Tsang, “GPIO_interupt_example”, lab3-examples.zip, https://canvas.ucdavis.edu/courses/897392/files/folder/lab/LAB3?preview=24137025

