# Final Project – LaunchPad Paint

- [Overview](#overview)
- [Project Objectives](#project-objectives)
- [Requirements and Dependencies](#requirements-and-dependencies)
  - [Hardware Requirements](#hardware-requirements)
  - [Software Requirements](#software-requirements)
- [System Architecture](#system-architecture)
- [Functional Specification](#functional-specification)
- [Part I: Drawing Implementation](#part-i--drawing-implementation)
- [Part II: User Interface](#part-ii--user-interface)
- [Part III: Save Image to Cloud Storage](#part-iii--save-image-to-cloud-storage)
- [Video Demonstration](#video-demonstration)
- [Reproducibility](#reproducibility)
- [Future Work](#future-work)
- [Bill of Materials](#bill-of-materials)

---

# Overview

This project implements a remote-controlled drawing system using the CC3200 microcontroller, an OLED display, an IR receiver, and AWS cloud services. The system allows a user to draw on a digital canvas using a standard TV remote and view the drawing in real time on the OLED display. Once the drawing is complete, the image can be uploaded to cloud storage and sent to the user via email.

The project demonstrates a complete IoT workflow that integrates embedded hardware interaction, graphical rendering, wireless networking, and cloud services.

---

# Project Objectives

The goals of this project are:

- Implement a remote-controlled drawing interface
- Display drawing output on an OLED screen
- Support multiple drawing tools (pixel, line, rectangle, ellipse, fill)
- Upload saved drawings to cloud storage
- Send saved drawings to a user through email

---


# Requirements and Dependencies

## Hardware Requirements

- SimpleLink CC3200 LaunchPad
- OLED display (SPI interface)
- IR receiver module
- TV remote controller
- Breadboard and jumper wires
- Micro-USB cable

## Software Requirements

- Code Composer Studio (CCS)
- CC3200 SDK
- AWS IoT Services
- AWS Lambda
- Amazon S3
- Amazon SES

---

# System Architecture
The system consists of three major components: the embedded drawing device, cloud services, and the user interface.

1. The CC3200 connects to Wi-Fi and initializes the drawing system.
2. The IR receiver detects signals from the TV remote.
3. The CC3200 decodes the input and performs drawing operations on a framebuffer.
4. The framebuffer is rendered on the OLED display.
5. When the user saves the drawing, the device uploads the image to AWS S3.
6. A Lambda function processes the upload and sends the image to the user via email.

![System Architecture Diagram](./system_architecture_diagram_final.png "a title")
---
# Functional Specification

The functional specification describes the workflow of the drawing system, including initialization, user interaction, drawing operations, and cloud image upload.

1. When the program starts, the CC3200 initializes board configurations, GPIO pins, SPI, OLED display, SysTick, and UART.
2. The system creates the canvas buffer, renders the UI, draws the initial canvas, and connects to Wi-Fi.
3. The program enters the main loop and continuously waits for input from the IR remote controller.
4. When a button is pressed, the IR signal is decoded and the system determines which drawing feature to execute.
5. The system updates the cursor position, drawing tool, color, or canvas state, and then re-renders the canvas and cursor on the OLED display.
6. If the save function is triggered, the canvas buffer is converted to image data and uploaded to cloud storage using a pre-signed URL obtained from AWS.
7. 
![Functional Specifiication Diagram](./functional_specification_diagram_final.png "a title")
---



# Part I: Drawing Implementation

The drawing system maintains a framebuffer representing the canvas. Each pixel is stored in RGB565 format and rendered to the OLED display.

Supported drawing operations include:

- Cursor movement
- Pixel drawing
- Line drawing
- Rectangle drawing
- Ellipse drawing
- Bucket fill

Each drawing operation updates the framebuffer and then refreshes the OLED display.




# Part II: User Interface

The system uses a TV remote as the primary input device.

Example button mappings:

| Button | Function |
|------|------|
| Vol Up / Down | Change drawing tool |
| Ch Up / Down | Change tool mode |
| Mute | Change color |
| Last | Confirm operation |
| 2 | Move cursor up |
| 4 | Move cursor left |
| 6 | Move cursor right |
| 8 | Move cursor down |
| 5 | Apply drawing action |
| 0 | Clear canvas |

---

# Part III: Save Image to Cloud Storage

When the user chooses to save a drawing:

1. The CC3200 sends a request to AWS API Gateway.
2. A Lambda function generates a **pre-signed URL**.
3. The device converts the framebuffer into a BMP image.
4. The image is uploaded to Amazon S3 using the pre-signed URL.
5. An S3 event triggers another Lambda function.
6. The Lambda function sends the saved image to the user via email using Amazon SES.

---

# Video Demonstration

The following video demonstrates the major features of the system:

- Cursor movement using the remote
- Drawing pixels and shapes
- Changing colors
- Clearing the canvas
- Saving the drawing to the cloud
- Receiving the drawing via email

<video src="./launchpad-paint_demo_faster.mp4" width="490" height="270" controls></video>

---

# Reproducibility

To reproduce this project:

1. Connect the hardware components according to the wiring diagram.
2. Install CCS and the CC3200 SDK.
3. Configure the Wi-Fi credentials in the project source code.
4. Set up AWS services including S3, Lambda, and API Gateway.
5. Build and flash the program onto the CC3200 LaunchPad.
6. Power the system and begin drawing using the remote.

---

# Future Work

Several improvements could be made in future versions of the project:

- Implement a gyroscope-based drawing tool
- Add a polygon drawing tool
- Support text input with an on-screen preview
- Allow switching between multiple Wi-Fi networks
- Add user authentication for cloud uploads
- Support multiple frames for animation creation

---

# Bill of Materials

| Component | Purpose |
|------|------|
| CC3200 Microcontroller | Core processing and Wi-Fi connectivity |
| SPI OLED Display | Display drawing canvas |
| IR Receiver | Receive remote control signals |
| AT&T S10-S3 Remote | User input device |
| AWS IoT Services | Cloud communication |
| Amazon S3 | Image storage |
| AWS Lambda | Cloud processing |
| Amazon SES | Email notification service |

---
