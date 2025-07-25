# Projeto de Transfer Learning em Python 
O projeto consiste em aplicar o método de Transfer Learning em uma rede de Deep Learning na linguagem Python no ambiente COLAB. 

# 0. IMPORTAR MÓDULOS
```
import tensorflow as tf
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow.keras.applications import MobileNetV2
from tensorflow.keras.layers import Dense, GlobalAveragePooling2D
from tensorflow.keras.models import Model
from tensorflow.keras.optimizers import Adam
import os
```
# 1. PARÂMETROS
```
IMG_SIZE = (160, 160)  # MobileNetV2 padrão
BATCH_SIZE = 32
EPOCHS = 5  # Aumente para melhor acurácia
```
# 2. CARREGANDO O DATASET (usaremos o cats_vs_dogs do TFDS)
## Baixa automaticamente o dataset Cats vs Dogs (~800MB)
```
# Baixar e extrair no /content
dataset_path = tf.keras.utils.get_file(
    fname="cats_and_dogs_filtered.zip",
    origin="https://storage.googleapis.com/mledu-datasets/cats_and_dogs_filtered.zip",
    extract=True,
    cache_dir='/content'  # <<--- altera o diretório base
)

# Ajustando os diretórios corretamente
base_dir = os.path.join('/content', 'datasets', 'cats_and_dogs_filtered_extracted' ,'cats_and_dogs_filtered')
train_dir = os.path.join(base_dir, 'train')
val_dir = os.path.join(base_dir, 'validation')
```
# 3. PRÉ-PROCESSAMENTO DAS IMAGENS
```
train_datagen = ImageDataGenerator(
    rescale=1./255,
    rotation_range=20,
    width_shift_range=0.2,
    height_shift_range=0.2,
    shear_range=0.2,
    zoom_range=0.2,
    horizontal_flip=True,
    fill_mode='nearest'
)

val_datagen = ImageDataGenerator(rescale=1./255)

train_generator = train_datagen.flow_from_directory(
    train_dir,
    target_size=IMG_SIZE,
    batch_size=BATCH_SIZE,
    class_mode='binary'
)

val_generator = val_datagen.flow_from_directory(
    val_dir,
    target_size=IMG_SIZE,
    batch_size=BATCH_SIZE,
    class_mode='binary'
)
```
# 4. TRANSFER LEARNING COM MOBILENETV2
```
base_model = MobileNetV2(input_shape=IMG_SIZE + (3,),
                         include_top=False,
                         weights='imagenet')

# Congela as camadas base (transfer learning)
base_model.trainable = False

# Cabeçalho personalizado
x = base_model.output
x = GlobalAveragePooling2D()(x)
x = Dense(128, activation='relu')(x)
predictions = Dense(1, activation='sigmoid')(x)

model = Model(inputs=base_model.input, outputs=predictions)
```
# 5. COMPILAÇÃO E TREINAMENTO
```
model.compile(optimizer=Adam(learning_rate=0.0001),
              loss='binary_crossentropy',
              metrics=['accuracy'])

history = model.fit(
    train_generator,
    epochs=EPOCHS,
    validation_data=val_generator
)
```
# 6. SALVANDO O MODELO
```
model.save("cats_vs_dogs_mobilenetv2.keras")
print("Modelo salvo como cats_vs_dogs_mobilenetv2.keras")
```
# Usar o Modelo para Previsão
```
import tensorflow as tf
import numpy as np
from tensorflow.keras.preprocessing import image

# Carrega o modelo treinado
model = tf.keras.models.load_model("cats_vs_dogs_mobilenetv2.keras")

# Carrega uma imagem de teste
img_path = "1.jpg"  # Substitua pelo caminho da sua imagem
img = image.load_img(img_path, target_size=(160, 160))
img_array = image.img_to_array(img) / 255.0
img_array = np.expand_dims(img_array, axis=0)

# Faz a previsão
prediction = model.predict(img_array)[0][0]
if prediction > 0.5:
    print("É um CÃO 🐶 (confiança:", prediction, ")")
else:
    print("É um GATO 🐱 (confiança:", 1 - prediction, ")")
```