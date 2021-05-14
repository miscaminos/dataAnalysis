# Wisconsin Breast Cancer 예측

Binary classification이 가능한 모델:
 - decision tree
 - logistics regression
 - random forest
 - DNN, 등
 
최선의 예측 값을 찾기위해 여러 모델을 돌려보고 (for문 활용), 가장 성능이 좋은 모델을 선택한다.

### DNN 모델 만들기

### 전반적인 과정 순서:

#### 1. Identify what to predict based on what data: 

    input parameter 30개, target parameter('diagnosis') 1개 - data set 파악
    
    target 클래스 정의: 2가지- B for Benign(=0), M for Malignant(=1)
    
    binary classification -> 0과 1로 변환
    

#### 2. train data, test data 구분 (train data는 train & validation 으로 구분) 
    
    sklearn.model_selection의 train_test_split함수 사용


#### 3. data 정규화/표준화

    sklearn.preprocessing의 MinMaxScaler() 또는 StandardScaler() 사용
    
    MinMaxScaler: converts 값 into scale of (0,1)
   
    StandardScaler: converts to follow normal curve (정규분포, mean=0, std=1)


#### 4. DNN 모델 만들기

    how many hidden layers?
    
    how many parameters? (how many equations are enough to make the closest prediction?)
    
    과대적합(overfitting)을 억제하기위한 규제방법?
    
    
#### 5. 모델 훈련 (epoch. batch 설정) 및 훈련 곡선 확인 (검증 손실 그래프)

    train data로 훈련 시키고 validation data로 검증
    

#### 6. 훈련된 모델로 예측

    test data로 evaluate. 정확도 확인


```python
import pandas as pd
```


