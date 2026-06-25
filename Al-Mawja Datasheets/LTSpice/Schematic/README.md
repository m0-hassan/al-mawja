# AL-MAWJA ADC Driver Simulation
## THS4551 → AD7380 Signal Chain Validation

## Files
- `almawja_adc_driver.cir` — Complete SPICE netlist (open in LTspice)
- `ths4551.lib` — THS4551 SPICE model (must be in same folder)
- `THS4551.asy` — LTspice symbol for THS4551 (optional, for .asc schematics)

## Setup
1. Put ALL three files in the same folder
2. Open `almawja_adc_driver.cir` in LTspice (File → Open)
3. LTspice will display it as a netlist — that's normal for .cir files

## Running the Four Simulations

### Sim 1: DC Operating Point
- Default config (already uncommented)
- Change V_IN line to: `V_IN SIG_IN 0 1.25` (DC only, no sine)
- Run simulation (click the running-man icon)
- Check: OUTP and OUTN should both read ~1.25V
- If yes: VOCM biasing is correct

### Sim 2: AC Frequency Sweep
- Comment out `.op` line (add * at start)
- Change V_IN line to: `V_IN SIG_IN 0 AC 0.5`
- Uncomment `.ac dec 200 100 100Meg`
- Run simulation
- Right-click the schematic → Add Trace → type: `V(OUTP)-V(OUTN)`
- Check: flat response to ~1 MHz, then rolling off
- The -3dB point is your filter cutoff frequency
- If cutoff too low: decrease Cf_top and Cf_bot
- If cutoff too high: increase Cf_top and Cf_bot

### Sim 3: Transient — Sine Wave
- Comment out `.ac` line
- Change V_IN back to: `V_IN SIG_IN 0 SINE(1.25 0.5 500k)`
- Uncomment `.tran 0 20u 0 10n`
- Run simulation
- Plot V(OUTP) and V(OUTN) — verify no flat-topping (clipping)
- Plot V(OUTP)-V(OUTN) — verify clean sine wave
- Check: OUTP stays above 0.1V, below 3.2V

### Sim 4: Transient — Step Response (Settling)
- Change V_IN to: `V_IN SIG_IN 0 PULSE(0.75 1.75 1u 10n 10n 5u 10u)`
- Keep `.tran` command (or use `.tran 0 4u 0 1n`)
- Run simulation
- Plot V(SAMP_P) — this is the voltage on the ADC sampling capacitor
- Check: signal settles to within 0.001% of final value within 250ns
- If ringing/overshoot: increase Cf or Cdiff

## Component Values to Adjust
| Component | Starting Value | What it controls |
|-----------|---------------|-----------------|
| Cf_top, Cf_bot | 150 pF | Filter cutoff frequency |
| Cdiff | 220 pF | Filter Q / peaking |
| Rg_sig, Rg_bias | 1 kΩ | Input impedance + gain |
| Rf_top, Rf_bot | 1 kΩ | Gain (Rf/Rg = 1 for unity) |
| R_out_p/n | 33 Ω | FIXED — AD7380 datasheet |
| C_cm_p/n | 68 pF | FIXED — AD7380 datasheet |
| C_diff_adc | 68 pF | FIXED — AD7380 datasheet |
| R_sw_p/n | 200 Ω | FIXED — AD7380 Figure 28 |
| C_samp_p/n | 15 pF | FIXED — AD7380 Figure 28 |

## After Simulation Passes
Write down your final Cf and Cdiff values.
Update your KiCad schematic with these verified values.
The output RC network and ADC equivalent circuit values never change.
