# Smart Glasses
Instead of holding a camera, smart glasses offer a hands free way to record stuff!  These involve mounting a camera onto a pair of glasses and connecting it to a simple processor system like a Raspberry Pi so it is portable.  The camera won't 100% detect what your eyes are seeing, but it still gets most of what your eyes are seeing.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jason P | Stratford Preparatory Blackford | Health | Incoming Sophomore |



![Headstone Image](JasonP.jpg)

Modification 2
For my final modification, I added an OLED screen so other people could see the functions I was performing.  When I send the picture to gemini, the OLED displays a gemini icon.  When I send the picture to google, it shows an upload icon.  Finally, when it terminates, it displays an "x" icon.  Since it is transparent, I can see from both sides.  So I made a holder

# Modification 1

<iframe width="560" height="315" src="https://www.youtube.com/embed/x4km8omDQBc?si=jzDNtt75-EbsKW5w" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
For my first modification, I wanted to integrate Gemini so it could recognize plant dieases from an image.  I also wanted to pair Gemini's response with TTS.  It would speak Gemini's response so the user wouldn't have to look in the terminal.  The biggest challenge happened when the TTS was not working.  It turns out it was trying to use a voice that wasn't supported but I installed espeak-ng and finally got the TTS to work.  Another problem I faced happened when the TTS was not coming out the headphone jack even though I had it plugged in.  This was a simple fix in the Pi settings.  I then used pyaudio voice recognition for hands free image capturing and image sending, so it would be more convient.  Most of the code stays the same, it still is able to capture an image and upload it.  However, now there is a while loop constantly running and picking up audio from my mic.  If it detects a keyword, such as "terminate," "send," or "upload," the respective commands will happen.  "Terminate" stops the program.  "Send" send the most recent "file.png" to gemini.  "Upload" takes a picture, overwrites "file.png," and sends it to google drive.


# Code
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



genai.configure(api_key="...")  

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
                    "First, check if this picture has a visible plant.  Answer yes or no. Then, check if it has a disease.  If it does, describe what type of disease briefly.",
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
# Final Milestone
<iframe width="560" height="315" src="https://www.youtube.com/embed/NZ_2Al3L7MA?si=PVjcaJ8wZCsASP4x" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For my final milestone, after hot gluing my camera to my glasses, I wanted to add modifications because I wanted to do more for this milestone.  I used google cloud to send pictures from my raspberry pi to my google drive folder. I first had to download google auth and get google cloud, make an OAuth client, and connect it.  Then, I had to authorize it so it could acess my google drive.  I ran into a lot of problems, such as not being able to log in because "the client didn't support javascript."  However, I fixed it by making sure my credentials were properly made.  When I logged in, the raspberry Pi got access.  It was able to upload a file of mine.  So, Then I added the image capturing code below, and now it works well.  I wanted to be able to store pictures so that the user could later look at them if they needed it.

# Code
```python
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


picam2 = Picamera2()
camera_config = picam2.create_still_configuration(main={"size": (1920, 1080)},
lores={"size": (640, 480)}, display="lores")
picam2.configure(camera_config)
#picam2.start_preview(Preview.QTGL) #Comment this out if not using desktop interface
picam2.start()
time.sleep(2)
im = picam2.capture_array()
im = cv2.cvtColor(im, cv2.COLOR_BGR2RGB)
cv2.imwrite('file.png', im)

# If modifying these scopes, delete the file token.json.
SCOPES = ['https://www.googleapis.com/auth/drive']
def upload_basic():
  
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
                "credentials.json", SCOPES
            )
        creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open("token.json", "w") as token:
            token.write(creds.to_json())
    """Insert new file.
    Returns : Id's of the file uploaded

    Load pre-authorized user credentials from the environment.
    TODO(developer) - See https://developers.google.com/identity
    for guides on implementing OAuth2 for the application.
    """

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

    return file.get("id")


if __name__ == "__main__":
  upload_basic()
```
This is my code for taking a photo and uploading it to my google drive.  First, I am importing core modules needed to authenticate with google cloud and upload files to google drive.  Then, I am importing modules related	to the Picam, which helps take pictures.  Next, the Picam takes a picture and writes in the "file.png" file.  Then, I am declaring my scope.  I set it to "allow all access to drive" so it could upload files and overcome permission errors.  Since I already made my token and authorized it, the code then checks the token.json first, for user access.  It only checks credentials if the token isn't there or it's invalid.  I set it to go to one of the folders in my google drive.
# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/KwafafNAArw?si=tgGg_9rC-52swaLf" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This is the second milestone of my project, where I performed basic object recognition using the Picam. I transitioned from openCV to tensorFlow so I could do object recognition.  The frame rate is low (because it's just constantly taking pictures, not an actual video) but the detection is decent.  The amount of commands I had to input into the terminal was a surprise.  Last milestone, I couldn't see exactly what the camera was seeing, but now I can see everything the camera is seeing thanks to tensorFlow.  There is a screen that pops up in the terminal that shows what the camera is seeing, frame by frame.  The camera is able to detect computer keyboards, mice, and other basic items.  The model says the object it detects out loud. This was a stepping stone for better recognition.  I kept running into errors in downloading tensorFlow, but i fixed it by running the commands line-by-line (i just copied and pasted a block of commands before this).
# Code

