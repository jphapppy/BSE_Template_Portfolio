# Smart Glasses
Instead of holding a camera, smart glasses offer a hands free way to record stuff!  These involve mounting a camera onto a pair of glasses and connecting it to a simple processor system like a raspberry Pi so it is portable.  The camera won't 100% detect what your eyes are seeing, but it still gets most of what your eyes are seeing.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jason P | Stratford Prepatory Blackford | Health | Incoming Sophomore |

<!--
**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)

-->
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/NZ_2Al3L7MA?si=PVjcaJ8wZCsASP4x" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

After hot gluing my camera to my glasses, I wanted to make modifications.  So, I worked on modifications.  Instead of using tensorFlow's general AI, I wanted to train my own.  The first step to this was taking a picture, and being able to send that to google drive.  I first had to download google auth and get google cloud, make an OAuth client, and connect it.  Then, I had to authorize it so it could acess my google drive.  I ran into a lot of problems, such as not being able to log in because "the client didn't support javascript."  However, I fixed it by making sure my creditals were properly made.  When I logged in, the raspberry Pi got access.  It was able to uplaod a file of mine.  So, Then I added the image capturing code above, and now it works well.



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/KwafafNAArw?si=tgGg_9rC-52swaLf" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

I transitioned from openCV to tensorFlow so I could get videos.  The frame rate is low, but the dectection is okay.  The amount of commands I had to input into the terminal was a suprise.  Last milestone, I couldn't see exactly what the camera was seeing, but now I can see everything the camera is seeing thanks to tensorFlow.  The camera is able to detect computer keyboards, mouses, and items tensorFlow knows.  The text to speech works well, but its voice sounds choppy, so I may want to get a better one. I want to put a longer cable for more mobility as well. I now need to put the camera on the glasses.

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/eIhywB4pccY?si=DQWUQpK5KKYoSYFi" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This is the first milestone for my smart glasses project, which is setting up the raspberry pi.  This was not much of a challenge but it look a long time to download all the apps I needed for the raspberry pi and downloading the software for the pi.  All waiting aside, now it works well.  I can only take one picture at a time using openCV, but this the first step at using the camera.

# Starter Project : Jitterbug

<iframe width="560" height="315" src="https://www.youtube.com/embed/7qXCmAjE5eM?si=l6Hw6ky5PzJQHeuj" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This was my first project here at BlueStamp where I put my soldering skills to the test.  I would've completed it on my first day if it wasn't for my soldering mistake.  I soldered the LEDs the wrong way, and removing them took time.  I learned that I must be exact when soldering or else I will suffer.
<!-- 
# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 
-->
# Code
This is my code for taking a photo and uploading it to my google drive.  First, I am importing core modules needed to authenticate with google cloud and upload files to google drive.  Then, I am importing modules realated to the piCam taking pictures.  Next, the piCam takes a picture and writes in in the "file.png" file.  Then, I am declaring my scope.  I set it to "allow all access to drive" so it could upload files and overcome permission errors.  Since I already made my token and authorized it, the code then checks the token.json first, for user access.  It only checks creditials if the token isn't there or it's invalid.  I set it to go to one of the folders in my google drive.

```c++
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError
from googleapiclient.http import MediaFileUpload

import os.path

from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow

from picamera2 import Picamera2, Preview
import time
import cv2

import vosk
import pyaudio
import json

import pyaudio

from PIL import Image
import google.generativeai as genai
import subprocess



genai.configure(api_key="AIzaSyAlwZVLGghp18yCng36Dp5XZnJCTtGx8qo")  

picam2 = Picamera2()
camera_config = picam2.create_still_configuration(main={"size": (1920, 1080)},
lores={"size": (640, 480)}, display="lores")
picam2.configure(camera_config)
#picam2.start_preview(Preview.QTGL) #Comment this out if not using desktop interface
picam2.start()


model = vosk.Model("/home/jasonpark/models/vosk-model-small-en-us-0.15")
rec = vosk.KaldiRecognizer(model, 16000, '["terminate", "upload", "send", "exit"]')

p = pyaudio.PyAudio()
stream = p.open(format=pyaudio.paInt16,
                channels=1,
                rate=16000,
                input=True,
                frames_per_buffer=8192)

SCOPES = ['https://www.googleapis.com/auth/drive']

  
"""Shows basic usage of the Drive v3 API.
Prints the names and ids of the first 10 files the user has access to.
"""
creds = None
# The file token.json stores the user's access and refresh tokens, and is
# created automatically when the authorization flow completes for the first
# time.
if os.path.exists("token.json"):
    creds = Credentials.from_authorized_user_file("token.json", SCOPES)
# If there are no (valid) credentials available, let the user log in.
if not creds or not creds.valid:
    if creds and creds.expired and creds.refresh_token:
        creds.refresh(Request())
    else:
        flow = InstalledAppFlow.from_client_secrets_file(
            "/home/jasonpark/credentials.json", SCOPES
        )
    creds = flow.run_local_server(port=0)
    # Save the credentials for the next run
    with open("token.json", "w") as token:
        token.write(creds.to_json())


print("Listening for speech. Say 'Terminate' to stop.")
# Start streaming and recognize speech
while True:
    data = stream.read(4096)#read in chunks of 4096 bytes
    if rec.AcceptWaveform(data):#accept waveform of input voice
        # Parse the JSON result and get the recognized text
        result = json.loads(rec.Result())
        recognized_text = result['text']
        print(f"Recognized: {recognized_text}")
        
        # Check for the termination keyword
        if "terminate" in recognized_text.lower():
            print("Termination keyword detected. Stopping...")
            break

        if "send" in recognized_text.lower():
            print("send keyword detected. sending to gemini...")
            img = Image.open("file.png")  # Ensure the image exists

            # Set up Gemini Vision model
            model = genai.GenerativeModel("gemini-2.5-flash")

            # Send image with a prompt
            response = model.generate_content(
                [
                    "First, check if this picture has a visible plant.  Asnswer yes or no. Then, check if it has a disease.  If it does, describe what type of disease briefly.",
                    img
                ]
            )
            gemini_text = response.text
            print(gemini_text)
            # Load image from Raspberry Pi

        if "upload" in recognized_text.lower():
            im = picam2.capture_array()
            im = cv2.cvtColor(im, cv2.COLOR_BGR2RGB)
            cv2.imwrite('file.png', im)
            print("upload keyword detected. uploading...")
            try:
                # create drive api client
                service = build("drive", "v3", credentials=creds)
                folder_id = "1vx-evFr1qBKo6yGuxheW8QfZfdWrDmRz"
                file_metadata = {"name": "file.png"
                                , "parents": [folder_id]}

                media = MediaFileUpload("file.png", mimetype="image/png")
                # pylint: disable=maybe-no-member
                file = (
                    service.files()
                    .create(body=file_metadata, media_body=media, fields="id")
                    .execute()
                )
                print(f'File ID: {file.get("id")}')

            except HttpError as error:
                print(f"An error occurred: {error}")
                file = None
            print("Upload complete.")

stream.stop_stream()
stream.close()

# Terminate the PyAudio object
p.terminate()



# If modifying these scopes, delete the file token.json.
```
<!--
# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
-->
# Other Resources/Examples
- [Example 1](https://zoemell.github.io/Zoe_BSE_Portfolio/)
- [Example 2](https://thedinosour.github.io/Chris_BlueStampPortfolio/)

