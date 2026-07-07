# Audio Visualizer
Replace this text with a brief description (2-3 sentences) of your project. This description should draw the reader in and make them interested in what you've built. You can include what the biggest challenges, takeaways, and triumphs from completing the project were. As you complete your portfolio, remember your audience is less familiar than you are with all that your project entails!

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Dylan T | Evergreen Valley High School | Mechanical Engineering | Incoming Senior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/y3VAmNlER5Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/EOAn__2-9FA?si=JDM3DLJOGKv7l_py" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your first milestone, describe what your project is and how you plan to build it. You can include:
- Currently, the project uses an ELEGOO UNO R3 Arduino, a breadboard, several jumper wires, a data-capable USB cable, an 8x32 MAX7219 Dot Matrix, and an LM393 Sound Detection Sensor Module.
- I've completed my base project; the pieces are all put together, and the code makes it run as intended.
- Challenges: My main challenge was adapting the tutorial code to work with the setup I had, as the tutorial code used features of older library versions on Arduino IDE, so despite using the same libraries, different versions called for adjustments.
  Another struggle I had was optimizing sensitivity: when it was too low, I wouldn't get the animation effect I wanted on the display; however, if it were too high, visuals would be displayed on the matrix even if no sound was being played. So to fix this, I added a sound floor, which        essentially made it so that if a sound registered as too quiet, the program would treat it as silence.
- Next, I plan on replacing the current display with a new one that is larger and has a wider selection of colors.
  On top of that, I intend to negate the need for a sound sensor so that I can just plug the Arduino into any device I wish, and it will display the animation based on the sound coming directly from the device.

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++
/*
  DIY MUSIC / AUDIO VISUALIZER
  =============================
  Rebuilt from the CircuitDigest Arduino Nano project so it works with:
    - ELEGOO UNO R3 (Arduino Uno)
    - 32x8 dot matrix display (four cascaded MAX7219 8x8 modules)
    - KY-038 / FC-04 style sound sensor module (AO pin)
  ...and with the CURRENT versions of:
    - arduinoFFT   v2.x, by Enrique Condes  - Library Manager: "arduinoFFT"
    - MD_MAX72XX   by MajicDesigns          - Library Manager: "MD_MAX72XX"

  WIRING
  ------
  Dot matrix -> Arduino Uno        Sound sensor -> Arduino Uno
    VCC -> 5V                        +  -> 5V
    GND -> GND                       G  -> GND
    DIN -> D11                       AO -> A0
    CS  -> D10                       DO -> not used here
    CLK -> D13

  On upload, all 32 columns flash on for half a second - that's a
  wiring self-test for the display, independent of the microphone.

  If that flash looks scrambled, mirrored, or only partly lights up,
  your matrix modules use different internal wiring than assumed.
  Change HARDWARE_TYPE below to one of MD_MAX72XX::FC16_HW / GENERIC_HW
  / PAROLA_HW / ICSTATION_HW and re-upload until the self-test flash
  fills the whole display cleanly. This mismatch is the single most
  common reason these cheap 4-in-1 matrix boards look glitchy - the
  internal wiring varies by manufacturer even when the boards look
  identical.
*/

#include <SPI.h>
#include <MD_MAX72xx.h>
#include <arduinoFFT.h>

// ---------------- Dot matrix display ----------------
#define HARDWARE_TYPE MD_MAX72XX::FC16_HW   // try GENERIC_HW / PAROLA_HW / ICSTATION_HW if the self-test looks wrong
#define MAX_DEVICES   4
#define CLK_PIN       13
#define DATA_PIN      11
#define CS_PIN        10

MD_MAX72XX mx = MD_MAX72XX(HARDWARE_TYPE, CS_PIN, MAX_DEVICES);

// ---------------- Sound sensor ----------------
const int MIC_PIN = A0;

// ---------------- FFT settings ----------------
const uint16_t SAMPLES            = 64;      // must be a power of 2; also 2x the number of display columns
const double   SAMPLING_FREQUENCY = 9000.0;  // Hz, approx. max for a stock Uno's analogRead() - fine for a visual effect

double vReal[SAMPLES];
double vImag[SAMPLES];
ArduinoFFT<double> FFT = ArduinoFFT<double>(vReal, vImag, SAMPLES, SAMPLING_FREQUENCY);

// Stand-in for the pot-based "sensitivity" the original Nano code read
// from a second analog pin - this sensor module doesn't expose one, so
//It's a fixed value you tune by hand instead. Raise it if the bars are
//always maxed out; lower it if they barely move. See DEBUG below.
double SENSITIVITY = 3.3;

// NOISE GATE: bars below this magnitude are treated as silence, so
// residual noise can't keep columns lit when nothing is actually
// playing. Raise this if the display still glows/lingers at rest;
// lower it if quiet sounds stop registering.
const long NOISE_FLOOR = 8;

// How many LEDs (bottom to top) are lit for a given bar height, 0-8.
const byte spectralHeight[] = {
  0b00000000, 0b10000000, 0b11000000, 0b11100000,
  0b11110000, 0b11111000, 0b11111100, 0b11111110, 0b11111111
};

// Set true to print the loudest bin's magnitude a few times a second -
// useful while tuning SENSITIVITY and the constrain() range below.
const bool DEBUG = false;

void setup() {
  Serial.begin(115200);
  mx.begin();
  mx.control(MD_MAX72XX::INTENSITY, 8); // brightness 0 (dim) - 15 (bright)

  // Display self-test: full-brightness flash, unrelated to audio.
  mx.clear();
  for (uint16_t c = 0; c < mx.getColumnCount(); c++) mx.setColumn(c, 0xFF);
  delay(500);
  mx.clear();
}

void loop() {
  sampleAudio();

  FFT.windowing(FFTWindow::Hamming, FFTDirection::Forward);
  FFT.compute(FFTDirection::Forward);
  FFT.complexToMagnitude();

  if (DEBUG) printDebug();

  drawSpectrum();
}

void sampleAudio() {
  // Sampled back-to-back as fast as analogRead() allows. Exact timing
  // isn't critical for a visual effect, just reasonably consistent.
  int raw[SAMPLES];
  long total = 0;

  for (uint16_t i = 0; i < SAMPLES; i++) {
    raw[i] = analogRead(MIC_PIN);
    total += raw[i];
  }

  double mean = total / (double)SAMPLES; // remove DC offset so silence sits near 0
  for (uint16_t i = 0; i < SAMPLES; i++) {
    vReal[i] = (raw[i] - mean) / SENSITIVITY;
    vImag[i] = 0.0;
  }
}

void drawSpectrum() {
  mx.control(MD_MAX72XX::UPDATE, MD_MAX72XX::OFF);
  for (uint16_t i = 0; i < 32; i++) {
    long magnitude = (long)constrain(vReal[i], 0.0, 80.0);

    if (magnitude < NOISE_FLOOR) {
      magnitude = 0; // treat anything below the floor as silence
    }

    int      rowsLit = map(magnitude, 0, 80, 0, 8);
    uint16_t column  = 31 - i; // low frequencies on the right; drop the "31 -" to mirror it
    mx.setColumn(column, spectralHeight[rowsLit]);
  }
  mx.control(MD_MAX72XX::UPDATE, MD_MAX72XX::ON);
}

void printDebug() {
  static unsigned long lastPrint = 0;
  if (millis() - lastPrint < 300) return;
  lastPrint = millis();

  double peak = 0;
  for (uint16_t i = 1; i < 32; i++) peak = max(peak, vReal[i]);
  Serial.println(peak);
}
```

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
