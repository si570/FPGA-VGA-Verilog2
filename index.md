---
layout: home
title: FPGA VGA Driver Project – Scary Maze
tags: fpga vga verilog game
categories: demo
---

Hi! Have you ever wondered how LCD screens manage to display millions of pixels every second?  
Today you're in luck — this page explains my FPGA VGA project, where I recreated the classic **Scary Maze Game** using **Verilog** and an **Artix-7** FPGA.

My VGA project started at **16:54 on 20/11/25**.

---

## **Template VGA Design**

### **Project Set-Up**
This project uses the **Artix-7 Basys3 FPGA**.  
All graphics are programmed in **Verilog**, and the VGA port sends out pixel colors according to horizontal and vertical counters.

<img src="https://raw.githubusercontent.com/melgineer/fpga-vga-verilog/main/docs/assets/images/VGAPrjSum.png">

<img src="https://github.com/si570/FPGA-VGA-Verilog2/blob/main/docs/assets/images/Code1.png">

### **Template Code**
The template provided the core VGA timing module.  
It generates:
- **hsync** and **vsync** pulses  
- A **pixel clock**  
- Horizontal (`h_count`) and vertical (`v_count`) counters  
- A **video_on** signal, so pixels are only drawn inside the visible region

This works because VGA requires *precise timing*:  
640×480 resolution at 60 Hz uses a 25.175 MHz pixel clock, with front porch, sync pulse, and back porch intervals.  
The template code handled all of this, allowing me to focus on adding custom graphics.

### **Simulation**
I simulated the VGA timing using Vivado’s waveform viewer.  
The simulation confirmed:
- Correct hsync/vsync timing  
- Counters reset at the correct cycle  
- `video_on` activates within 640×480 only

<img src="https://github.com/si570/FPGA-VGA-Verilog2/blob/main/docs/assets/images/Code2.png">

### **Synthesis**
Vivado synthesis successfully mapped the VGA drivers onto the Artix-7 logic.  
No critical warnings were produced, and the timing report passed with positive slack.

### **Demonstration**
Here you can add a photo of the VGA display running the template pattern.

---

# **My VGA Design Edit — Scary Maze Game**

For my custom design, I recreated a simplified **Scary Maze Game**.  
The idea:  
- The player moves a small **cursor square** through a narrow **maze path**  
- If they touch the blue walls → **jump scare appears**  
- The scary face is stored in a **ROM pixel image file**

I researched VGA image ROM techniques and pixel storage from sites like:  
- https://projectf.io  
- https://fpga4fun.com  
- https://nandland.com

---

## **Code Adaptation**

I modified the template to include:

### 1. **Maze drawing logic**  
I used coordinate checks:

```verilog
// Maze walls (blue)
if ((x < 100) || (x > 540) || (y < 50) || (y > 430))
    rgb <= 12'h00F;   // blue
else if (hit_wall)
    rgb <= scary_pixel;  // scary face ROM
else
    rgb <= 12'hFFF;  // white path

