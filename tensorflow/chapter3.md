# Chapter 3


## ch3.1

### 텐서와 그래프 실행

#### 텐서플로 라이브러리 임포트

#### tf.contant로 상수를 변수에 저장


```python
import tensorflow.compat.v1 as tf
tf.disable_v2_behavior()
```

    WARNING:tensorflow:From c:\users\jiyun lee\appdata\local\programs\python\python37\lib\site-packages\tensorflow_core\python\compat\v2_compat.py:65: disable_resource_variables (from tensorflow.python.ops.variable_scope) is deprecated and will be removed in a future version.
    Instructions for updating:
    non-resource variables are not supported in the long term
    


```python
hello = tf.constant('Hello, TensorFlow!')
print(hello)
```

    Tensor("Const:0", shape=(), dtype=string)
    

#### +) hello 변수의 값을 출력한 결과로 hello가 텐서 플로의 Tensor라는 자료형이며 상수를 담고 있음을 의미


```python
a=tf.constant(10)
b=tf.constant(32)
c= tf.add(a,b)
print(c)
```

    Tensor("Add:0", shape=(), dtype=int32)
    

#### +) 변수가 Tensor 자료형이며 int 를 담고 있음

#### 그래프 실행은 Session 안에서 이루어지며 Session 객체와 run 메서드를 이용


```python
sess = tf.Session()
```


```python
print(sess.run(hello))
print(sess.run([a,b,c]))
```

    b'Hello, TensorFlow!'
    [10, 32, 42]
    


```python
sess.close()
```


## ch3.2

### placeholder and variable

#### 플레이스홀더는 그래프에 사용할 입력값을 나중에 받기 위해 사용하는 매개변수


```python
X = tf.placeholder(tf.float32, [None,3])
print(X)
```

    Tensor("Placeholder:0", shape=(?, 3), dtype=float32)
    

#### +) placeholder 라는 (?,3) 모양의 float32 자료형을 가진 텐서가 생성됨
#### +) 텐서 모양을 (?, 3)으로 정의했으므로, 두 번째 차원은 요소를 3개씩 가지고 있어야 한다.

#### 변수는 그래프를 최적화하는 용도로 텐서플로가 학습한 결과를 갱신하기 위해 사용하는 변수이다. 이 변수의 값들이 신경망의 성능을 좌우하게 된다.


```python
x_data = [[1,2,3],[4,5,6]]
```


```python
W = tf.Variable(tf.random_normal([3,2]))
b = tf.Variable(tf.random_normal([2,1]))
```

#### +) 정규분포의 무작위 값으로 초기화


```python
# W = tf.Variable([[0.1,0.1],[0.2,0.2],[0.3,0.3]])
```


```python
expr = tf.matmul(X,W) + b
```

#### +) X와 W 가 행렬임으로 tf.matnul 사용, 행렬이 아닌 경우 \*, tf.mul 사용

#### tf.global_variables_initializer는 압에서 정의한 변수들을 초기화하는 함수


```python
sess = tf.Session()
sess.run(tf.global_variables_initializer())
```


```python
print("=== x_data ===")
print(x_data)
print("=== W ===")
print(sess.run(W))
print("=== b ===")
print(sess.run(b))
print("=== expr ===")
print(sess.run(expr,feed_dict={X: x_data}))
```

    === x_data ===
    [[1, 2, 3], [4, 5, 6]]
    === W ===
    [[ 0.5738788  -1.7276537 ]
     [ 0.37379605 -0.24920726]
     [-0.37167853  0.49387127]]
    === b ===
    [[ 1.0353996 ]
     [-0.25780025]]
    === expr ===
    [[ 1.2418349  0.2909451]
     [ 1.6766241 -5.451224 ]]
    


```python
sess.close()
```


## ch3.3

### 선형 회귀 모델 구현하기


```python
x_data = [1,2,3]
y_data = [1,2,3]
```

#### +) 위 설정과 같은 함수를 선형 회귀 (주어진 x와 y 값을 가지고 서로간의 관계를 파악하는 것)하여 다름 입력에 대한 출력값을 예측

#### tf.random_uniform은 지정한 범위 사이의 균등균포를 가진 무작위 값으로 초기화


```python
W = tf.Variable(tf.random_uniform([1],-1.0,1.0))
b = tf.Variable(tf.random_uniform([1],-1.0,1.0))
```


