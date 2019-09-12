---
layout: post
title: "Tensorflow-GAN: MNIST데이터셋으로 GAN 구현해보기"
date: 2019-09-09 00:00:00
excerpt: "MNIST 데이터셋과 Tensorflow를 이용한 GAN 구현"  
tags:
- Study
categories:
- 공부
---
**이 글은 [Generative Adversial Networks using Tensorflow](https://towardsdatascience.com/generative-adversarial-networks-using-tensorflow-c8f4518406df)를 번역한 글입니다.**  
  
    
    
가장 우선, 필요한 라이브러리들과 MNIST 데이터셋을 불러옵니다.


```
import tensorflow as tf
import numpy as np
import matplotlib.pyplot as plt
from tensorflow.examples.tutorials.mnist import input_data

mnist=input_data.read_data_sets("MNIST_data")
```

다음으로, 2개의 Network(Discriminator, Generator)를 나타내는 함수를 만듭니다. 각 Network는 총 3개의 FC Layer로 구성되어 있습니다.


```
def generator(z, reuse=None):
    with tf.variable_scope('gen',reuse=reuse):
        hidden1=tf.layers.dense(inputs=z,units=128,activation=tf.nn.leaky_relu)
        hidden2=tf.layers.dense(inputs=hidden1,units=128,activation=tf.nn.leaky_relu)
        output=tf.layers.dense(inputs=hidden2,units=784,activation=tf.nn.tanh)
        
        return output  
```


```
def discriminator(img, reuse=None):
    with tf.variable_scope('dis',reuse=reuse):
        hidden1=tf.layers.dense(inputs=img,units=128,activation=tf.nn.leaky_relu)
        hidden2=tf.layers.dense(inputs=hidden1,units=128,activation=tf.nn.leaky_relu)
        logits=tf.layers.dense(hidden2,units=1)
        output=tf.sigmoid(logits)
        
        return output,logits
```

이제 train 이미지와 random noise를 담을 placeholder들을 생성해줍니다.
*train_img* 은 MNIST에서 불러온 데이터를 담을 placeholder이고,  *z*는 크기가 100인 random-noise 입니다.  
이후 *generator*와 *discriminator*를 호출합니다.
*discriminator*가 작동하기 위해서는, 실제 이미지가 어떤지 알아야 하기 때문에 *train_img*와 *z*에 대해 각각 호출합니다. *discriminator* 내부에서 같은 변수 호출로 인해 오류가 발생할 가능성이 있으므로  **reuse=True**로 두고 함수를 호출합니다.


```
tf.reset_default_graph()

train_img = tf.placeholder(tf.float32, [None, 784])
z = tf.placeholder(tf.float32, [None, 100])

G = generator(z)
D_output_real,D_logits_real=discriminator(train_img)
D_output_fake,D_logits_fake=discriminator(G,reuse=True)
```

이제 loss function을 정의합니다. **loss_func**의 labels_in 파라미터는 loss function에 target label을 제공합니다.
**D_real_loss**의 두번째 파라미터는 discriminator가 generator를 overpowering하는것을 막기위해 0.9로 설정하였습니다.


```
def loss_func(logits_in,labels_in):
    return tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(logits=logits_in,labels=labels_in))

D_real_loss=loss_func(D_logits_real,tf.ones_like(D_logits_real)*0.9) #Smoothing for generalization
D_fake_loss=loss_func(D_logits_fake,tf.zeros_like(D_logits_real))
D_loss=D_real_loss+D_fake_loss

G_loss= loss_func(D_logits_fake,tf.ones_like(D_logits_fake))
```

이제 hyperparameter 등을 설정 후 그래프를 실행합니다.

여기서는 z[0]값의 변화에 따른 generator의 변화를 보기 위해
50번 epoch 학습 후 z[0]값을 달리하며 이미지를 생성했습니다.


```
lr=0.001

#Do this when multiple networks interact with each other
tvars=tf.trainable_variables()  #returns all variables created(the two variable scopes) and makes trainable true
d_vars=[var for var in tvars if 'dis' in var.name]
g_vars=[var for var in tvars if 'gen' in var.name]

D_trainer=tf.train.AdamOptimizer(lr).minimize(D_loss,var_list=d_vars)
G_trainer=tf.train.AdamOptimizer(lr).minimize(G_loss,var_list=g_vars)

batch_size=100
epochs=100
init=tf.global_variables_initializer()
```


```
samples=[] #generator examples
epochs=50
with tf.Session() as sess:
    sess.run(init)
    for epoch in range(epochs):
        num_batches=mnist.train.num_examples//batch_size
        for i in range(num_batches):
            batch=mnist.train.next_batch(batch_size)
            batch_images=batch[0].reshape((batch_size,784))
            batch_images=batch_images*2-1
            batch_z=np.random.normal(size=[batch_size,100])
            
            _=sess.run(D_trainer,feed_dict={train_img:batch_images,z:batch_z})
            _=sess.run(G_trainer,feed_dict={z:batch_z})
            
        print("on epoch{}".format(epoch))
        
        
    for i in range(100):
      sample_z = np.random.normal(size=[1,100])
      sample_z[0][0] = 0.02*i - 1 
      _ = sess.run([generator(z, reuse=True)], feed_dict={z:sample_z})
      plt.imshow(_.reshape(10,10))
      plt.show()

```


    ---------------------------------------------------------------------------


### References
1. [Generative Adversial Networks using Tensorflow](https://towardsdatascience.com/generative-adversarial-networks-using-tensorflow-c8f4518406df)