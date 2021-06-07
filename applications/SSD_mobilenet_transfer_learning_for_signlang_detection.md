# SSD Mobilenet pretrained model을 활용한 sign language detection model train 및 test

### SSD + MobileNet이란?
#### Single Shot MultiBox Detector(SSD):
    A type of AI model framework that uses deep learning to detect objects from a single image
   
    SSD was the first deep neural architecture that did not use region proposals and featured an End-to-End approach to detecting objects in an image using a single deep neural network that was just as accurate as methods which did.

    The most crucial ingredient of the SSD framework and what also allows it achieve good performance in terms of both speed and accuracy is the fact that it utilizes small convolution filters to predict object classes and bounding box locations for different aspect ratios and does so across multiple feature maps from the later stages of the network allowing it to aggregate detections at multiple scales.

    Therefore, by utilizing multiple layers which are of relatively lower input resolution, and using those to generate predictions at different scales, it can provide high accuracy results with faster detection speeds. 

    two major shortcomings of SSD:
1. lower accuracy in detecting small objects (need data augmentation)
2. need improvements in inference time

#### MobileNet
    a CNN architecture for imae classification and mobile vision - require less computational power to be able to run mobile devices, embedded systems, computers without GPUs, or web browsers

# 0. Label images and split into train & test data sets

