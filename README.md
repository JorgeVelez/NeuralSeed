# NeuralSeed

Neural Seed uses neural networks to emulate amps/pedals on the Daisy Seed hardware by [Electro-Smith](https://www.electro-smith.com/), and the Terrarium
pedal by [PedalPCB](https://www.pedalpcb.com/product/pcb351/). Models are trained on audio recordings from your amp or pedal, and can be uploaded to the Daisy Seed
as firmware. Effective for Amps/PreAmps (direct out, no cab), Distortion/Overdrive/Boost pedals (non-time based, no Reverb/Delay/Flange/Phaser).

![app](https://github.com/GuitarML/NeuralSeed/blob/main/neuralseed.jpg)

The Daisy Seed is a powerful ARM Cortex-M7 powered board intended for audio effects. The Terrarium pcb provides input and output buffers needed 
for a digital guitar effect pedal. NeuralSeed is the software developed by GuitarML to run on this hardware. Total material cost was less than $100
to build a fully capable digital guitar multi-effect.

## Technical Info
In comparison to other GuitarML plugins, Neural Seed is very minimal, running only a LSTM size 7 (by comparison
[Proteus](https://github.com/GuitarML/Proteus) uses LSTM size 40). Using this size model, may not be able to accurately capture certain devices, especially
high gain amps, but should work decently well for distortion/overdrive pedals and low - medium gain amps (direct out, not
from a microphone). This is due to limited processing power on the M7 microcontroller. The current code processes on
float32 audio data, so this could be optimized for the M7 chip using quantized int16 data. 

The compiled binary and model data fits into Flash Memory, which is limited to 128KB. The [RTNeural](https://github.com/jatinchowdhury18/RTNeural)
engine is used for fast inferencing of the neural models with a very tiny footprint.  It is possible to add more models utilizing other data storage 
areas on the Daisy Seed.

Audio processing uses 48kHz, 24-bit, mono.

## Getting started
Build the daisy libraries with:
```
make -C DaisySP
make -C libDaisy
```

Then flash your terrarium with the following commands (or use the [Electrosmith Web Programmer](https://electro-smith.github.io/Programmer/))
```
cd your_pedal
# using USB (after entering bootloader mode)
make program-dfu
# using JTAG/SWD adaptor (like STLink)
make program
```

# Control

| Control | Description | Comment |
| --- | --- | --- |
| Ctrl 1 | Input Level | Adjusts the input level to the neural net model (0 to 3x volume) |
| Ctrl 2 | Mix | Set the dry/wet amount (full left for dry, full right for only neural net output |
| Ctrl 3 | Volume | Sets the output volume of the pedal |
| Ctrl 4 | Neural Param 1 | Adjusts the first model parameter, such as gain/tone (only applies to conditioned models) |
| Ctrl 5 | Neural Param 2 | Adjusts the second model parameter, such as gain/tone (only applies to conditioned models)  |
| Ctrl 6 | N/A | Reserved for future 3-Parameter models |
| SW 1 - 4 | 4Band EQ Boost | Bass, Low Mid, High Mid, Treble Boost (Band Pass) |
| FS 1 | Bypass/Active | Bypass / effect engaged |
| FS 2 | Cycle Neural Model | Loads the next available Neural Model, starts at beginning after the last in the list (i.e. next pedal/amp) |
| LED 1 | Bypass/Active Indicator |Illuminated when effect is set to Active |
| LED 2 | Brightness based on selected model parameterization | Off = Snapshot, Dim = 1 Knob, Bright = 2 Knobs |
| Audio In 1 | Audio input | Mono only for Terrarium |
| Audio Out 1 | Mix Out | Mono only for Terrarium |

## Training your own Models (Amp/Pedal Capture)

You can train models for Neural Seed using the GuitarML fork of [Automated-GuitarAmpModelling](https://github.com/GuitarML/Automated-GuitarAmpModelling) code.
Run "scripts/convert_json_to_c_header.py" on the resulting JSON model to generate a .h file, which you can copy and paste the weights into the "all_model_data.h" header before
compiling "neuralseed.bin". Only LSTM size 7 models (snapshot, 1-param, or 2-param) models are currently compatible. Train models using 48kHz to match Daisy Seed processing 
(note that this is different from most other GuitarML plugins). A 48kHz input wav file based on the Proteus input wav is available [here]().
