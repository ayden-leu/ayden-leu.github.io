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

<img src="./system_architecture_diagram_final.png" width="60%" alt="System Architecture Diagram">

# Functional Specification

The functional specification describes the workflow of the drawing system, including initialization, user interaction, drawing operations, and cloud image upload.

1. When the program starts, the CC3200 initializes board configurations, GPIO pins, SPI, OLED display, SysTick, and UART.
2. The system creates the canvas buffer, renders the UI, draws the initial canvas, and connects to Wi-Fi.
3. The program enters the main loop and continuously waits for input from the IR remote controller.
4. When a button is pressed, the IR signal is decoded and the system determines which drawing feature to execute.
5. The system updates the cursor position, drawing tool, color, or canvas state, and then re-renders the canvas and cursor on the OLED display.
6. If the save function is triggered, the canvas buffer is converted to image data and uploaded to cloud storage using a pre-signed URL obtained from AWS.
7. 

<img src="./functional_specification_diagram_final.png" width="70%" alt="System Architecture Diagram">

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

This system allows the user to save the generated drawing to the cloud and automatically receive the image through email. The process integrates the CC3200 device with several AWS services including API Gateway, Lambda, Amazon S3, and Amazon SES.

### Step 1: Request a Pre-Signed Upload URL

When the user selects the save function, the CC3200 first prepares the canvas image for upload by converting the framebuffer into a `.bmp` image.

The device then establishes a secure TLS connection to the AWS API Gateway endpoint using HTTPS (port 443). After the connection is created, the device sends an HTTP **GET request** to a Lambda function through the API Gateway. The Lambda function generates a **pre-signed URL** that allows the device to upload a file directly to the S3 bucket without exposing credentials.

Once the Lambda function returns the response, the device extracts the upload URL from the JSON response.

---

### Step 2: Upload Image to Amazon S3

After receiving the pre-signed URL, the CC3200 sends an HTTP **PUT request** to upload the generated `.bmp` image file to the S3 bucket.

The request includes:

- The pre-signed URL endpoint
- The content type (`image/bmp`)
- The size of the image data
- The binary image data generated from the canvas buffer

Once the PUT request completes successfully, the image is stored in the Amazon S3 bucket.

---

### Step 3: Trigger S3 Event Notification

The S3 bucket is configured with an **event notification** that triggers whenever a new file is uploaded.

When the drawing image is saved in the bucket, Amazon S3 automatically emits a **PutObject event**, which invokes a second AWS Lambda function. This Lambda function receives the event information, including the bucket name and file key of the uploaded image.

---

### Step 4: Retrieve the Image from S3

The Lambda function processes the event by extracting the uploaded file information. It then retrieves the image file from the S3 bucket using the AWS SDK.

The downloaded image data is converted into a binary buffer so it can be attached to an email.

---

### Step 5: Send Image to the User via Email

The final step uses **Amazon SES (Simple Email Service)** to send the drawing image to the user.

The Lambda function constructs a raw MIME email message containing:

- The sender email address
- The recipient email address
- The email subject and message body
- The uploaded drawing image as an attachment

The image is encoded into **Base64 format** so it can be included in the email attachment.

After the email message is created, the Lambda function sends it using the SES API. The user then receives the generated drawing image as an email attachment.

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
