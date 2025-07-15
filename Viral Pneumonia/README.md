# Chest X-Ray Pneumonia Detection

Проект направлен на разработку и обучение сверточной нейронной сети (CNN) для автоматической диагностики пневмонии по рентгеновским снимкам грудной клетки.

## 📁 Структура датасета

Данные были разделены на три подмножества:
- **Train** — 6000 снимков без пневмонии и 800 снимков с пневмонией
- **Validation** — 2000 снимков без пневмонии и 200 с пневмонией
- **Test** — 2192 снимка без пневмонии и 345 с пневмонией

Файлы с некорректными изображениями автоматически удалялись перед обучением.

## 🧠 Архитектура нейронной сети

Модель сверточной нейронной сети (CNN) реализована с использованием Keras и имеет следующую архитектуру:

- `Conv2D` (32 фильтра, размер ядра 3x3, ReLU, L2-регуляризация)
- `MaxPooling2D` (2x2)
- `Conv2D` (64 фильтра, ReLU, L2)
- `MaxPooling2D`
- `Conv2D` (64 фильтра, ReLU, L2=0.01)
- `MaxPooling2D`
- `Conv2D` (128 фильтров, ReLU, L2=0.01)
- `MaxPooling2D`
- `Conv2D` (128 фильтров, ReLU, L2=0.001)
- `MaxPooling2D`
- `Dropout` (0.5)
- `Flatten`
- `Dense` (512 нейронов, ReLU)
- `Dense` (1 нейрон, сигмоида — бинарная классификация)

Функция потерь: `binary_crossentropy`  
Оптимизатор: `Adam`  
Метрика: `accuracy`

## ⚙️ Аугментация данных

Для повышения обобщающей способности модели использовалась агрессивная аугментация:
- Вращение, масштабирование, сдвиги, отражение
- Яркость, сдвиг каналов
- `fill_mode='reflect'`

## 🔁 Обучение

Обучение модели происходило в два этапа:

1. **Базовое обучение**
   ```python
   model.fit(
       train_generator,
       steps_per_epoch=340,
       epochs=50,
       validation_data=validation_generator,
       validation_steps=110
   )
2. **Финальное дообучение с объединенным генератором и callbacks**
    ```python
    callbacks_final = [
    EarlyStopping(
        monitor='loss',
        patience=5,
        restore_best_weights=True
    ),
    ReduceLROnPlateau(
        monitor='loss',
        factor=0.5,
        patience=2,
        min_lr=1e-6
    )
    ]

    merged_gen = combined_generator(train_generator, validation_generator)

    total_steps = train_generator.samples // train_generator.batch_size \
            + validation_generator.samples // validation_generator.batch_size

    history = model.fit(
        merged_gen,
        steps_per_epoch=total_steps,
        epochs=30,
        callbacks=callbacks_final
    )

## 🧪 Оценка на тестовой выборке

Для тестирования использовался отдельный генератор с отключённым перемешиванием:

    results = model.evaluate(test_generator, 
                         steps=test_generator.samples//test_generator.batch_size,
                         verbose = 1)
Результаты выводятся по метрикам:
- loss: функция потерь
- acc: точность на тестовой выборке

## 📌 Заметки

- Все изображения были приведены к формату grayscale и размеру 224x224.
- Плохие изображения отфильтрованы с помощью библиотеки PIL.
- Для улучшения устойчивости модели использована L2-регуляризация и Dropout.
