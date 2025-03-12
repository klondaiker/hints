# HotWire (HTML Over The Wire)
HotWire cостоит из Turbo, Stimulus и Native
# Turbo
Turbo состоит из Drive, Frames, Streams

## Drive
Что это такое?
1) Ex-Turbolinks
2) SPA для бедных: заставляет HTML приложения мимикрировать под SPA

Что делает?
1) TurboDrive загружает новую страницу асинхронно (fetch)
2) document.body заменяется на новый
3) document.header вливается в старый
4) TurboDrive восстанавливает полосу прокрутки

Как добавить?
```js
import "@hotwired/turbo-rails"
```
Все! TurboDrive подключен автоматически

Можно вручную загрузить страницы с Турбо \
**Application visits**:
```js
// Визит по-умолчанию c {action: "advance"} - добавляет запись в историю history.pushState
Turbo.visit("/example")

// Заменяет самую верхнюю запись истории history.replaceState
Turbo.visit("/edit", { action: "replace" })
```
**Restoration visits (TurboDrive: кэш)**: Turbo Drive отобразит копию страницы из кэша без запроса.
1) Turbo сохраняет слепки страниц при переходах
2) Слепок показывается при навигации Вперед/Назад в браузере и перед переходом (preview)
3) Слепок исключает элeменты, помеченные data-turbo-remporary (например alert или flash)


**Morphing and Scroll** \
Morphing позволяет заменять только элементы которые изменились. Используется библиотека [idiomorph](https://github.com/bigskysoftware/idiomorph)\
Scroll можно управлять прокруткой страницы
```ruby
# method - replace(default) or morph
# scroll - reset(default) or preserve
turbo_refreshes_with method: :morph, scroll: :preserve
```
`data-turbo-permanent` говорит Turbo не перерисовывать элемент если id совпадает, \
те исключает элемент из процесса морфинга или replace

## Frames
Что это такое?
1) TurboDrive, только для частей страницы (frames)
2) Перехват навигации внутри frame  (но можно и выходить за его пределы)
3) Ленивая загрузка частей страницы

```ruby
# во вьюхе помечаем область через `turbo_frame_tag`
<%= turbo_frame_tag(@example) do %>
  <div><a href="/example">example</a></div
<% end %>
```

Теперь при клике ссылок внутри turbo_frame_tag, Turbo ждет от результата запроса turbo_frame_tag \
и подставляет внутрь turbo_frame_tag с соответсвующим id. \
Если не находит, то подставляет ошибку `content missing` внутри turbo_frame_tag.

Чтобы выйти за пределы фрейма, добавляют `target: "_top"` в turbo_frame_tag, и все ссылки внутри фрейма становятся глобальными. \
Чтобы ссылки работали только внутри фрейма нужно добавить `"data-turbo-frame": "_self"`


В контроллере можно выдавать не весь html, если это turbo_frame запрос
```ruby
if turbo_frame_request? # или по конкретному id: turbo_frame_request_id ~= /example/
  render partial: "example", locals: locals
end

```

## Streams
Что это такое?
1) Декларативное изменение DOM
2) Изначально - для обновления по инициатие сервера (push-модель, WebSockets, SSE и тд)
3) Но сейчас активно используется и в рамках HTTP как более гибкое решение, чем Frames

Что из себя представляет?
1) Самоисполняемый и самоудаляемый HTML элемент
2) Содержит действие (action), ID цели (target) и почти всегда шаблон (template)

```erb
<turbo-stream action="replace" target="player">
  <template>
    <div class="player">...</div>
  </template>
</turbo-stream>
```

Список actions: append, prepend, replace, update, after, before, remove
Можно добавить свои с 7.2 версии

# Stimulus
Что это такое? \
js-микрофреймворк

Для чего?
1) UX не всегда требует участия сервера (закрытие модального окна, показать пароль в поле ввода и тд)
2) Когда нужно склеить интерактив, контролируемый turbo (dibounce/trottle для поиска)
3) Когда без настоящего js-фреймворка не обойтись

Пример:
```erb
<div data-controller="banner">
  <svg data-action="click->banner#hide"></svg>
  <p> Some notification </p>
</div>
```
```js
import { Controller } from "@hotwired/stimulus"

export class BannerController extends Controller {
  hide() {
    this.element.remove();
  }
}
```
```js
import { Application } from '@hotwired/stimulus';
import { BannerController } from 'banner_controller';

const application = Application.start();

application.register("banner", BannerController)
```
Чтобы не перечислять все контроллеры, можно подключить след образом (пример с Vite):
```js
import { registerControllers } from 'stimulus-vite-helpers';
const controllers = import.meta.glob('./**/*_controller.*', { eager: true });
registerControllers(application, controllers);
```

Типы контроллеров:
1) Поведение (removeable_controller) - атомарное поведение, атомарность позволяет переиспользовать и комбинировать стимулы
2) Компонент (banner_controller) - неплохо работает в связке с view components, не силен в сложных js-компонентах

API:
1) data-action: обработчики событии (которые не утекают)
2) data-target: инъекция DOM-зависимостей
3) data-value: реактивное состояние
4) data-class: скажем нет связности с CSS
5) data-outlet: строим составные стимулы
