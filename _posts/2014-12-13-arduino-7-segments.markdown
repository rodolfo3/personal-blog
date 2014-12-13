---
layout: post
title:  "Arduino: 7 segments display (less pins)"
date:   2014-12-13 00:00:00 -2
categories: arduino
---

Control a 7 segment display with an arduino is easy:

<img src="{{ "/2014-12-12-arduino-7-segments/7-segment-small.png" | prepend:site.staticurl }}" />

Assembling an common cathode display like the picture above, when the pin is `HIGH`, it turns on one of the segments. If the display is a common anode, the difference is the middle pin should be `5+` and `LOW` in the I/O pin turns the segment on.

But using this approach, if you need 2 displays, you'll use 14 I/O ports. There is some solutions using other CI, but is possible do the same "multiplexing" it. It is the same used to build LED matrix. We should use 7 pins to control the segments and other pins to select every display. With this, is possible control 2 displays with 9 pins - or 3 displays with 10 pins.

The assembly is quite easy:

<img src="{{ "/2014-12-12-arduino-7-segments/2x7-segment-small.png" | prepend:site.staticurl }}" />

The code is a little bit more complicated:

``` c
#define SEGMENT_0_PIN 11
#define SEGMENT_1_PIN 12

#define SEGMENT_START_PIN 2

void turn_all_off() {

  digitalWrite(SEGMENT_0_PIN, HIGH);
  digitalWrite(SEGMENT_1_PIN, HIGH);

  for (int i=SEGMENT_START_PIN; i<SEGMENT_START_PIN + 7; i++) {
    digitalWrite(i, LOW);
  }
}

void setup() {
  // put your setup code here, to run once:
  for (int i=SEGMENT_START_PIN; i<SEGMENT_START_PIN + 7; i++) {
    pinMode(i, OUTPUT);
  }

  pinMode(SEGMENT_0_PIN, OUTPUT);
  pinMode(SEGMENT_1_PIN, OUTPUT);

  turn_all_off();
}

void loop() {
  digitalWrite(SEGMENT_0_PIN, LOW);
  digitalWrite(SEGMENT_START_PIN, HIGH);
  digitalWrite(SEGMENT_START_PIN + 6, HIGH);

  delay(5);
  turn_all_off();

  digitalWrite(SEGMENT_1_PIN, LOW);
  digitalWrite(SEGMENT_START_PIN + 2, HIGH);
  digitalWrite(SEGMENT_START_PIN + 4, HIGH);

  delay(5);
  turn_all_off();
}
```

When `SEGMENT_0_PIN` is `LOW`, the Arduino connects it to the ground. If the pin of the segment it `HIGH`, then the segment on the display attached on `SEGMENT_0_PIN` is on. The `SEGMENT_1_PIN` is completely off at this point - the `GND` pin is `HIGH`, so the LED cannot turn on.

You don't see the LED blink because they blink too fast to see. If you put higher values on the `delay(5)`, you'll see the effect.
When I assembly this, I do not put the resistors: it can reduce the life of LEDs in the displays, but it makes it brighter.

There is how it looks like in my breadboard:

<img src="{{ "/2014-12-12-arduino-7-segments/2x7-segment-assembled-small.jpg" | prepend:site.staticurl }}" />

Next step is encapsulate this in functions (or classes) and keep the complexity out of the rest of the project - and make it reusable.
