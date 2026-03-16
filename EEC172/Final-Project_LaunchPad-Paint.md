# Final Project – LaunchPad Paint

LaunchPad Paint is a real-time drawing program for the CC3200 LaunchPad.  On top of the board, it utilizes an IR receiver and TV remote for input, an Adafruit OLED screen to display everything, and Amazon AWS services to store images in the cloud as well as send the images to subscribers via email.

This project demonstrates a complete IoT workflow that integrates embedded hardware interaction, graphical rendering, wireless networking, and cloud services.

---

- [Download](#download)
- [Setup](#setup)
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
- [Future Work](#future-work)
- [Bill of Materials](#bill-of-materials)

---

# Download [[Link]](./launchpad-paint_no-sensitive-info.zip)
Be sure to unzip the file before using.

---

# Setup
You will need the following hardware:
1. A CC3200 LaunchPad.
2. A TSOP31xxx IR Receiver
3. A 100Ω resistor.
4. A 100μF capacitor.
5. An Adafruit 1.5" SSD1351 128x128 RGB OLED.
6. An AT&T S10-S3 TV Remote + 2 AA batteries.
7. A USB-to-Micro-USB cable.

And the following software:
1. Code Composer Studio v12.5.0
2. CC3200 SDK Files
3. (Optional) UniFlash v3.4.1

Instructions for properly setting up this software can be found [here](https://tailailihe.github.io/UCDavis-EEC172-Lab-Manual/labs/lab-setup.html).

To setup the hardware, connect everything according to the following table:

| Pin From | Pin To |
| -------- | ------ |
| OLED SI  | CC3200 P2.6 |
| OLED CL  | CC3200 P1.7 |
| OLED DC  | CC3200 P1.5 |
| OLED R   | CC3200 P2.4 |
| OLED OC  | CC3200 P2.10 |
| OLED +   | CC3200 VCC |
| OLED G   | CC3200 GND |
| IR Receiver GND | CC3200 GND |
| IR Receiver Vs | 100𝜇 F Capacitor + 100Ω Resistor |
| 100𝜇F Capacitor | CC3200 GND |
| 100Ω Resistor | CC3200 VCC |
| IR Receiver OUT | CC3200 P2.2 |

Then plug in your CC3200 LaunchPad into your computer and open Code Composer Studio.  When asked to open a workspace, navigate to where you unzipped the file and select the `workspace` folder.  With the project loaded, right-click the project and enter its properties.  Then navigate to `Resource > Linked Resources`.  Here, change the value of the `CC3200_SDK_ROOT` to where yourr CC3200 SDK files are located.

Next open up `wirelss_comm.h`.  Here, you'll need to replace all fields in the angle brackets `< >` with the appropriate values corresponding to your AWS services, as well as update the date and time stored in the macros below it.  To create the needed AWS services to run the saving functionality of this proogram, please refer to sections `3.1.4`, `3.1.6`, `3.1.10`, and `3.1.11` in [this project report](./EEC172_Final-Project_Report.pdf).

Finally, open up `commmon.h` (hold ctrl and click the include statement that references it) and update `SSID_NAME`, `SECURITY_TYPE`, and `SECURITY_KEY` with your Wi-Fi network's configuration.  This connection must have a 2.4 GHz band.  If you have trouble connecting to it with a password, try no password.

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
6. A Lambda function is setup to listen for any changes to the AWS S3 Bucket's storage and sends any modified `.bmp` files to subscribers via email.

<img src="./system_architecture_diagram_final.png" width="60%" alt="System Architecture Diagram">

# Functional Specification

The functional specification describes the workflow of the drawing system, including initialization, user interaction, drawing operations, and cloud image upload.

1. When the program starts, the CC3200 initializes board configurations, GPIO pins, SPI, OLED display, SysTick, and UART.
2. The system creates the canvas buffer, renders the UI, draws the initial canvas, and connects to Wi-Fi.
3. The program enters the main loop and continuously waits for input from the IR remote controller.
4. When a button is pressed, the IR signal is decoded and the system determines which drawing feature to execute.
5. The system updates the cursor position, drawing tool, color, or canvas state, and then re-renders the canvas and cursor on the OLED display.
6. If the save function is triggered, the canvas buffer is converted to image data and uploaded to cloud storage using a pre-signed URL obtained from AWS.

<img src="./functional_specification_diagram_final.png" width="80%" alt="System Architecture Diagram">

# Part I: Drawing Implementation

The drawing system was developed using the code from **Lab 4** as the base template. Lab 4 already contains the TV remote input handling and text interface functionality, which allows the CC3200 to receive and decode button commands from the IR remote. Building on this foundation, the drawing features were integrated into the existing framework. The new implementation adds canvas management, cursor control, and drawing tools while reusing the remote input system to control drawing operations.

### Framebuffer and Canvas Management

The drawing canvas is represented by a framebuffer stored in memory. Each element of the framebuffer corresponds to a pixel on the OLED display. All drawing operations modify this buffer first before the updated image is rendered on the screen. This design allows the system to keep track of the entire drawing state and refresh the display efficiently.

### Cursor Movement

A cursor is used to indicate the current drawing position on the canvas. The cursor can move in four directions based on user input from the remote controller. Boundary checks ensure that the cursor remains within the canvas area. The cursor is visually displayed on the screen to help users position their drawings accurately.

### Basic Drawing Operations

The system supports drawing individual pixels using a pencil tool. When the user activates the drawing action, the pixel at the cursor position is updated with the currently selected color. The eraser tool works similarly but replaces the pixel color with the background color. These operations allow users to manually create or modify drawings on the canvas.

### Canvas Control

Users can reset the entire canvas when needed. The clear function removes all drawing content by resetting every pixel in the framebuffer to the background color. The updated blank canvas is then rendered to the display.

### Color and Tool Selection

The drawing system supports multiple colors and tools that users can switch between during operation. The available tools include pencil, eraser, line, rectangle, ellipse, and bucket fill. Switching tools changes how input actions are interpreted by the system.

### Shape Drawing

For geometric shapes such as lines, rectangles, and ellipses, the user selects two points on the canvas. The first point defines the starting position, and the second point defines the ending position or bounding region. The system then generates the corresponding shape between the two points.

### Bucket Fill

The bucket fill tool allows users to quickly color a connected region of the canvas. When activated, the system replaces neighboring pixels that share the same base color with the selected drawing color, producing a flood-fill effect similar to standard paint applications.


# Part II: User Interface

## Physical

The system uses a TV remote as the primary input device.

Button mappings:

| Button | Function |
|------|------|
| Vol Up / Down | Change drawing tool |
| Mute | Change color |
| Ch Up / Down | Change program function to run |
| Last | Run program function |
| 2 | Move cursor up |
| 4 | Move cursor left |
| 6 | Move cursor right |
| 8 | Move cursor down |
| 5 | Use currently select tool |
| 0 | Clear canvas |

## Visual

<img src=./ui_preview.png width=50%>

The top left is the name of the program.  The text in this preview image is different from the final program because we do not know what font the text drawing function uses.  The top right is the Wi-Fi indicator, which shows if the device is connected to a wireless access point or not.  On the left are all available tools the user can choose from.  Tools with similar functions are grouped together.  On the right are all program functions the user can run.  In the center is the main canvas the user can draw on.  It is green in the preview image instead of white to better show the drawable area.  Near the bottom right corner of the canvas is the point indicator, which becomes solid if a point has been set by one of the tools.  Unimplemented tools and program functions have a gray background.

### Tools

<img src=./icons/pencil.png width=24px>
Pencil: Draws a single pixel in the user's selected color.

<img src=./icons/eraser.png width=24px>
Eraser: Erases a single pixel and replaces it with the background color.

<img src=./icons/gyro.png width=24px>
(Not implemented) Gyro: Spawns a ball that makes all pixels under it the user's selected color.  The ball can be moved by tilting the CC33200 LaunchPad.

<img src=./icons/square.png width=24px>
Rectangle: Draws a hollow rectangle of the user's selected color between a previously selected point and the cursor's position.

<img src=./icons/square_filled.png width=24px>
Filled Rectangle: Draws a filled rectangle of the user's selected color between a previously selected point and the cursor's position.

<img src=./icons/circle.png width=24px>
Ellipse: Draws a hollow ellipse of the user's selected color between a previously selected point and the cursor's position.

<img src=./icons/circle_filled.png width=24px>
Filled Ellipse: Draws a filled ellipse of the user's selected color between a previously selected point and the cursor's position.

<img src=./icons/line.png width=24px>
Line: Draws a line of the user's selected color between a previously selected point and the cursor's position.

<img src=./icons/polygon.png width=24px>
(Not implemented) Polygon: Draws a polygon of the user's selected color based on several points selected by the user.

<img src=./icons/bucket.png width=24px>
Bucket: Fills an area with the user's selected color.

### Program Functions

<img src=./icons/save.png width=24px>
Save: Convert the drawing on the canvas into a `.bmp`, save it to cloud storage, and email it subscribers.

<img src=./icons/wifi.png width=24px>
(Not implemented) Wi-Fi: Switch which wireless access point the device is connected to.

<img src=./icons/login.png width=24px>
(Not implemented) Login: Login into a registered account.

---

# Part III: Save Image to Cloud Storage

This part describes how the system uploads the generated drawing to cloud storage and sends the saved image to the user through email. The process uses AWS API Gateway, AWS Lambda, Amazon S3, and Amazon SES.

### Step 1: Prepare the Image Data

When the user triggers the save function, the drawing stored in the device's framebuffer is converted into a `.bmp` image file. This image data will later be uploaded to cloud storage.

### Step 2: Connect to AWS API Gateway

The device must establish a secure connection with AWS in order to request permission to upload the image.

1. Determine the API Gateway endpoint in the format  
   `<id>.execute-api.<region>.amazonaws.com`.
2. Open a TLS connection to this server using port **443**, the standard HTTPS port.
3. Once the secure connection is established, the device can communicate with the Lambda service through API Gateway.

### Step 3: Request a Pre-signed Upload URL

Instead of directly uploading files with AWS credentials, the system requests a **pre-signed upload URL** from a Lambda function.

1. Send an HTTP **GET request** to the Lambda endpoint through API Gateway.
2. The Lambda function generates a temporary upload URL that allows the device to upload a file directly to the S3 bucket.
3. The Lambda function returns a JSON response containing:
   - the **pre-signed upload URL**
   - the **file key** that will identify the stored image.

### Step 4: Extract the Upload URL

After receiving the server response:

1. The device processes the returned JSON message.
2. The program extracts the **upload URL** from the response.
3. This URL will be used as the destination for the image upload.

### Step 5: Upload the Image to Amazon S3

Using the pre-signed URL, the device uploads the generated BMP image to the S3 bucket.

1. Determine the S3 host endpoint in the format  
   `<bucket-name>.s3.<region>.amazonaws.com`.
2. Create a secure TLS connection to the S3 server on port **443**.
3. Send an HTTP **PUT request** to the pre-signed URL.
4. Include the BMP image data as the payload of the request.
5. After the request completes successfully, the image is stored in the S3 bucket.

### Step 6: Trigger an S3 Event

Once the image is uploaded:

1. The S3 bucket is configured with an **event notification rule**.
2. When a new object is uploaded to the bucket, the bucket automatically emits an event.
3. This event triggers a second AWS Lambda function that handles post-processing.

### Step 7: Process the Uploaded Image

The triggered Lambda function performs the following tasks:

1. Extract information from the S3 event, including:
   - the bucket name
   - the uploaded file name
2. Retrieve the uploaded image file from the S3 bucket.
3. Convert the file into a usable data buffer.

### Step 8: Send the Image by Email

The Lambda function then sends the uploaded image to the user through Amazon SES.

1. Configure verified email identities in **Amazon SES**.
2. The Lambda function constructs an email message with the image attached.
3. The email is sent using the **SendRawEmail** command through SES.
4. The user receives the drawing image as an email attachment.

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