```python
df = pd.read_csv('./data/wisc_bc_data.csv')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>diagnosis</th>
      <th>radius_mean</th>
      <th>texture_mean</th>
      <th>perimeter_mean</th>
      <th>area_mean</th>
      <th>smoothness_mean</th>
      <th>compactness_mean</th>
      <th>concavity_mean</th>
      <th>points_mean</th>
      <th>...</th>
      <th>radius_worst</th>
      <th>texture_worst</th>
      <th>perimeter_worst</th>
      <th>area_worst</th>
      <th>smoothness_worst</th>
      <th>compactness_worst</th>
      <th>concavity_worst</th>
      <th>points_worst</th>
      <th>symmetry_worst</th>
      <th>dimension_worst</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>87139402</td>
      <td>B</td>
      <td>12.32</td>
      <td>12.39</td>
      <td>78.85</td>
      <td>464.1</td>
      <td>0.10280</td>
      <td>0.06981</td>
      <td>0.03987</td>
      <td>0.03700</td>
      <td>...</td>
      <td>13.50</td>
      <td>15.64</td>
      <td>86.97</td>
      <td>549.1</td>
      <td>0.1385</td>
      <td>0.1266</td>
      <td>0.12420</td>
      <td>0.09391</td>
      <td>0.2827</td>
      <td>0.06771</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8910251</td>
      <td>B</td>
      <td>10.60</td>
      <td>18.95</td>
      <td>69.28</td>
      <td>346.4</td>
      <td>0.09688</td>
      <td>0.11470</td>
      <td>0.06387</td>
      <td>0.02642</td>
      <td>...</td>
      <td>11.88</td>
      <td>22.94</td>
      <td>78.28</td>
      <td>424.8</td>
      <td>0.1213</td>
      <td>0.2515</td>
      <td>0.19160</td>
      <td>0.07926</td>
      <td>0.2940</td>
      <td>0.07587</td>
    </tr>
    <tr>
      <th>2</th>
      <td>905520</td>
      <td>B</td>
      <td>11.04</td>
      <td>16.83</td>
      <td>70.92</td>
      <td>373.2</td>
      <td>0.10770</td>
      <td>0.07804</td>
      <td>0.03046</td>
      <td>0.02480</td>
      <td>...</td>
      <td>12.41</td>
      <td>26.44</td>
      <td>79.93</td>
      <td>471.4</td>
      <td>0.1369</td>
      <td>0.1482</td>
      <td>0.10670</td>
      <td>0.07431</td>
      <td>0.2998</td>
      <td>0.07881</td>
    </tr>
    <tr>
      <th>3</th>
      <td>868871</td>
      <td>B</td>
      <td>11.28</td>
      <td>13.39</td>
      <td>73.00</td>
      <td>384.8</td>
      <td>0.11640</td>
      <td>0.11360</td>
      <td>0.04635</td>
      <td>0.04796</td>
      <td>...</td>
      <td>11.92</td>
      <td>15.77</td>
      <td>76.53</td>
      <td>434.0</td>
      <td>0.1367</td>
      <td>0.1822</td>
      <td>0.08669</td>
      <td>0.08611</td>
      <td>0.2102</td>
      <td>0.06784</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9012568</td>
      <td>B</td>
      <td>15.19</td>
      <td>13.21</td>
      <td>97.65</td>
      <td>711.8</td>
      <td>0.07963</td>
      <td>0.06934</td>
      <td>0.03393</td>
      <td>0.02657</td>
      <td>...</td>
      <td>16.20</td>
      <td>15.73</td>
      <td>104.50</td>
      <td>819.1</td>
      <td>0.1126</td>
      <td>0.1737</td>
      <td>0.13620</td>
      <td>0.08178</td>
      <td>0.2487</td>
      <td>0.06766</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 32 columns</p>
</div>




```python
df.shape
```




    (569, 32)




```python
df.columns
```




    Index(['id', 'diagnosis', 'radius_mean', 'texture_mean', 'perimeter_mean',
           'area_mean', 'smoothness_mean', 'compactness_mean', 'concavity_mean',
           'points_mean', 'symmetry_mean', 'dimension_mean', 'radius_se',
           'texture_se', 'perimeter_se', 'area_se', 'smoothness_se',
           'compactness_se', 'concavity_se', 'points_se', 'symmetry_se',
           'dimension_se', 'radius_worst', 'texture_worst', 'perimeter_worst',
           'area_worst', 'smoothness_worst', 'compactness_worst',
           'concavity_worst', 'points_worst', 'symmetry_worst', 'dimension_worst'],
          dtype='object')




```python
df['diagnosis'] = df['diagnosis'].str.replace("B",'0')
df['diagnosis'] = df['diagnosis'].str.replace("M",'1')
```


```python
df_target = df['diagnosis'].astype(int)
len(df_target)
```




    569




```python
df_input = df.copy()
df_input = df_input.drop(['diagnosis','id'], axis=1)
```


```python
df_input.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>radius_mean</th>
      <th>texture_mean</th>
      <th>perimeter_mean</th>
      <th>area_mean</th>
      <th>smoothness_mean</th>
      <th>compactness_mean</th>
      <th>concavity_mean</th>
      <th>points_mean</th>
      <th>symmetry_mean</th>
      <th>dimension_mean</th>
      <th>...</th>
      <th>radius_worst</th>
      <th>texture_worst</th>
      <th>perimeter_worst</th>
      <th>area_worst</th>
      <th>smoothness_worst</th>
      <th>compactness_worst</th>
      <th>concavity_worst</th>
      <th>points_worst</th>
      <th>symmetry_worst</th>
      <th>dimension_worst</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>12.32</td>
      <td>12.39</td>
      <td>78.85</td>
      <td>464.1</td>
      <td>0.10280</td>
      <td>0.06981</td>
      <td>0.03987</td>
      <td>0.03700</td>
      <td>0.1959</td>
      <td>0.05955</td>
      <td>...</td>
      <td>13.50</td>
      <td>15.64</td>
      <td>86.97</td>
      <td>549.1</td>
      <td>0.1385</td>
      <td>0.1266</td>
      <td>0.12420</td>
      <td>0.09391</td>
      <td>0.2827</td>
      <td>0.06771</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10.60</td>
      <td>18.95</td>
      <td>69.28</td>
      <td>346.4</td>
      <td>0.09688</td>
      <td>0.11470</td>
      <td>0.06387</td>
      <td>0.02642</td>
      <td>0.1922</td>
      <td>0.06491</td>
      <td>...</td>
      <td>11.88</td>
      <td>22.94</td>
      <td>78.28</td>
      <td>424.8</td>
      <td>0.1213</td>
      <td>0.2515</td>
      <td>0.19160</td>
      <td>0.07926</td>
      <td>0.2940</td>
      <td>0.07587</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11.04</td>
      <td>16.83</td>
      <td>70.92</td>
      <td>373.2</td>
      <td>0.10770</td>
      <td>0.07804</td>
      <td>0.03046</td>
      <td>0.02480</td>
      <td>0.1714</td>
      <td>0.06340</td>
      <td>...</td>
      <td>12.41</td>
      <td>26.44</td>
      <td>79.93</td>
      <td>471.4</td>
      <td>0.1369</td>
      <td>0.1482</td>
      <td>0.10670</td>
      <td>0.07431</td>
      <td>0.2998</td>
      <td>0.07881</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11.28</td>
      <td>13.39</td>
      <td>73.00</td>
      <td>384.8</td>
      <td>0.11640</td>
      <td>0.11360</td>
      <td>0.04635</td>
      <td>0.04796</td>
      <td>0.1771</td>
      <td>0.06072</td>
      <td>...</td>
      <td>11.92</td>
      <td>15.77</td>
      <td>76.53</td>
      <td>434.0</td>
      <td>0.1367</td>
      <td>0.1822</td>
      <td>0.08669</td>
      <td>0.08611</td>
      <td>0.2102</td>
      <td>0.06784</td>
    </tr>
    <tr>
      <th>4</th>
      <td>15.19</td>
      <td>13.21</td>
      <td>97.65</td>
      <td>711.8</td>
      <td>0.07963</td>
      <td>0.06934</td>
      <td>0.03393</td>
      <td>0.02657</td>
      <td>0.1721</td>
      <td>0.05544</td>
      <td>...</td>
      <td>16.20</td>
      <td>15.73</td>
      <td>104.50</td>
      <td>819.1</td>
      <td>0.1126</td>
      <td>0.1737</td>
      <td>0.13620</td>
      <td>0.08178</td>
      <td>0.2487</td>
      <td>0.06766</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 30 columns</p>
