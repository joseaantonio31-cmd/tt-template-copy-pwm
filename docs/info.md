## How it works

This project is a 4-bit PWM (pulse-width modulation) generator built from discrete logic gates and D flip-flops.

**Counter.** A 4-bit synchronous binary counter (Q3..Q0) increments on every rising edge of `clk` and wraps from 15 back to 0, so the PWM period is 16 clock cycles. The next-state logic is:

- D0 = NOT Q0
- D1 = Q1 XOR Q0
- D2 = Q2 XOR (Q1 AND Q0)
- D3 = Q3 XOR (Q2 AND Q1 AND Q0)

All flip-flops share the same clock. `rst_n` (active low) is inverted and drives the active-high reset (R) of every flip-flop, which clears the counter to 0. The set (S) inputs are tied to ground.

**Magnitude comparator.** The duty-cycle setting D = `ui_in[3:0]` (ui_in[0] = LSB) is compared with the counter value Q, and the output is high while Q < D. For each bit:

- g_i = (NOT Q_i) AND D_i (D is greater at bit i)
- e_i = (NOT Q_i) OR D_i (Q is not greater at bit i)

These are combined MSB-first as:

PWM = g3 + e3·(g2 + e2·(g1 + e1·g0))

The result is a PWM signal with a duty cycle of D/16, adjustable in steps of 6.25 %, from 0 % (D = 0) to 93.75 % (D = 15). The PWM frequency is f_clk / 16.

**Outputs**

| Pin        | Signal                 |
|------------|------------------------|
| uo_out[0]  | PWM output             |
| uo_out[1]  | Counter bit Q0 (LSB)   |
| uo_out[2]  | Counter bit Q1         |
| uo_out[3]  | Counter bit Q2         |
| uo_out[4]  | Counter bit Q3 (MSB)   |
| uo_out[7:5]| Unused                 |

**Inputs**

| Pin        | Signal                            |
|------------|-----------------------------------|
| ui_in[3:0] | Duty-cycle setting D (0–15)       |
| ui_in[7:4] | Unused                            |
| clk        | Counter clock                     |
| rst_n      | Active-low reset (clears counter) |

## How to test

1. Assert `rst_n` low briefly, then release it. The counter starts from 0.
2. Set the duty cycle with `ui_in[3:0]` (DIP switches 1–4, switch 1 = LSB).
3. **Manual stepping:** clock the design one cycle at a time. Q3..Q0 on `uo_out[4:1]` count 0, 1, 2 … 15, 0 … `uo_out[0]` is high for exactly D of every 16 clock cycles.
4. **Free-running:** apply a continuous clock and observe `uo_out[0]` with an oscilloscope or logic analyzer. Verify that:
   - the period is 16 / f_clk;
   - the high time is D / f_clk;
   - D = 0 gives a constant low output, D = 8 gives a 50 % duty cycle and D = 15 gives 93.75 %.

The counter outputs `uo_out[4:1]` can be used as a trigger or reference to check that the PWM edge falls exactly at count D.

## External hardware

None required. An LED on `uo_out[0]` shows the PWM brightness (use a sufficiently high clock to avoid visible flicker). An oscilloscope or logic analyzer is recommended to measure the duty cycle.
