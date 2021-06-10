## YOLOv4(tiny) API

#### API that returns an output of a YOLOv4(tiny) model when an image is received from another application server.

This API is to be used to provide a detection answer(sign language word) that corresponds to a frame which is sent from a web application which captures a real-time sign language action of a user.)


```python
import cv2
import numpy as np
import matplotlib.pyplot as plt
import time

config_path=r'C:\Study\Python\notebook\Video_Detection\yolov4\custom_weight\tiny\yolov4-tiny.cfg'
weights_path=r'C:\Study\Python\notebook\Video_Detection\yolov4\custom_weight\tiny\yolov4-tiny_last.weights'

# classes of sign language words (represented by unique indexes)
classes = ['12','15','21','29','37','45','46']

# provide the path for testing cofing file and tained model weight in order to create a net
net = cv2.dnn.readNetFromDarknet(config_path, weights_path)
```


```python
# given an image, resize the image to prepare to put into the model
img_path = r'C:\Study\Python\notebook\Video_Detection\finalproject_signlang\sign_lang_dict\12_hungry_0227_92.png'
img = cv2.imread(img_path)

img = cv2.resize(img,(1280,720))
height, width, _ = img.shape
```


```python
blob = cv2.dnn.blobFromImage(img, 1/255,(416,416),(0,0,0),swapRB = True,crop= False)
net.setInput(blob)
output_layers_name = net.getUnconnectedOutLayersNames()
layerOutputs = net.forward(output_layers_name)
```


```python
boxes =[]
confidences = []
class_ids = []

for output in layerOutputs:
    for detection in output:
        score = detection[5:]
        #print("score: ",score)
        class_id = np.argmax(score)
        #print("class_id: ",class_id)
        confidence = score[class_id]
        #print("confidence: ",confidence)
        if confidence > 0.5:
            center_x = int(detection[0] * width)
            center_y = int(detection[1] * height)
            
            w = int(detection[2] * width)
            h = int(detection[3]* height)
            #print('w: ' ,w,'h: ',h)
            x = int(center_x - w/2)
            y = int(center_y - h/2)
            #print('x: ' ,x,'y: ',y)

            boxes.append([x,y,w,h])
            confidences.append((float(confidence)))
            class_ids.append(class_id)

indexes = cv2.dnn.NMSBoxes(boxes,confidences,.8,.4)
font = cv2.FONT_HERSHEY_PLAIN
colors = np.random.uniform(0,255,size =(len(boxes),3))
if  len(indexes)>0:
    for i in indexes.flatten():
        x,y,w,h = boxes[i]
        label = str(classes[class_ids[i]])
        confidence = str(round(confidences[i],2))
```


```python
# save the output of the model as a json data (to be sent/received as Httprequest)

import json

send_to_spring = {
    'word_index':label,
    'box_coord':[x,y,w,h]
}
json_dump = json.dumps(send_to_spring)
```


```python
# as expected, word hungry (represented by index 12) corresponds to the input image.
json_dump
```




    '{"word_index": "12", "box_coord": [486, 516, 312, 114]}'