```python
X = tf.placeholder(tf.float32, name="X")
Y = tf.placeholder(tf.float32, name="Y")
```


```python
print(X)
print(Y)
```

    Tensor("X:0", dtype=float32)
    Tensor("Y:0", dtype=float32)
    

#### X와 Y의 상관관계 (여기서는 상관관계)를 분석하기 위한 수식을 작성


```python
hypothesis = W * X + b
```

#### +) X가 주어졌을 때 W의 곱과 b의 합으로 Y를 만들수 있는 W와 b를 찾아내기 위함 (W는 가중치, b는 편향)

#### 손실함수 loss function = 한 쌍(X,Y)의 데이터에 대한 손실값을 계산하는 함수, 손실을 전체 데이터에 대해 구한 경우 이를 비용cost라고 한다


```python
cost = tf.reduce_mean(tf.square(hypothesis - Y))
```

#### tf.train.GradientDescentOptimizer : 텐서플로가 기본 제공하는 경사하강법 최적화 함수 이용하여 손실값을 최소화 하는 연산 그래프 생성

#### learning_rate : 학습률은 학습을 얼마나 급하게 할 것인가 설정, 값이 크면 최적의 손실값을 찾지 못하고 값이 작으면 학습속도가 느려짐 이처럼 학습에 영향을 주는 변수를 "하이퍼파라미터"라고 한다.


```python
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1)
train_op = optimizer.minimize(cost)
```


```python
sess = tf.Session()
sess.run(tf.global_variables_initializer())
```

#### train_op는 최적화를 수행하는 그래프, feed_dict 는 x_data, y_data를 전달하기 위한 매개변수


