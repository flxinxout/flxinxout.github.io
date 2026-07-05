---
layout: page
title: Master Semester Project
description: Enabling Joint Communication and Sensing Through RFSoC-Controlled Phased Array Waveforms
img: assets/img/sens_of_doom.png
importance: 1
category: work
related_publications: false
---

Full Report: [HERE](/assets/pdf/Master_Project_JCAS_Phased_Array.pdf)
SENS of Doom PCB Documentation: [HERE](/assets/pdf/SENS_of_Doom_PCB_Documentation.pdf)

This project was carried out in the Sensing and Networking Laboratory (SENS) at EPFL, under the supervision of Samah Hussein and Prof. Haitham Al Hassanieh. The main idea was to enable **Joint Communication and Sensing (JCAS)** on a real phased array, a key enabler of future 6G systems where the same RF hardware and waveform are used both to transmit data and to sense the environment. Concretely, I connected a Sivers EVK06005 phased array (57–71 GHz, 16 TX/RX beamforming elements) to an AMD/Xilinx ZCU208 RFSoC evaluation board, and integrated everything into HELIX, a 6G real-time experimentation platform. The two goals were (i) fast beam-switching between OFDM symbols, and (ii) reproducing the overlapped radar-communication waveform described in ["Orthogonal Coexistence of Overlapped Radar and Communication Waveforms"](https://ieeexplore.ieee.org/document/9771679).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid
            loading="eager"
            path="assets/img/zcu208.png"
            title="ZCU208 Evaluation Board"
            class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid
            loading="eager"
            path="assets/img/sivers_evk.png"
            title="Sivers EVK06005 phased array"
            class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The two hardware modules bridged in this project: the AMD/Xilinx ZCU208 RFSoC evaluation board (left), and the Sivers EVK06005 phased array mounted on its beamforming RF module (right). One of the first goals was simply to physically connect them so that I/Q samples and BF/SPI signals could flow between the two.
</div>

Beamforming (BF) is the core signal-processing technique here: a phased array is a collection of antenna elements, and by applying a set of weights (a *steering vector*) to each element you can shape, steer or null the beam. The interesting part for JCAS is that the optimal beam can change per OFDM symbol. If you can switch beam patterns fast enough, you can interleave a few probing symbols (to test other beams) with many data symbols on the best known beam, minimizing throughput loss while still sensing the channel. The Sivers EVK06005 provides a dedicated BF interface for fast weight-switching, which combined with the ZCU208 makes a very capable JCAS testbed.

<h1>SENS of Doom: a custom breakout PCB</h1>

The first big challenge was purely physical. The ZCU208 was never designed to be wired to the EVK06005. The phased array exposes its SPI and BF interfaces on two small 8-pin Molex connectors, while the ZCU208 only exits through its FMC+ HSPC connector, a 560-pin Samtec connector following the VITA 57.4 standard, with no breakout to any usable pin header. Soldering directly on the FMC+ pins was too risky (the pins are extremely dense and the board costs 16k CHF), and no commercial breakout matched the phased array's pinout. So I designed my own breakout board, **SENS of Doom**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid
            loading="eager"
            path="assets/img/sens_of_doom.png"
            title="SENS of Doom front layer"
            class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid
            loading="eager"
            path="assets/img/sens_of_doom_back.png"
            title="SENS of Doom back layer"
            class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Render of the front (left) and back (right) layers of SENS of Doom. The board mates with the ZCU208's FMC+ HPC connector and breaks out the FPGA-side I/O: three 8-pin Molex headers mirroring the EVK pinout for SPI, BF and AGC, a 2 kB I2C EEPROM (M24C02) storing the VITA 57.4 FRU descriptor, an Si570 I2C-programmable low-jitter clock, and eight GTY high-speed transceivers routed to 50 Ω SMA connectors with 100 Ω differential impedance.
</div>

Designing SENS of Doom was a deep dive into the VITA 57.4 standard and its quirks. One subtle but critical detail is the **VADJ_FMC power rail**: following the standard, any mezzanine must carry a small EEPROM describing its configurable voltage rails. The ZCU208 System Controller reads this descriptor and powers VADJ_FMC accordingly, and PL Banks 64 and 67 (which drive most FMC+ signals) depend on this rail. Without a valid EEPROM, VADJ_FMC stays at 0 V and no signal can leave the board. Embedding a compliant FRU EEPROM was therefore not optional, it was the difference between a working board and a dead one. I want to thank Dr. Philipp Födisch from IAMELECTRONICS for helping me navigate this (closed-source) standard and for reviewing the PCB.

<h1>Interfacing the phased array</h1>

Once the hardware bridge existed, I implemented three things to actually drive the EVK06005 from the ZCU208.

**A bare-metal SPI driver.** All registers of the phased array are programmed over a 4-wire SPI interface, where each transaction starts with a 16-bit header (13-bit address + 3-bit command) and auto-increments the address for multi-byte transfers. I built the driver on top of Xilinx's `XSpiPs` bare-metal driver, using a built-in SPI controller from PL Bank 67 routed through EMIO, because configuring registers is not time-sensitive and no SPI is needed directly in the PL.

**A beamforming IP.** The BF interface uses three signals (`BF_INC`, `BF_RST`, `BF_RTN`) to walk through the Antenna Weight Vector (AWV) tables that store beam configurations. Rather than toggling these pins from software, I wrote a command-driven HDL module: the higher-level IP issues single-cycle *triggers* (reset, return-to-home, N-step increment, autonomous sweep) and the module generates the correctly-timed pulses on its own, raising a `busy` flag and emitting a `done` pulse on completion.

<div class="row justify-content-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid
            loading="eager"
            path="assets/img/bf_fsm.png"
            title="Beamforming control FSM"
            class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    State machine of the beamforming control module. The phased array imposes a minimum 5 ns pulse width, but HELIX's PL runs at 245.76 MHz (≈ 4.08 ns per cycle), so pulses cannot live on a single clock cycle. The FSM widens each pulse to two cycles to respect this hardware constraint, and cleanly latches a return-to-home request so it never corrupts an ongoing sweep.
</div>

**JCAS IPs.** Finally, to reproduce the overlapped radar-communication waveform, I built two custom IPs, a *chirp adder* and a *chirp remover*, that inject and extract a sensing chirp directly in the HELIX OFDM pipeline. The sensing waveform is a chirp over a finite time, pre-computed and packed into the PL, and centered around DC to avoid aliasing in the complex baseband. Both IPs use a counter synchronized to the existing `triggerOut` slot marker so the chirp lines up with the OFDM symbols.

<h2>Honest results and limitations</h2>

This was a project at the frontier of hardware/software co-design, and I think it's more useful to be honest about what didn't fully work than to pretend everything was perfect. The project leaves behind a working hardware/software base and a documented codebase, but a few aspects did not reach completion:

<table style="width:100%; border-collapse: collapse; font-family: sans-serif;">
  <thead>
    <tr style="background-color: #004080; color: white;">
      <th style="padding: 10px 8px; text-align: left;">Aspect</th>
      <th style="padding: 10px 8px; text-align: left;">What happened</th>
    </tr>
  </thead>
  <tbody>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong style="color: #000000;">SPI MISO path</strong></td>
      <td style="padding: 8px; color: #000000;">
        The SPI return path could not be validated: the ZCU208 returned inconsistent register reads, and a fallback attempt on an RFSoC 4x2 showed MISO stuck high. Only the MOSI side was confirmed via ILA.
      </td>
    </tr>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong style="color: #000000;">FRU / EEPROM</strong></td>
      <td style="padding: 8px; color: #000000;">
        Because VITA 57.4 is closed-source, the FRU descriptor written to the mezzanine EEPROM could not be checked against the spec, and intermittent VADJ_FMC deactivations suggest it is not yet fully compliant.
      </td>
    </tr>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong style="color: #000000;">Dual-stream HELIX</strong></td>
      <td style="padding: 8px; color: #000000;">
        With only one ZCU208, I tried to modify the hardware design to include a second phased array on the board. After roughly a month, the lack of documentation and the difficulty of modifying a custom Verilog UDP stack forced me to abandon this idea.
      </td>
    </tr>
    <tr style="background-color: #f2f6fc;">
      <td style="padding: 8px;"><strong style="color: #000000;">Chirp sensing bins</strong></td>
      <td style="padding: 8px; color: #000000;">
        The constraint that the chirp length must divide gcd(N, CP1, CP2) = 16 forces the chirp to repeat at least 128 times per OFDM symbol, leaving substantially fewer sensing bins than the reference paper assumes.
      </td>
    </tr>
  </tbody>
</table>

The most natural next steps are to probe the I2C bus to validate the FRU data, debug the SPI return path on the EVK06005, and extend the JCAS IPs with runtime-configurable chirp parameters through a DDS compiler. A second ZCU208 would then finally enable the full bistatic measurements that were originally envisioned. Beyond the results, this project gave me hands-on experience with the full Vivado and Vitis flow and with hardware/software co-design for communication systems, from the block design down to a bare-metal application running on FreeRTOS. You can read the [full report](/assets/pdf/Master_Project_JCAS_Phased_Array.pdf) and the [SENS of Doom PCB documentation](/assets/pdf/SENS_of_Doom_PCB_Documentation.pdf) for all the details.
