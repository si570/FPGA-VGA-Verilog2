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
<img src="https://github.com/si570/FPGA-VGA-Verilog2/blob/main/docs/assets/images/scarymaze.png">

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

module DisplaySI_4bit (
    input clk, rst,
    input [10:0] row, col,              // Current VGA pixel position
    output [3:0] red, green, blue       // 4-bit VGA color channels
);

reg [3:0] red_reg, green_reg, blue_reg; // Registered RGB outputs
reg [3:0] pixel;                        // 4-bit pixel value (0 = black, F = white)


// -----------------------------------------------------------------------------
// 20×20 4-bit bitmap array for letter S
// Each entry is a pixel intensity (4'h0 to 4'hF)
// -----------------------------------------------------------------------------
reg [3:0] S_map [0:19][0:19];

integer i, j;

// -----------------------------------------------------------------------------
// Initialize the "S" bitmap
// -----------------------------------------------------------------------------
initial begin
    // Fill entire 20×20 area with black pixels (4'h0)
    for (i=0; i<20; i=i+1)
        for (j=0; j<20; j=j+1)
            S_map[i][j] = 4'h0;

    // Draw the top, middle, and bottom horizontal white lines
    for (j=0; j<20; j=j+1) begin
        S_map[0][j]  = 4'hF;   // Top bar
        S_map[9][j]  = 4'hF;   // Middle bar
        S_map[19][j] = 4'hF;   // Bottom bar
    end

    // Upper-left vertical line for S
    for (i=1; i<9; i=i+1)
        S_map[i][0] = 4'hF;

    // Lower-right vertical line for S
    for (i=10; i<19; i=i+1)
        S_map[i][19] = 4'hF;
end



// -----------------------------------------------------------------------------
// 20×20 4-bit bitmap array for letter I
// -----------------------------------------------------------------------------
reg [3:0] I_map [0:19][0:19];

initial begin
    // Fill all with black
    for (i=0; i<20; i=i+1)
        for (j=0; j<20; j=j+1)
            I_map[i][j] = 4'h0;

    // Top bar of "I"
    for (j=0; j<20; j=j+1)
        I_map[0][j] = 4'hF;

    // Vertical bar down the center of "I"
    for (i=1; i<19; i=i+1)
        I_map[i][9] = 4'hF;

    // Bottom bar of "I"
    for (j=0; j<20; j=j+1)
        I_map[19][j] = 4'hF;
end



// -----------------------------------------------------------------------------
// Pixel selection logic
// Chooses which letter to draw based on VGA (row, col) position
// -----------------------------------------------------------------------------
always @* begin
    pixel = 4'h0;  // Default background = black

    // -----------------------------------------
    // Draw letter "S"
    // Displayed at: rows 100–119, cols 50–69
    // -----------------------------------------
    if (row >= 100 && row < 120 &&
        col >= 50  && col < 70) begin
        
        // Map VGA coordinates → bitmap index
        pixel = S_map[row-100][col-50];
    end

    // -----------------------------------------
    // Draw letter "I"
    // Displayed at: rows 100–119, cols 100–119
    // -----------------------------------------
    else if (row >= 100 && row < 120 &&
             col >= 100 && col < 120) begin

        pixel = I_map[row-100][col-100];
    end

    // -----------------------------------------
    // Assign grayscale pixel value to RGB
    // white pixel = FFF, black pixel = 000
    // -----------------------------------------
    red_reg   = pixel;
    green_reg = pixel;
    blue_reg  = pixel;
end


// -----------------------------------------------------------------------------
// Output the RGB values to VGA
// -----------------------------------------------------------------------------
assign red   = red_reg;
assign green = green_reg;
assign blue  = blue_reg;

endmodule

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