```python
for step in range(100):
    _,cost_val = sess.run([train_op, cost], feed_dict={X: x_data, Y: y_data})
    print(step, cost_val,sess.run(W), sess.run(b))
```

    0 0.90168256 [0.8761389] [0.38973105]
    1 0.030394228 [0.8358502] [0.36132926]
    2 0.019054396 [0.844525] [0.35472333]
    3 0.018031068 [0.84774566] [0.34596866]
    4 0.017173192 [0.85146224] [0.33767664]
    5 0.01635745 [0.85502684] [0.32955644]
    6 0.015580434 [0.8585125] [0.3216344]
    7 0.014840362 [0.86191374] [0.3139025]
    8 0.014135424 [0.8652332] [0.3063565]
    9 0.013463981 [0.868473] [0.29899192]
    10 0.012824443 [0.8716348] [0.29180434]
    11 0.012215269 [0.8747206] [0.28478956]
    12 0.011635032 [0.8777322] [0.27794343]
    13 0.011082367 [0.88067144] [0.27126187]
    14 0.010555938 [0.88354003] [0.2647409]
    15 0.010054532 [0.88633966] [0.25837672]
    16 0.00957693 [0.889072] [0.25216553]
    17 0.009122006 [0.8917386] [0.24610361]
    18 0.008688718 [0.8943412] [0.24018747]
    19 0.008275998 [0.8968811] [0.2344135]
    20 0.007882872 [0.89935994] [0.22877835]
    21 0.007508435 [0.9017793] [0.2232787]
    22 0.007151766 [0.9041404] [0.21791123]
    23 0.0068120603 [0.9064449] [0.21267283]
    24 0.0064884988 [0.9086939] [0.20756032]
    25 0.0061802715 [0.9108888] [0.20257068]
    26 0.005886704 [0.913031] [0.19770102]
    27 0.005607089 [0.9151217] [0.19294845]
    28 0.005340745 [0.91716206] [0.18831009]
    29 0.0050870464 [0.9191534] [0.18378323]
    30 0.0048454176 [0.9210969] [0.17936523]
    31 0.004615258 [0.9229937] [0.17505342]
    32 0.00439603 [0.92484486] [0.17084524]
    33 0.004187212 [0.92665154] [0.16673823]
    34 0.0039883126 [0.9284148] [0.16272996]
    35 0.003798871 [0.93013567] [0.15881804]
    36 0.0036184269 [0.93181515] [0.15500017]
    37 0.0034465452 [0.9334543] [0.15127407]
    38 0.003282833 [0.935054] [0.14763756]
    39 0.0031268923 [0.9366152] [0.14408845]
    40 0.0029783659 [0.93813896] [0.14062466]
    41 0.0028368868 [0.93962604] [0.13724414]
    42 0.0027021319 [0.9410774] [0.13394488]
    43 0.0025737823 [0.94249386] [0.13072494]
    44 0.00245153 [0.94387627] [0.1275824]
    45 0.0023350788 [0.9452255] [0.12451542]
    46 0.0022241557 [0.94654214] [0.12152213]
    47 0.0021185083 [0.9478273] [0.11860085]
    48 0.0020178838 [0.9490815] [0.11574975]
    49 0.0019220245 [0.9503055] [0.1129672]
    50 0.0018307259 [0.9515001] [0.11025155]
    51 0.0017437706 [0.9526661] [0.1076012]
    52 0.0016609394 [0.9538039] [0.10501451]
    53 0.0015820465 [0.95491445] [0.10249005]
    54 0.001506899 [0.95599836] [0.10002627]
    55 0.0014353157 [0.95705605] [0.09762167]
    56 0.0013671368 [0.9580884] [0.09527492]
    57 0.0013021968 [0.9590959] [0.09298457]
    58 0.0012403422 [0.9600792] [0.09074929]
    59 0.0011814241 [0.9610389] [0.08856776]
    60 0.0011253074 [0.96197546] [0.08643865]
    61 0.0010718559 [0.9628896] [0.08436074]
    62 0.0010209399 [0.96378165] [0.08233275]
    63 0.0009724426 [0.9646523] [0.08035352]
    64 0.00092625216 [0.9655021] [0.0784219]
    65 0.0008822551 [0.96633136] [0.07653669]
    66 0.00084034866 [0.9671408] [0.07469682]
    67 0.00080042746 [0.9679307] [0.07290114]
    68 0.0007624095 [0.9687016] [0.07114863]
    69 0.0007261937 [0.969454] [0.06943826]
    70 0.00069170125 [0.9701883] [0.06776901]
    71 0.0006588418 [0.97090495] [0.06613989]
    72 0.0006275462 [0.97160435] [0.06454992]
    73 0.0005977382 [0.972287] [0.06299819]
    74 0.00056934374 [0.97295314] [0.06148374]
    75 0.0005422998 [0.97360337] [0.06000572]
    76 0.0005165386 [0.9742379] [0.05856323]
    77 0.0004920051 [0.9748572] [0.05715542]
    78 0.00046863538 [0.97546166] [0.05578147]
    79 0.00044637534 [0.97605157] [0.05444052]
    80 0.00042517073 [0.97662723] [0.0531318]
    81 0.00040497686 [0.9771891] [0.05185455]
    82 0.00038573693 [0.9777374] [0.05060798]
    83 0.00036741715 [0.9782727] [0.04939142]
    84 0.00034996183 [0.97879493] [0.04820405]
    85 0.00033334154 [0.9793048] [0.04704529]
    86 0.00031750472 [0.97980225] [0.04591433]
    87 0.00030242125 [0.98028773] [0.04481055]
    88 0.00028805996 [0.98076165] [0.04373334]
    89 0.00027437412 [0.98122406] [0.04268201]
    90 0.0002613433 [0.9816755] [0.04165599]
    91 0.00024892946 [0.982116] [0.04065459]
    92 0.00023710268 [0.98254585] [0.03967727]
    93 0.0002258406 [0.98296547] [0.03872346]
    94 0.00021511367 [0.98337495] [0.03779258]
    95 0.00020489586 [0.9837746] [0.03688408]
    96 0.00019516458 [0.98416466] [0.03599741]
    97 0.00018589404 [0.98454535] [0.03513207]
    98 0.00017706097 [0.9849168] [0.03428749]
    99 0.0001686508 [0.98527944] [0.03346327]
    

#### +) -회 cost [W] [b]


```python
print("X: 5, Y:",sess.run(hypothesis, feed_dict={X: 5}))
print("X: 2.5, Y:", sess.run(hypothesis, feed_dict={X: 2.5}))
sess.close()
```

    X: 5, Y: [4.959861]
    X: 2.5, Y: [2.496662]
    

#### X = Y인 함수를 주어진 값들로 예측하여 입력이 5일때 4.959861이라는 근사값을 출력