</div>




```python
# 0(음성): 350수준, 1(양성):210수준
df_target.value_counts()
```




    0    357
    1    212
    Name: diagnosis, dtype: int64




```python
350/210 #음성이 양성의 대략 1.5배로 target값들이 구성되어있음.
```




    1.6666666666666667




```python
#훈련, 테스트 데이터 split
from sklearn.model_selection import train_test_split

train_input, test_input, train_target, test_target = train_test_split(
                                                df_input, df_target, stratify=df_target, random_state=42)
```


```python
train_input.shape, train_target.shape
```




    ((426, 30), (426,))




```python
test_input.shape, test_target.shape
```




    ((143, 30), (143,))




```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 569 entries, 0 to 568
    Data columns (total 32 columns):
     #   Column             Non-Null Count  Dtype  
    ---  ------             --------------  -----  
     0   id                 569 non-null    int64  
     1   diagnosis          569 non-null    object 
     2   radius_mean        569 non-null    float64
     3   texture_mean       569 non-null    float64
     4   perimeter_mean     569 non-null    float64
     5   area_mean          569 non-null    float64
     6   smoothness_mean    569 non-null    float64
     7   compactness_mean   569 non-null    float64
     8   concavity_mean     569 non-null    float64
     9   points_mean        569 non-null    float64
     10  symmetry_mean      569 non-null    float64
     11  dimension_mean     569 non-null    float64
     12  radius_se          569 non-null    float64
     13  texture_se         569 non-null    float64
     14  perimeter_se       569 non-null    float64
     15  area_se            569 non-null    float64
     16  smoothness_se      569 non-null    float64
     17  compactness_se     569 non-null    float64
     18  concavity_se       569 non-null    float64
     19  points_se          569 non-null    float64
     20  symmetry_se        569 non-null    float64
     21  dimension_se       569 non-null    float64
     22  radius_worst       569 non-null    float64
     23  texture_worst      569 non-null    float64
     24  perimeter_worst    569 non-null    float64
     25  area_worst         569 non-null    float64
     26  smoothness_worst   569 non-null    float64
     27  compactness_worst  569 non-null    float64
     28  concavity_worst    569 non-null    float64
     29  points_worst       569 non-null    float64
     30  symmetry_worst     569 non-null    float64
     31  dimension_worst    569 non-null    float64
    dtypes: float64(30), int64(1), object(1)
    memory usage: 142.4+ KB
    


```python
#input 데이터 표준화 (각 column값이 다른 scale이기때문에)
from sklearn.preprocessing import StandardScaler

