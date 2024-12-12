- 👋 Hi, I’m Abderrazaq
- 👀 I’m interested in Software engineering, and Embedded software engineering
- 🌱 I’m currently learning Deep learning and computer vision

<!---
elmaanaou/elmaanaou is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import cv2
import serial as sl
import sounddevice as sdevice
from gtts import gTTS
import soundfile as sfile
import threading as trd
import matplotlib.pyplot as plt
import numpy as np
import time

########################## Warning voice message ####################################
def text2Voice(text, fileName): 
    '''this function will help you generate voices from text. you need just to call
    the function, like that: text2Voice("hello", "hello.mp3")'''
    gtts = gTTS(text, lang='en')    
    gtts.save(fileName) 

class sound(): # Sound object
    def __init__(self, fileName):
        self.data, self.sf = sfile.read(fileName)
    def play(self):
        sdevice.play(self.data, self.sf)
        sdevice.wait()

sound1 = sound("voice1.mp3")
sound2 = sound("voice2.mp3")
sound3 = sound("alarm.mp3")

#######################################################################################


######################### global variables ############################################
lock = trd.Lock()
dtFlag = False
mvFlag = False
robber_face_position = []
#########################################################################################


######################### Behavior based on flag variable ################################
'''in this section we will tell the software what it can do based on flags.
       Here I will suffice just by running voice warnings'''
def behavior(): 
    global dtFlag, mvFlag
    repeat = False
    while True:
        if dtFlag == 1 and repeat == False:       
            sound1.play()
            repeat = True
        if repeat == True and mvFlag==1:
            sound2.play()
            sound3.play()
###########################################################################################

########################## Mouvement detection ############################################
''' This function will detect object mouvement, 
       and set mvFlag which indicate that the robber is mouving or not'''
def mouvement_detection():
    global mvFlag
    while True:
        D0 = robber_face_position
        time.sleep(1)
        D1 = robber_face_position
        if len(D0) == 2 and len(D1) == 2:
            deltax = (D1[0]-D0[0])**2
            deltay = (D1[1]-D0[1])**2
            if deltax>500 or deltay>500:
                mvFlag=True
            else:
                mvFlag=False
###########################################################################################

############################### Robber detection ##########################################
def detection():
    global dtFlag
    global robber_face_position
    '''You can specifie the opencv haarcascade that you want. 
       Here I used haarcascade_frontalface_default.xml'''
    haarcascade = 'haarcascade_frontalface_default.xml'
    face_cascade = cv2.CascadeClassifier(haarcascade)
    device_index = 0 # you must change this depending on the index related to your camera
    capture = cv2.VideoCapture(device_index)   
    while True:
        status, frame = capture.read()
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        faces = face_cascade.detectMultiScale(gray, 1.3, 5)
        for (x,y,w,h) in faces:
            robber_face_position = [x+w/2,y+h/2]
        cv2.imshow('Frames', frame)
        cv2.waitKey(1)
        if len(faces)>0:
            dtFlag = True
        else:
            dtFlag = False
###########################################################################################

def main():
    #As you know to make our software fast we must use threading
    targets = [detection, mouvement_detection, behavior]
    threads = []
    for target in targets:
        thread = trd.Thread(target=target)
        thread.start()
    for j in threads:
        j.join()


if __name__ == "__main__":
    main()