test 및 train data set으로 활용할 수어 구현 이미지를 확보한 후, labelImg를 활용하여 각 수어 이미지를 구별할 수 있는 수어단어를 label한다.
(labelImg 활용 방법 참고: 
https://github.com/tzutalin/labelImg)


```python
# 1. AiHUB의 수어 영상에서부터 이미지로 활용할 프레임 확보

import pandas as pd
import numpy as np

# 전체 데이터 셋에서 선택단어에 해당하는 annotation 데이터만 가져오기
selected = pd.read_excel('./data/selected_words.xlsx')

word_list = pd.Series(selected['단어'])
print(word_list)

dataset_annotation = pd.read_excel('./data/KETI-2017-SL-Annotation-v2_1.xlsx')
dataset_annotation = dataset_annotation.drop(["Unnamed: 7","Unnamed: 8"], axis=1)
dataset_annotation = dataset_annotation[dataset_annotation['한국어'].isin(word_list)]
dataset_annotation = dataset_annotation[dataset_annotation['방향']=='정면']
print(dataset_annotation)
```


```python
import cv2
import os
import time

# 수어 영상이 저장되어있는 폴더경로 (KETI_SL파일 1번 ~ 8380번)
video_folder = 'C:/MyDrive/0.FinalProject/Aihub_data_videos/'

# 영상에서 추출한 frame저장할 폴더
frame_folder = "./2.images/allImages"

file_list = list(dataset_annotation['파일명'])
for i in range(len(file_list)):
    # 영상 파일명과 같은 new _folder 생성
    new_folder = file_list[i][:-4]
    new_path = frame_folder + "/{0}".format(new_folder)
    
    if not (os.path.isdir(new_path)):
        # 해당 파일명과 동일한 폴더가 없으면 생성
        os.mkdir(os.path.join(new_path))

    # 영상 읽어서 원하는 부분 캡쳐(0.3초 간격 / 9 Frame)
    file_name = file_list[i].replace(file_list[i][-4:],'.avi')
    file_path = video_folder + file_name
    
    #video capture object생성
    cap = cv2.VideoCapture(file_path)
    time.sleep(5)
    count = 0
    while cap.isOpened():
        ret, frame = cap.read()

        if ret:
            count += 1
            # every single frame 추출
            if count % 1 == 0:
                save_path = new_path + "/{}_{}.png".format(file_list[i][14:-4],count)
                cv2.imwrite(save_path, frame)
            k = cv2.waitKey(33)
            if k == 27:
                break
        else:
            break

    cap.release()
    if (i % 10) == 9 :
        time.sleep(5)
```


```python
# 2. labelledImages 폴더에 있는 라벨링된 이미지들을 단어별, 영상 모델별로 train,test set으로 나눈다.
import os, shutil
from sklearn.model_selection import train_test_split

# dictionary형태의 input data에 데이터추가
# key값은 단어 index; value값은 라벨링된 이미지 파일명 리스트
input_data={}
person_data={}
for index in os.listdir('./Tensorflow/workspace/images/labelledImages'):
    file_list=[]
    person_list=[]
    for file in os.listdir('./Tensorflow/workspace/images/labelledImages/{}'.format(index)):
        if file.split('.')[-1].lower() == 'png': file_list.append(file.split('.')[0])
        person_list.append(file.split('.')[0][:4])
    persons = set(person_list)
    input_data[index] = file_list
    person_data[index] = persons
```


```python
word_list = list(input_data.keys())
print(word_list)
```


```python
# 단어별 사람별 구분 로직
for j in range(len(word_list)):
    p_list = list(person_data[word_list[j]])
    print(word_list[j])
    for p in range(len(p_list)):
        X_person=[]
        for z in input_data[word_list[j]]:
            if p_list[p] == z[:4]:
                X_person.append(z)
        print(p_list[p],":",X_person)
```


```python
# 단어별, 사람별 train_test_split - target data
p_list=[]
y=[]
for j in range(len(word_list)):
    p_list = list(person_data[word_list[j]])
    print(word_list[j])
    print(p_list)
    y_word=[]
    for p in range(len(p_list)):
        y_person=[]
        for z in input_data[word_list[j]]:
            if p_list[p] == z[:4]:
                y_person.append(word_list[j])
        print("len(y_person): ",len(y_person))
        y_word.append(y_person) 
    y.append(y_word)
```


```python
# 단어별, 사람별 train_test_split - input data
X_train_list=[]
X_test_list=[]

for j in range(len(word_list)):
    print(word_list[j])
    p_list = list(person_data[word_list[j]])
    print(p_list)
    X_word_train=[]
    X_word_test=[]
    for p in range(len(p_list)):
        X_person=[]
        for z in input_data[word_list[j]]:
            if p_list[p] == z[:4]:
                X_person.append(z)
        print("len(X_person): ",len(X_person))
        X_train, X_test, y_train, y_test = train_test_split(X_person, y[j][p], test_size=0.33, random_state=42)
        print(X_train)
        print(X_test)
        X_word_train.append(X_train)
        X_word_test.append(X_test)
    X_train_list.append(X_word_train)
    X_test_list.append(X_word_test)
```


```python
# train_test split한 파일들을 train, test 폴더로 이동
for n in range(len(word_list)):
    # train 파일 옮기기
    for r in X_train_list[n]:
        for filename in r:
            #.png파일 옮기기
            b_png='./Tensorflow/workspace/images_test3/labelledImages/{}/{}.png'.format(word_list[n],filename)
            a_png='./Tensorflow/workspace/images_test3/train/{}.png'.format(filename)
            shutil.move(b_png, a_png)
            #.xml파일 옯기기
            b_xml='./Tensorflow/workspace/images_test3/labelledImages/{}/{}.xml'.format(word_list[n],filename)
            a_xml='./Tensorflow/workspace/images_test3/train/{}.xml'.format(filename)
            shutil.move(b_xml, a_xml)
        
    # test 파일 옮기기
    for r in X_test_list[n]:
        for filename in r:
            #.png파일 옮기기
            b_png='./Tensorflow/workspace/images_test3/labelledImages/{}/{}.png'.format(word_list[n],filename)
            a_png='./Tensorflow/workspace/images_test3/test/{}.png'.format(filename)
            shutil.move(b_png, a_png)
            #.xml파일 옯기기
            b_xml='./Tensorflow/workspace/images_test3/labelledImages/{}/{}.xml'.format(word_list[n],filename)
            a_xml='./Tensorflow/workspace/images_test3/test/{}.xml'.format(filename)
            shutil.move(b_xml, a_xml)
```

# 1. Setup Paths

pretrained model을 불러와서 위과정을 통해 만든 데이터를 input으로 transfer learning을 수행하기위해 먼저 directory 경로를 정의한다.


```python
#workspace
WORKSPACE_PATH = './Tensorflow/workspace'
ANNOTATION_PATH = WORKSPACE_PATH+'/annotations'
IMAGE_PATH = WORKSPACE_PATH+'/images'
MODEL_PATH = WORKSPACE_PATH+'/models'
PRETRAINED_MODEL_PATH = WORKSPACE_PATH+'/pre-trained-models'

APIMODEL_PATH = './Tensorflow/models'
SCRIPTS_PATH = './Tensorflow/scripts'

# config file에 deel learning model이 사용하는 상세정보들이 담겨있다
CUSTOM_MODEL_NAME = 'my_ssd_mobnet' 
CHECKPOINT_PATH = MODEL_PATH+'/'+CUSTOM_MODEL_NAME
CONFIG_PATH = MODEL_PATH+'/'+CUSTOM_MODEL_NAME+'/pipeline.config'
```

# 2. Create Label Map

선택단어들을 level 1~4로 나누어서 모델훈련을 시킬것이기에, 각 레벨별 label map pbtxt파일 생성한다.


```python
import pandas as pd
import numpy as np

df_labels = pd.read_excel('./Tensorflow/workspace/annotations/labels.xlsx', index_col=None)
df_labels
```


```python
name_1 = list(df_labels[df_labels.레벨==1]['index'])
name_2 = list(df_labels[df_labels.레벨==2]['index'])
name_3 = list(df_labels[df_labels.레벨==3]['index'])
name_4 = list(df_labels[df_labels.레벨==4]['index'])
names = [name_1, name_2, name_3, name_4]
```


```python
def assign_levels(name):
    num=1; labels=[]
    for k in range(len(name)):
        word={}
        word['name']=name[k]
        word['id']=num; num+=1
        labels.append(word)
    return labels
```


```python
labels_all=[]

for name in names:
    labels_all.append(assign_levels(name))
```


```python
# tensorflow library가 label을 표현/사용하기위해 필요한 label map생성
# annotation 파일에 넣었던 레이블(.xml파일 > object > name)과 동일해야한다.

def make_labelmap(labels,i):    
    with open(ANNOTATION_PATH +'/level_{}/'.format(i)+ 'label_map.pbtxt', 'w') as f:
        for label in labels:
            f.write('item { \n')
            f.write('\tname:\'{}\'\n'.format(label['name']))
            f.write('\tid:{}\n'.format(label['id']))
            f.write('}\n')
```


```python
for i in range(1,5):
    make_labelmap(labels_all[i-1],i)
```

# 3. Create TF records

라벨링된 img, xml파일들과 labelmap을 담은 .pbtxt파일의 정보를 binary형태로 담은 record 파일을 단어 레벨별로 생성한다.


```python
# object detection api가 사용하는 representation of the data를 담고있는 record
!python {SCRIPTS_PATH + '/generate_tfrecord.py'} -x {IMAGE_PATH + '/level_1/train'} -l {ANNOTATION_PATH + '/level_1/label_map.pbtxt'} -o {ANNOTATION_PATH + '/level_1/train.record'}
!python {SCRIPTS_PATH + '/generate_tfrecord.py'} -x {IMAGE_PATH + '/level_1/test'} -l {ANNOTATION_PATH + '/level_1/label_map.pbtxt'} -o {ANNOTATION_PATH + '/level_1/test.record'}
```

# 4. Download TF Models Pretrained Models from Tensorflow Model Zoo

Tensorflow/models에 object detection api를 사용하여 model training을 진행하기위해 필요한것들이 담겨있다.
(PC에서 실행 하려면 3,4,5번 부터는 object detection api가 OS에 설치되어있어야한다.

object detection 설치안내 페이지:
https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/install.html)


```python
# Tensorflow 폴더안에 git clone하기위한 pythons script
!cd Tensorflow && git clone https://github.com/tensorflow/models
```

#### Model Zoo에서 사용할 model 선택

mAP와 속도를 기준으로 model zoo에서 해결하려는 문제에 적합한 model을 받아서 pretrained model로 사용할 수 있다. (적합한 model을 선택해서 클릭하면 다운로드시작됨)

model zoo github: https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf2_detection_zoo.md

**참고:**
model zoo에서 선택한 모델을 가져오는 코드(python script):


```python
wget.download('http://download.tensorflow.org/models/object_detection/tf2/20200711/ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8.tar.gz')
!mv ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8.tar.gz {PRETRAINED_MODEL_PATH}
!cd {PRETRAINED_MODEL_PATH} && tar -zxvf ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8.tar.gz
```

# 5. Copy Model Config to Training Folder

#### model config setup

config file은 model을 표현하는 내용을 담고있다.

object detection api를 사용해서 model training을 할땐 model config 파일을 기반으로 model을 build한다. 그래서 내가 pretrained model을 기반으로 새로 만들 model을 train 및 build하기위해 model zoo에서 가져온 model의 config 파일을 copy해온다.(파일명: pipeline.config) copy해온 config파일의 내용을 우리문제에 적합하게 되도록 수정 할 예정이다.

- copy from:

PRETRAINED_MODEL_PATH+'/ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8/pipeline.config'


- copy to:

MODEL_PATH+'/'+CUSTOM_MODEL_NAME


```python
CUSTOM_MODEL_NAME = 'my_ssd_mobnet' 
```


```python
!mkdir {'Tensorflow\workspace\models\\'+CUSTOM_MODEL_NAME}
!cp {PRETRAINED_MODEL_PATH+'/ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8/pipeline.config'} {MODEL_PATH+'/'+CUSTOM_MODEL_NAME}
```

# 6. Update Config For Transfer Learning

#### pipeline config 파일내의 내용 설명:

가져온 pretrained model의 config파일을 기반으로
사용자가 전이학습을 수행하기위해 
ouput을 결정짓는 몇가지 parameter와 input데이터를 넣어서 custom model을 만들 수 있는 config file을 작성할 수 있다.

이 config 파일안에 세부 내용을 tuning해서
전이학습에서 모델의 성능의 input data에 맞는 맞춤형 + 향상된 성능을
얻을 수 있다.

아래 코드는 config 파일내의 tuning할 수 있는 항목을 comment로 설명해두었다

자료: 
https://tarak-gopani.medium.com/grocery-item-detection-using-tensorflow-object-detection-api-1581fb5df6d6


```python
# google의 protobuf를 사용해서 config 파일의 주요 항목만 수정했다.
import tensorflow as tf
from object_detection.utils import config_util
from object_detection.protos import pipeline_pb2
from google.protobuf import text_format
```


```python
CONFIG_PATH = MODEL_PATH+'/'+CUSTOM_MODEL_NAME+'/pipeline.config'
```


```python
config = config_util.get_configs_from_pipeline_file(CONFIG_PATH)
```


```python
# python으로 config file 내용을 수정한다
pipeline_config = pipeline_pb2.TrainEvalPipelineConfig()
with tf.io.gfile.GFile(CONFIG_PATH, "r") as f:                                                                                                                                                                                                                     
    proto_str = f.read()                                                                                                                                                                                                                                          
    text_format.Merge(proto_str, pipeline_config)  
```


```python
# config file 수정내용(to-be):
# num_classes: 7 (7개의 수어단어)
# batch_size: 60 (H/W capacity에따라 지정. GPU를 사용한다면 더 큰 size 가능)
# 어디서부터 train시작할지: ckpt0 (pretrained model이 trained된 지점부터 우리 data로 train시작하겟다는 의미)
#                           model zoo에서 pretrained model을 가져올때 함께 담겨온다
# checkpoint type: detection
# label_map_path: annotation폴더에 있는 label map pbtxt파일
# tf_record_input: annotation폴더에 있는 train record 파일
# test data에 대해서도 동일하게 적용

pipeline_config.model.ssd.num_classes = 28
pipeline_config.train_config.batch_size = 60
pipeline_config.train_config.fine_tune_checkpoint = PRETRAINED_MODEL_PATH+'/ssd_mobilenet_v2_fpnlite_320x320_coco17_tpu-8/checkpoint/ckpt-0'
pipeline_config.train_config.fine_tune_checkpoint_type = "detection"
pipeline_config.train_input_reader.label_map_path= ANNOTATION_PATH + '/label_map.pbtxt'
pipeline_config.train_input_reader.tf_record_input_reader.input_path[:] = [ANNOTATION_PATH + '/train.record']
pipeline_config.eval_input_reader[0].label_map_path = ANNOTATION_PATH + '/label_map.pbtxt'
pipeline_config.eval_input_reader[0].tf_record_input_reader.input_path[:] = [ANNOTATION_PATH + '/test.record']
```


```python
# 위까지는 notebook 메모리내에서만 반영이 되어있다. 
# 실제 workspace/models/my_ssd_mobnet 디렉토리에있는 pipelineconfig 파일에 변경된 내용을 써야한다

# google.protobuf의 text_formt을 사용해서 작성할수있다.

config_text = text_format.MessageToString(pipeline_config)                                                                                                                                                                                                        
with tf.io.gfile.GFile(CONFIG_PATH, "wb") as f:                                                                                                                                                                                                                     
    f.write(config_text)   
```

# 7. Train the model

python shell script를 다음과 같이 string으로 확보하여 실행한다.

script 설명:
- model_main_tf2 = tensorflow version 2를 사용
- model_dir로 지정한 CUSTOM MODEL dir 지정
- pipeline config로 위에서 수정한 CUSTOM MODEL dir안의 config 파일 지정
- 마지막 num_train_steps를 지정. (5개의 단어를 detect하는 sign language detection model 예시를 보면 10,000steps 수준으로 지정함)


```python
print("""!python {}/research/object_detection/model_main_tf2.py --model_dir={}/{} --pipeline_config_path={}/{}/pipeline.config --num_train_steps=5000 --alsologtostderr""".format(APIMODEL_PATH, MODEL_PATH,CUSTOM_MODEL_NAME,MODEL_PATH,CUSTOM_MODEL_NAME))
```

# 8. Evaluate the model

AP(average precision), AR(average recall)값을 확인할 수 있다.

cocodataset의 detection metrics를 사용한다. 상세 정보:https://cocodataset.org/#detection-eval


```python
print("""!python {}/research/object_detection/model_main_tf2.py --model_dir={}/{} --pipeline_config_path={}/{}/pipeline.config --checkpoint_dir={}/{}""".format(APIMODEL_PATH, MODEL_PATH,CUSTOM_MODEL_NAME,MODEL_PATH,CUSTOM_MODEL_NAME, MODEL_PATH,CUSTOM_MODEL_NAME))
```

# 9. Load Train Model From Checkpoint

checkpoint 파일의 내용을 기반으로 훈련시킨 모델을 build할 수 있다.

about tensorflow checkpoints:
https://www.tensorflow.org/guide/checkpoint


```python
import os
from object_detection.utils import label_map_util
from object_detection.utils import visualization_utils as viz_utils
from object_detection.builders import model_builder
```


```python
# Load pipeline config and build a detection model
# 모델 구조를 불러와서 build
configs = config_util.get_configs_from_pipeline_file(CONFIG_PATH)
detection_model = model_builder.build(model_config=configs['model'], is_training=False)

# Restore checkpoint
# 모델구조에 넣기위해 checkpoint에서 가중치 값들 가져오기
# update x in ckpt-x to the latest ckpt
ckpt = tf.compat.v2.train.Checkpoint(model=detection_model)
ckpt.restore(os.path.join(CHECKPOINT_PATH, 'ckpt-11')).expect_partial()

# h5로 저장하는 방법


# 결과적으로 detection_model 변수에 h5형태로 저장할 수 있는 모델이 저장된다.
# detection_model =  모델 (구조 + 숫자)
# 추후, detection_model을 불러와서 predict를 수행하면됨.
@tf.function
def detect_fn(image):
    image, shapes = detection_model.preprocess(image)
    prediction_dict = detection_model.predict(image, shapes)
    detections = detection_model.postprocess(prediction_dict, shapes)
    return detections
```

# 10. Detect in Real-Time

opencv를 통해 실시간 영상에서 프레임을 object detection model로 분석하여 영상 속 동작에 해당하는 수어 단어를 detect할 수 있다. detect되는 box에 model이 detect한 단어의 확률도 함께 출력된다. 


```python
import cv2 
import numpy as np
import pandas as pd
```


```python
category_index = label_map_util.create_category_index_from_labelmap(ANNOTATION_PATH+'/level_4/label_map.pbtxt')
```


```python
# Setup capture
cap = cv2.VideoCapture(0)
width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
```


```python
while True: 
    ret, frame = cap.read()
    image_np = np.array(frame)
    
    # load background (could be an image too)
    #bk = np.full(frame.shape, 255, dtype=np.uint8)  # white bk
    
    input_tensor = tf.convert_to_tensor(np.expand_dims(image_np, 0), dtype=tf.float32)
    detections = detect_fn(input_tensor)
    
    num_detections = int(detections.pop('num_detections'))
    detections = {key: value[0, :num_detections].numpy() for key, value in detections.items()}
    detections['num_detections'] = num_detections

    # detection_classes should be ints.
    detections['detection_classes'] = detections['detection_classes'].astype(np.int64)

    label_id_offset = 1
    image_np_with_detections = image_np.copy()

    viz_utils.visualize_boxes_and_labels_on_image_array(
                image_np_with_detections,
                detections['detection_boxes'],
                detections['detection_classes']+label_id_offset,
                detections['detection_scores'],
                category_index,
                use_normalized_coordinates=True,
                max_boxes_to_draw=5,
                min_score_thresh=0.6, #threshold 조절 (아직 성능이 낮아서 20%로 낮춰야함)
                agnostic_mode=False)

    cv2.imshow('object detection',  cv2.resize(image_np_with_detections, (800, 600)))
    
    if cv2.waitKey(1) & 0xFF == ord('q'):
        cap.release()
        break
cv2.destroyAllWindows()
```