ss = StandardScaler()

ss.fit(train_input)

train_scaled = ss.transform(train_input)
test_scaled = ss.transform(test_input)
```


```python
# 클래스 2가지 - B:음성, M:양성
df['diagnosis'].unique()
```




    array(['0', '1'], dtype=object)




```python
class_name = set(df['diagnosis'])
class_name
```




    {'0', '1'}




```python
#model만들기 - 방법1

import tensorflow as tf
from tensorflow import keras
import numpy as np

model1 = keras.models.Sequential()
model1.add(keras.Input(shape=(30,)))
model1.add(keras.layers.Dense(60, activation="relu"))
model1.add(keras.layers.Dense(40, activation="relu"))
model1.add(keras.layers.Dense(20, activation="relu"))
model1.add(keras.layers.Dense(2, activation="softmax"))

# sgd 확률적 평가 하강법 사용 - sgd사용. 
# adam을 사용하면 성능 더 좋음.
model1.compile(loss="sparse_categorical_crossentropy",
              optimizer="sgd",
              metrics=["accuracy"])
```


```python
model1.summary()
```

    Model: "sequential_4"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_14 (Dense)             (None, 50)                1550      
    _________________________________________________________________
    dense_15 (Dense)             (None, 40)                2040      
    _________________________________________________________________
    dense_16 (Dense)             (None, 20)                820       
    _________________________________________________________________
    dense_17 (Dense)             (None, 2)                 42        
    =================================================================
    Total params: 4,452
    Trainable params: 4,452
    Non-trainable params: 0
    _________________________________________________________________
    


```python
#model 만들기 - 방법2

model2 = tf.keras.models.Sequential([
      tf.keras.layers.Flatten(input_shape=(30,)),
      tf.keras.layers.Dense(60, activation='relu'),
      tf.keras.layers.Dropout(0.3),
      tf.keras.layers.Dense(30, activation='relu'),
      tf.keras.layers.Dense(2, activation='softmax')
      ])

#optimizer = tf.keras.optimizers.RMSprop(0.001)

model2.compile(loss='sparse_categorical_crossentropy',
            optimizer='adam',
            metrics=['accuracy'])
```


```python
model2.summary()
```

    Model: "sequential_5"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    flatten (Flatten)            (None, 30)                0         
    _________________________________________________________________
    dense_18 (Dense)             (None, 60)                1860      
    _________________________________________________________________
    dropout (Dropout)            (None, 60)                0         
    _________________________________________________________________
    dense_19 (Dense)             (None, 30)                1830      
    _________________________________________________________________
    dense_20 (Dense)             (None, 2)                 62        
    =================================================================
    Total params: 3,752
    Trainable params: 3,752
    Non-trainable params: 0
    _________________________________________________________________
    


```python
#model 만들기 - 방법3
# binary classification이기때문에, loss = binary cross entropy사용

