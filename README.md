# Rhino.Inside AutoCAD – GH Components Pack (v2)

Набор Grasshopper C#-компонентов для Rhino.Inside.AutoCAD
Создание 2D проекций и аннотаций (Polyline / MText / Dimension / MLeader) прямо в AutoCAD из Grasshopper.

---

## Требования

* AutoCAD 2020 (или выше)
* Rhino 8 + Grasshopper
* Rhino.Inside.AutoCAD (ядро подключено)

---

## Установка

### Подключить библиотеку GH компонентов

Файл:

```
Rhino.inside.AutoCAD.Components.gha
```

Поместить в папку Grasshopper:

```
%AppData%\Grasshopper\Libraries
```

---
### Подключить Rhino.Inside в AutoCAD

В AutoCAD выполнить команду:

```
NETLOAD
```

→ выбрать файл:

```
Rhino.inside.AutoCAD.Core.dll
```

(из установки Rhino.Inside)

После загрузки Rhino и Grasshopper становятся доступны внутри AutoCAD.

---


## Общие правила работы компонентов

### Bake

У всех компонентов есть вход `bake` (`bool`):

* `false` → ничего не создаётся в AutoCAD (только расчёт / preview)
* `true` → объект создаётся в `ModelSpace` AutoCAD

---

### layerName

* Если слой существует → используется
* Если нет → создаётся автоматически

---

### Стили (TextStyle, DimStyle, MLeaderStyle)

* Если стиль существует → применяется
* Если не найден → используется стиль по умолчанию AutoCAD, в выход `A` приходит предупреждение

---

### Output A

Строка с логом: успех / предупреждение / ошибка.
Рекомендую подключать к `Panel`.

---

## Компоненты

---

## MAKE2D

### Назначение
<img width="849" height="465" alt="image" src="https://github.com/user-attachments/assets/e085665d-5f17-4a07-aa1e-8a3b103601be" />

Получить 2D ортогональную проекцию 3D модели и разделить линии на `visible` и `hidden`.

---

### Входы

* **`geometry`** (`List<Brep>` или `Curve`)
  3D модель

* **`direction`** (`Vector3d`)
  Направление взгляда (по умолчанию `0,0,-1` – вид сверху)

* **`up`** (`Vector3d`, optional)
  Вектор вверх (по умолчанию auto)

---

### Выходы

* **`visible`** (`DataTree<Curve>`)
  Видимые линии, ветки `{i}` по исходным объектам

* **`hidden`** (`DataTree<Curve>`)
  Скрытые линии, ветки `{i}` по исходным объектам

---

### Типовой пайплайн

```
Brep model → MAKE2D → Visible / Hidden → Полилиния по кривым → AutoCAD слои
```

---

## Полилиния по кривым v.2
<img width="844" height="424" alt="image" src="https://github.com/user-attachments/assets/92aac606-c921-4463-864b-df328e8deb87" />

### Назначение

Запекание Rhino `Curve` в AutoCAD `Polyline` с сохранением дуговых сегментов (bulge).

---

### Входы

* **`inputCurve`** (`Curve`)
  Кривая из MAKE2D или любая Rhino кривая

* **`layerName`** (`string`)

* **`bake`** (`bool`)

* **`tolerance`** (`double`)
  Для аппроксимации не-дуговых сегментов (`0.01–1.0`)

---

### Особенности

* Дуги сохраняются как настоящие скругления (`bulge`)
* `PolyCurve` разбивается на сегменты, дуги сохраняются
* `NURBS` / сложные кривые аппроксимируются

---

### Выход

* **`A`** — сообщение

---

## Полилиния по точкам v.2
<img width="968" height="539" alt="image" src="https://github.com/user-attachments/assets/1deab9e7-3096-4ae7-9d14-2e76feca23e5" />

### Назначение

Создать AutoCAD `Polyline` по списку точек.

---

### Входы

* **`pts`** (`List<Point3d>`)
  Минимум 2 точки

* **`layerName`** (`string`)

* **`bake`** (`bool`)

---

### Выход

* **`A`** — сообщение

---

## MText v.2
<img width="732" height="629" alt="image" src="https://github.com/user-attachments/assets/31f611d7-c968-4ecd-bf9b-d4eea0bbd954" />

### Назначение

Создать AutoCAD `MText`.

---

### Входы

* **`insertPoint`** (`Point3d`)
  Точка вставки

* **`textContent`** (`string`)
  Текст (`\P` для новой строки)

* **`textHeight`** (`double`)
  Высота

* **`layerName`** (`string`)

* **`textStyleName`** (`string`)
  Имя `TextStyle` (если нет – стандартный)

* **`rotation`** (`int`)
  Поворот в градусах

* **`alignment`** (`int`)
  Выравнивание (`1–9`):

  ```
  1 TopLeft,    2 TopCenter,    3 TopRight
  4 MiddleLeft, 5 MiddleCenter, 6 MiddleRight
  7 BottomLeft, 8 BottomCenter, 9 BottomRight
  ```

* **`backgroundMask`** (`bool`)
  Маска / фон

* **`bake`** (`bool`)

---

### Preview

Контур текста (виден в viewport AutoCAD).
Настоящий текст в preview не рендерится в ACAD viewport (ограничение Rhino.Inside).

---

### Выход

* **`A`** — сообщение

---

## Линейный размер v.2 (Rotated)
<img width="1166" height="515" alt="image" src="https://github.com/user-attachments/assets/6533ffba-fa34-4625-a23e-81790274a316" />
<img width="1181" height="521" alt="image" src="https://github.com/user-attachments/assets/7ebd5fdb-5fca-4d44-83f9-e0783a3b173c" />


### Назначение

Линейный размер с произвольным поворотом (параллелен `dimensionLine`).

---

### Входы

* **`dimensionLine`** (`Line`)
  Направление размерной линии

* **`dimensionStart`**, **`dimensionEnd`** (`Point3d`)
  Точки измерения

* **`layerName`** (`string`)

* **`bake`** (`bool`)

* **`dimensionStyleName`** (`string`)

* **`userOverrideText`** (`string`, optional)
  Свой текст

---

### Выход

* **`A`** — сообщение

---

## Повернутый размер v.2 (Aligned)
<img width="781" height="481" alt="image" src="https://github.com/user-attachments/assets/bd307508-d12d-4fbe-a105-f97c515a4b81" />

### Назначение

`AlignedDimension` – размер по направлению между точками,
размещение по ближайшей точке на `dimensionLine`.

---

### Входы

Те же, что в **Линейный размер v.2 (Rotated)**.

---

### Выход

* **`A`** — сообщение

---

## Мультивыноска v.2 (MLeader)
<img width="770" height="697" alt="image" src="https://github.com/user-attachments/assets/876ed254-1d95-4e3d-9cea-637dcaa56e9c" />

### Назначение

Создание AutoCAD `MLeader` + `MText`.

---

### Входы

* **`leaderText`** (`string`)
  Текст выноски

* **`leaderStart`** (`Point3d`)
  Начало стрелки

* **`leaderEnd`** (`Point3d`)
  Конец (текст)

* **`additionalPoint`** (`Point3d`)
  Вторая выноска (`0,0,0` если не нужна)

* **`layerName`** (`string`)

* **`bake`** (`bool`)

* **`annotationStyleName`** (`string`)
  Стиль `MLeader`

* **`backgroundMask`** (`bool`)

* **`scaleFactor`** (`double`)
  Масштаб стрелки (трюк)

* **`arrowType`** (`string`)
  Тип стрелки (из `ValueList` на русском)

---

### Preview

Контур

---

### Выход

* **`A`** — сообщение

