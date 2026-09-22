# NyxilumEngine

Невеликий 2D ігровий рушій, написаний і скриптований самою
[NyxilumLang](https://github.com/Faneraiy14/NyxilumLang) - без
вбудовування чужої мови скриптів. GameObject/Component-стиль,
"віддалено схоже на Unity за структурою" - не за амбіцією рівня.
Windows-only (успадковано від 2D-графіки NyxilumLang - Windows Forms).

Повний план (архітектура, свідомі обмеження v1):
[план у сесії 22.09.2026](https://github.com/Faneraiy14/NyxilumLang/blob/main/NATIVE_ROADMAP.md)
(шукайте "NyxilumEngine" в історії коментарів) - коротко нижче.

## Структура

- **`lib/engine.nx`** - ядро: `GameObject`, `Scene`, `Renderer`
  (rect/circle/sprite), `BoxCollider`, `Engine`.
- **`examples/dodge.nx`** - робоча гра-приклад "Ухилення" - гравець
  рухається вліво/вправо, ухиляючись від блоків, що падають зверху.
- **`tests/headless_logic_test.nx`** - логіка рушія (GameObject/
  Scene/колізії/`self.go`) БЕЗ canvas - перевірено живцем на Linux
  (не потребує Windows Forms).

## Як це працює

"Компонент" - НЕ окремий механізм мови, а КОНВЕНЦІЯ: будь-який
`struct` із методом `update(dt, canvas)` (і опційно
`onCollide(other)`) - рушій сам викликає ці методи щокадру для
кожного скрипта, прикріпленого через `gameObject.addScript(s)`.
Кожен скрипт отримує `s.go` (посилання на свій GameObject) АВТОМАТИЧНО
від `addScript()` - завдяки цьому скрипт може міняти позицію
власника (`self.go.x = ...`), а не лише власні поля.

```nx
import "../lib/engine.nx" { GameObject, createGameObject, BoxCollider, boxCollider }

struct Mover {
    go: any
    speed: number

    func update(dt, canvas) {
        self.go.x = self.go.x + self.speed * dt
    }
}

var hero = createGameObject("hero", 0, 0)
hero.addScript(Mover { go: 0, speed: 100 })
```

## Чесно про v1 (не приховано)

- Лише 2D, лише Windows (успадковано від NyxilumLang - `#if WINDOWS`
  Windows Forms).
- Колізії - лише AABB (прямокутне перетинання), без гравітації/
  імпульсів за замовчуванням.
- `Renderer` - ОДИН struct із полем `kind` ("rect"/"circle"/
  "sprite"), а не окремі типи `ShapeRenderer`/`SpriteRenderer` - мова
  не дає `typeOf()`-подібної функції, що розрізняла б КОНКРЕТНИЙ
  struct-тип (перевірено живцем - `typeOf()` для будь-якого struct
  повертає просто `"struct"`), тож tagged-union через поле `kind`.
- `update(dt, canvas)`, а не буквально `update(dt)` (як у ранньому
  плані) - скрипту реально треба читати ввід (`isKeyDown(canvas,
  ...)`).
- Немає редактора сцен, серіалізації в файл - сцена описується кодом.
- Немає спрайт-аркушів/анімації - лише статичні кадри
  (`loadImage`/`drawImage`, вже в NyxilumLang).
- `Engine.run(canvas, scene)` - мінімальний головний цикл БЕЗ
  callback-механізму для дострокового виходу (напр. game-over) -
  `examples/dodge.nx` свідомо НЕ викликає `Engine.run()` напряму, а
  перевикористовує ті самі три функції кроку кадру
  (`updateAllScripts`/`checkCollisions`/`drawAll`) у власному циклі з
  додатковою умовою виходу - задокументовано в самому прикладі.

## Перевірено

- `nx check` (синтаксис) - `lib/engine.nx` і `examples/dodge.nx`.
- `tests/headless_logic_test.nx` - живий запуск (Linux, без canvas):
  `GameObject`/`Scene`/`findByName`, AABB-перетин (true/false),
  `update()` міняє власне поле скрипта, `self.go` коректно вказує на
  власника, `notifyCollision` викликає `onCollide` коли є й тихо
  пропускає, коли немає (try/catch).
- **Реальний живий запуск гри З ВІКНОМ (`examples/dodge.nx`) ЩЕ НЕ
  побачено власними очима** - потребує Windows-машини (Windows
  Forms), написано й перевірено наскільки можливо на Linux-сесії.