model3 = tf.keras.models.Sequential([
      tf.keras.layers.Flatten(input_shape=(30,)),
      tf.keras.layers.Dense(60, activation='relu'),
      tf.keras.layers.Dropout(0.3),
      tf.keras.layers.Dense(30, activation='sigmoid'),
      tf.keras.layers.Dense(1, activation='sigmoid')
      ])

#optimizer = tf.keras.optimizers.RMSprop(0.001)

model3.compile(loss='binary_crossentropy',
            optimizer='adam',
            metrics=['accuracy'])
```


```python
model3.summary()
```

    Model: "sequential_6"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    flatten_1 (Flatten)          (None, 30)                0         
    _________________________________________________________________
    dense_21 (Dense)             (None, 60)                1860      
    _________________________________________________________________
    dropout_1 (Dropout)          (None, 60)                0         
    _________________________________________________________________
    dense_22 (Dense)             (None, 30)                1830      
    _________________________________________________________________
    dense_23 (Dense)             (None, 1)                 31        
    =================================================================
    Total params: 3,721
    Trainable params: 3,721
    Non-trainable params: 0
    _________________________________________________________________
    


```python
#이거는 이미지 classification에서 왜 한거지?
keras.backend.clear_session()
np.random.seed(42)
tf.random.set_seed(42)
```


```python
model1.layers
```




    [<tensorflow.python.keras.layers.core.Dense at 0x1747f572208>,
     <tensorflow.python.keras.layers.core.Dense at 0x1747f568608>,
     <tensorflow.python.keras.layers.core.Dense at 0x1747f59f088>,
     <tensorflow.python.keras.layers.core.Dense at 0x1747f5b1ec8>]




```python
hidden1 = model1.layers[1]
hidden1.name
```




    'dense_15'




```python
weights, biases = hidden1.get_weights()
```


```python
weights
```


```python
biases
```


```python
len(train_scaled)
```


```python
#전체 훈련 세트를 검증 세트와 훈련 세트로 나누기 (x)

valid_X, train_X = train_scaled[:200], train_scaled[200:]
valid_y, train_y = train_target[:200], train_target[200:]

# ==> 이렇게 임의로 앞200개와 나머지로 나누기보다는 train_test_split 함수를 사용하는것이 옳다.
```


```python
#전체 훈련 세트를 검증 세트와 훈련 세트로 나누기 (O)

train_sc_input, valid_input, train_sc_target, valid_target = train_test_split(
                                train_scaled, train_target, stratify=train_target, 
                                test_size=0.25, random_state=42)
```


```python
history = model1.fit(train_sc_input, train_sc_target, epochs=30,
                    validation_data=(valid_input, valid_target), verbose=0)
```


```python
history.history.keys()
```




    dict_keys(['loss', 'accuracy', 'val_loss', 'val_accuracy'])




```python
import pandas as pd
%matplotlib inline
import matplotlib as mpl
import matplotlib.pyplot as plt

pd.DataFrame(history.history).plot(figsize=(8, 5))
plt.grid(True)
plt.gca().set_ylim(0, 1)
plt.show()
```


    
![png](output_37_0.png)
    



```python
model1.evaluate(test_scaled, test_target)
```

    5/5 [==============================] - 0s 4ms/step - loss: 0.1150 - accuracy: 0.9790
    




    [0.11499959975481033, 0.9790209531784058]




```python
X_new = test_scaled[:3]
```


```python
X_new
```


```python
y_proba = model1.predict(X_new)
```


```python
y_proba.round(3)
```




    array([[0.   , 1.   ],
           [0.967, 0.033],
           [0.002, 0.998]], dtype=float32)


