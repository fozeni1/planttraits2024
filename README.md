# PlantTraits 2024
Проект посвящен предсказанию 6 различных характеристик цветов, растений и т.д. по фотографиям и табличным данным пользователей iNaturalist. Ссылка на соревнование: [kaggle.com](https://www.kaggle.com/competitions/planttraits2024)
<br>
Важность компетишна подверждают сами организаторы:<br><br>
   `«Why are plant traits currently so relevant? Plant traits are plant properties that are used to describe how plants function how they interact with the environment. For instance, the trait of plant canopy height indicates how good a plant is at overshadowing its neightbors in the competition for sun light. Robust leaves (indicated by the leaf mass pear leaf area) indicate that plants optimize towards extreme conditions, such as heavy winds or droughts. Yet, environmental conditions are not static. Due to global change, the biosphere is being transformed at accelerating pace. Especially climate change is assumed to drastically impact the functioning of the ecosystems. This includes several processes, e.g. adaptions of plants and their traits to new conditions or even a altered plant species distribution with a resulting modification of the distribution of plant traits. However, we can hardly project on a global scale how plant traits and as such entire ecosystems will react to climate change because we do not have sufficient data on plant traits.»`

Моя цель в данном соревновании - обучится работе с нейросетями, CV, в частности с трансформерами. Познакомиться с библиотекой torch, особенностями обучения моделей ну и архитектурой CLiP.

Почитать про CLiP можно здесь: [OpenAI CLiP](https://openai.com/index/clip/)

## V1
Первое, с чего я начал - определиться с архитектурой. Начал я с ноутбука, который нашел на хабре: [вот ссылка](https://colab.research.google.com/drive/1SH-VgAM62v0CAFVlbMPlfRYrbD5o8hsA?usp=sharing#scrollTo=alePijoXy6AH).
И собственно с этого началось построение моей мультимодальной модели. При помощи и интернета, и чатгпт выбрал вот такое:<br>
CLiP + TabularEncoder -> regression_head.<br>
*TabularEncoder:*
- nn.Linear(n_features, 256),
- nn.ReLU(),
- nn.Linear(256, 128),
- nn.ReLU()
<br><br>
*Regression head:*
- nn.Linear(640,128),
- nn.ReLU(),
- nn.Linear(128,6)

Веса CLiP я заморозил. Оптимизатор - AdamW.
Некоторые сложности:
- Данные экстремальные: смещение вправо очень сильное.
- В качестве экспермиента, я их просто убрал, следовательно, модель плохо описывает такие данные.
- Практически вся квадратичная ошибка пяти таргетов приходится на десять экстремальных наблюдений.
<br>

В итоге на 16 эпохе получалось что то такое:<br>
### Результаты обучения (Epoch 16/20)

| Target | Original R² | Filtered R² | Top 10 SSE (%) |
| :--- | ---: | ---: | ---: |
| X4_mean | 0.235214 | 0.235214 | 27.648125 |
| X11_mean | -0.000072 | 0.178733 | 99.999958 |
| X18_mean | -0.000181 | 0.459967 | 100.000000 |
| X26_mean | -0.000217 | 0.094624 | 99.999848 |
| X50_mean | -0.000093 | 0.155842 | 99.999720 |
| X3112_mean | -0.000090 | 0.200284 | 100.000000 |
| **Mean** | **0.039094** | **0.220777** |
<br>
FilteredR2 не учитывает значения, лежащие после 99,9% процентиля.