```python
 while not capture_manager.stopped:
        if capture_manager.frame is None:
            continue
        buffer.fill((0,0,0))
        frame = capture_manager.read()
        # get the raw data frame & swap red & blue channels
        previewframe = np.ascontiguousarray(capture_manager.frame)
        # make it an image
        img = pygame.image.frombuffer(previewframe, capture_manager.resolution, 'RGB')
        img = pygame.transform.scale(img, scaled_resolution)

        cropped_region = (
            (img.get_width() - buffer.get_width()) // 2,
            (img.get_height() - buffer.get_height()) // 2,
            buffer.get_width(),
            buffer.get_height()
        )

        # draw it!
        buffer.blit(img, (0, 0), cropped_region)

        timestamp = time.monotonic()
        if args.tflite:
            prediction = model.tflite_predict(frame)[0]
        else:
            prediction = model.predict(frame)[0]
        logging.info(prediction)
        delta = time.monotonic() - timestamp
        logging.info("%s inference took %d ms, %0.1f FPS" % ("TFLite" if args.tflite else "TF", delta * 1000, 1 / delta))
        print(last_seen)

        # add FPS & temp on top corner of image
        fpstext = "%0.1f FPS" % (1/delta,)
        fpstext_surface = smallfont.render(fpstext, True, (255, 0, 0))
        fpstext_position = (buffer.get_width()-10, 10) # near the top right corner
        buffer.blit(fpstext_surface, fpstext_surface.get_rect(topright=fpstext_position))
        try:
            temp = int(open("/sys/class/thermal/thermal_zone0/temp").read()) / 1000
            temptext = "%d\N{DEGREE SIGN}C" % temp
            temptext_surface = smallfont.render(temptext, True, (255, 0, 0))
            temptext_position = (buffer.get_width()-10, 30) # near the top right corner
            buffer.blit(temptext_surface, temptext_surface.get_rect(topright=temptext_position))
        except OSError:
            pass

        for p in prediction:
            label, name, conf = p
            if conf > CONFIDENCE_THRESHOLD:
                print("Detected", name)

                persistant_obj = False  # assume the object is not persistant
                last_seen.append(name)
                last_seen.pop(0)

                inferred_times = last_seen.count(name)
                if inferred_times / len(last_seen) > PERSISTANCE_THRESHOLD:  # over quarter time
                    persistant_obj = True

                detecttext = name.replace("_", " ")
                detecttextfont = None
                for f in (bigfont, medfont, smallfont):
                    detectsize = f.size(detecttext)
                    if detectsize[0] < screen.get_width(): # it'll fit!
                        detecttextfont = f
                        break
                else:
                    detecttextfont = smallfont # well, we'll do our best
                detecttext_color = (0, 255, 0) if persistant_obj else (255, 255, 255)
                detecttext_surface = detecttextfont.render(detecttext, True, detecttext_color)
                detecttext_position = (buffer.get_width()//2,
                                       buffer.get_height() - detecttextfont.size(detecttext)[1])
                buffer.blit(detecttext_surface, detecttext_surface.get_rect(center=detecttext_position))

                if persistant_obj and last_spoken != detecttext:
                    subprocess.call(f"echo {detecttext} | festival --tts &", shell=True)
                    last_spoken = detecttext
                break
        else:
            last_seen.append(None)
            last_seen.pop(0)
            if last_seen.count(None) == len(last_seen):
                last_spoken = None

        screen.blit(pygame.transform.rotate(buffer, args.rotation), (0,0))
        pygame.display.update()

if __name__ == "__main__":
    args = parse_args()
    try:
        main(args)
    except KeyboardInterrupt:
        capture_manager.stop()

```
This code keeps checking for objects is recognizes until it is forcefully stopped by the keyboard command "control C"(assuming no errors with piCam, system, etc.)  If it's confidence is over 50%, it will say the name of the object it thinks it saw on the screen.
# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/eIhywB4pccY?si=DQWUQpK5KKYoSYFi" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This is the first milestone for my smart glasses project, which is setting up the raspberry pi.  This was not much of a challenge but it took a long time to download all the apps I needed for the raspberry pi and downloading the software for the pi.  I first had to get OBS, but I couldn't control the Pi on my computer yet.  I had to plug in a keyboard and mouse into the Pi so I could see what was happening on my computer screen.  I needed a direct connection so I could enable the ssh and then got tigerVNC, where I could control the Pi directly on my computer, and then setting up an SSH with VS code so I could code in python and run that code on the Pi.  All waiting aside, now it works well.  I can only take one picture at a time using openCV, but this the first step at using the camera.  Now that I know the Picam works properly, I can continue to improve on it.
# Code

```python
from picamera2 import Picamera2, Preview
import time
import cv2
picam2 = Picamera2()
camera_config = picam2.create_still_configuration(main={"size": (1920, 1080)},
lores={"size": (640, 480)}, display="lores")
picam2.configure(camera_config)
#picam2.start_preview(Preview.QTGL) #Comment this out if not using desktop interface
picam2.start()
time.sleep(2)
im = picam2.capture_array()
im = cv2.cvtColor(im, cv2.COLOR_BGR2RGB)
cv2.imwrite('file.png', im)

```
What happens here is that the piCam is getting set up, and openCV takes a picture, and it's designated to go to a file named "file.png".  It will overwrite file.png if it already exists.

# Starter Project : Jitterbug

<iframe width="560" height="315" src="https://www.youtube.com/embed/7qXCmAjE5eM?si=l6Hw6ky5PzJQHeuj" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

This was my first project here at BlueStamp where I put my soldering skills to the test.  I would've completed it on my first day if it wasn't for my soldering mistake.  I soldered the LEDs the wrong way, and removing them took time.  I learned that I must be exact when soldering or else I will suffer.
<!--
Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

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

