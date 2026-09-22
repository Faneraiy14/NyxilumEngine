# NyxilumEngine API

Усе визначено в [`lib/engine.nx`](../lib/engine.nx). Імпортуй лише те,
що реально потрібно - NyxilumLang's selective `import { ... }` вимагає
явно перелічити й СТРУКТУРИ (типи), і функції-конструктори (методи
належать типу, не функції) - див. README.md, розділ "Чесно про v1".

## GameObject

```
struct GameObject {
    name: string
    x: number
    y: number
    rotation: number
    scale: number
    active: bool
    renderer: any    // Renderer{} чи 0
    collider: any    // BoxCollider{} чи 0
    scripts: array   // прикріплені скрипти (addScript)
}

createGameObject(name, x, y) -> GameObject
```

- `gameObject.addScript(s)` - додає скрипт (будь-який struct із
  `update(dt, canvas)`, опційно `onCollide(other)`); АВТОМАТИЧНО
  проставляє `s.go = gameObject` - скрипт бачить свого власника через
  `self.go`.

## Renderer

Один struct із полем `kind` ("rect"/"circle"/"sprite") - НЕ окремі
типи для кожної форми (мова не дає `typeOf()`-подібної диференціації
конкретних struct-типів).

```
rectRenderer(w, h, r, g, b) -> Renderer
circleRenderer(diameter, r, g, b) -> Renderer
spriteRenderer(image, w, h) -> Renderer   // image = результат loadImage()
```

Присвоюється в `gameObject.renderer`.

## BoxCollider

```
boxCollider(w, h) -> BoxCollider
```

AABB (прямокутне перетинання), (x,y) власника - ЦЕНТР, не кут.
Присвоюється в `gameObject.collider`.

## Scene

```
struct Scene {
    objects: array
}

createScene() -> Scene
```

- `scene.addObject(go)` - додає GameObject у сцену.
- `scene.findByName(name)` - повертає перший GameObject із цим `name`
  (чи `0`, якщо не знайдено).

## Крок кадру (окремі функції - переюзабельні напряму, не лише через Engine.run)

```
updateAllScripts(scene, dt, canvas)  // викликає update(dt, canvas) кожного активного скрипта
checkCollisions(scene)               // AABB-перевірка + notifyCollision для всіх активних collider'ів
drawAll(canvas, scene)               // малює renderer кожного активного об'єкта
```

`examples/dodge.nx` викликає ці три функції НАПРЯМУ у власному циклі
(замість `Engine.run()`) саме через потребу перевіряти game-over між
кадрами.

## Engine

```
struct Engine {}
createEngine() -> Engine
```

- `engine.run(canvas, scene)` - мінімальний головний цикл: поки НЕ
  `canvasShouldClose(canvas)` - `updateAllScripts` → `checkCollisions`
  → `clearCanvas` → `drawAll` → `presentCanvas`. БЕЗ callback-механізму
  для дострокового виходу (напр. game-over) - для такого пиши власний
  цикл із тими самими трьома функціями (див. `examples/dodge.nx`).

## Колізії - `onCollide`

Опційний метод скрипта: `func onCollide(other) { ... }` - `other` це
`GameObject`, з яким зіткнувся власник ЦЬОГО скрипта. Викликається для
КОЖНОГО скрипта на об'єкті, що має `onCollide` - скрипти без нього
тихо пропускаються (try/catch у `notifyCollision`).

## Конвенція "компонента"

Не окремий механізм мови - будь-який `struct` із методом
`update(dt, canvas)` (і опційно `onCollide(other)`), прикріплений через
`gameObject.addScript(s)`.
