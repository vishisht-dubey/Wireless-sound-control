# Wireless-sound-control
import cv2
import mediapipe as mp
import numpy as np
import math
from mediapipe.python.solutions.drawing_utils import draw_landmarks
from ctypes import cast, POINTER
from comtypes import CLSCTX_ALL
from pycaw.pycaw import AudioUtilities, IAudioEndpointVolume
devices = AudioUtilities.GetSpeakers()
interface = devices.Activate(
    IAudioEndpointVolume._iid_, CLSCTX_ALL, None)
volume = cast(interface, POINTER(IAudioEndpointVolume))

mpDraw = mp.solutions.drawing_utils
mpHands = mp.solutions.hands
hands = mpHands.Hands()
cap = cv2.VideoCapture(0)
while True:
    success,img = cap.read()
    results = hands.process(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    if results.multi_hand_landmarks:
        for handLms in results.multi_hand_landmarks:
            lmList = []
            for id ,lm in enumerate(handLms.landmark):
               # print(id, lm)
                h ,w ,c = img.shape
                cx ,cy = int(lm.x*w) , int(lm.y*h)
                lmList.append([id,cx,cy])
                mpDraw.draw_landmarks(img, handLms ,mpHands.HAND_CONNECTIONS)
                
                
            if lmList:
                x1 ,y1 = lmList[4][1] , lmList[4][2]
                x2 ,y2 = lmList[8][1] , lmList[8][2]
                cv2.circle(img, (x1,y1), 20, (35,56,110), cv2.FILLED)
                cv2.circle(img, (x2,y2), 20, (35,56,110), cv2.FILLED)
                cv2.line(img, (x1,y1), (x2,y2), (255, 0 , 255) , 3)
                length = math.hypot(x2-x1,y2-y1)
                z1 , z2 = (x1+x2)//2 , (y1+y2)//2
                
            if length<50 :
                cv2.circle(img , (z1,z2) ,15 , (250,250,250) ,cv2.FILLED)
                
                
            volRange = volume.GetVolumeRange()
                
            minVol = volRange[0]    
            maxVol = volRange[1]
            vol = np.interp(length , [30,350] , [minVol ,maxVol])
            volBar = np.interp(length , [50, 300] , [400 ,150])
            volPer = np.interp(length , [50 ,300] , [0 , 100])
            
            volume.SetMasterVolumeLevel(vol, None)
            cv2.rectangle(img , (50 ,150) , (85 , 400) ,(123,213,122) ,3)
            cv2.rectangle(img , (50 , int(volBar)) , (85 ,400) ,(0, 231,23) ,cv2.FILLED)
            cv2.putText(img , str(int(volPer)) , (40, 450) ,cv2.FONT_HERSHEY_PLAIN ,4 , (24,34,34) , 3)
         
            cv2.imshow("image",img)
            cv2.waitKey(1)
