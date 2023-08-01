---
outline: deep
---

# Render-функции & JSX {#render-functions-jsx}

В подавляющем большинстве случаев Vue рекомендует использовать шаблоны для построения приложений. Однако бывают ситуации, когда нам необходима вся программная мощь JavaScript. Именно в таких случаях мы можем использовать **render-функции**.

> Если вы впервые знакомитесь с концепцией виртуального DOM и функций рендеринга, обязательно прочтите сначала главу [механизмы отрисовки](/guide/extras/rendering-mechanism.html).

## Базовое применение {#basic-usage}

### Создание узлов Vnode {#creating-vnodes}

Vue предоставляет функцию `h()` для создания vnode:

```js
import { h } from 'vue'

const vnode = h(
  'div', // тип
  { id: 'foo', class: 'bar' }, // входные параметры
  [
    /* дочерние элементы */
  ]
)
```

`h()` это сокращение от **hyperscript**, что означает "JavaScript, создающий HTML (язык гипертекстовой разметки)". Это название унаследовано от соглашений, общих для многих реализаций виртуального DOM. Более описательным именем могло бы быть `createVnode()`, но более короткое имя помогает, когда приходится многократно вызывать эту функцию в функции рендеринга.

Функция `h()` спроектирована очень гибко:

```js
// все аргументы, кроме типа, являются необязательными
h('div')
h('div', { id: 'foo' })

// в реквизитах могут использоваться как атрибуты, так и свойства
// Vue автоматически выбирает правильный способ назначения
h('div', { class: 'bar', innerHTML: 'hello' })

// Модификаторы реквизитов, такие как .prop и .attr, могут быть добавлены
// с префиксами '.' и `^' соответственно
h('div', { '.name': 'some-name', '^width': '100' })

// класс и стиль имеют ту же поддержку значений
// объектов / массивов, что и в шаблонах
h('div', { class: [foo, { bar }], style: { color: 'red' } })

// слушатели событий должны передаваться как onXxx
h('div', { onClick: () => {} })

// дочерние элементы могут быть строкой
h('div', { id: 'foo' }, 'hello')

// входные параметры могут быть опущены, если они отсутствуют
h('div', 'hello')
h('div', [h('span', 'hello')])

// Массив дочерних элементов может содержать смешанные vnodes и строки
h('div', ['hello', h('span', 'hello')])
```

Результирующий vnode имеет следующую форму:

```js
const vnode = h('div', { id: 'foo' }, [])

vnode.type // 'div'
vnode.props // { id: 'foo' }
vnode.children // []
vnode.key // null
```

:::warning Примечание
Полный интерфейс `VNode` содержит множество других внутренних свойств, но настоятельно рекомендуется не полагаться на какие-либо свойства, кроме перечисленных здесь. Это позволит избежать непредвиденных поломок в случае изменения внутренних свойств.
:::

### Объявление render-функций {#declaring-render-functions}

<div class="composition-api">

При использовании шаблонов с Composition API возвращаемое значение хука `setup()` используется для передачи данных шаблону. Однако при использовании функций рендеринга мы можем напрямую возвращать функцию рендеринга:

```js
import { ref, h } from 'vue'

export default {
  props: {
    /* ... */
  },
  setup(props) {
    const count = ref(1)

    // возврат render-функции
    return () => h('div', props.msg + count.value)
  }
}
```

Render-функция объявляется внутри `setup()`, поэтому она, естественно, имеет доступ к входным параметрам и любому реактивному состоянию, объявленному в той же области видимости.

Помимо возврата одного vnode, можно также возвращать строки или массивы:

```js
export default {
  setup() {
    return () => 'привет мир!'
  }
}
```

```js
import { h } from 'vue'

export default {
  setup() {
    // использование массива для возврата нескольких корневых узлов
    return () => [
      h('div'),
      h('div'),
      h('div')
    ]
  }
}
```

:::tip Совет
Убедитесь, что вместо прямого возврата значений возвращается функция! Функция `setup()` вызывается только один раз для каждого компонента, в то время как возвращаемая render-функция будет вызвана несколько раз.
:::

</div>
<div class="options-api">

Мы можем объявить render-функцию, используя опцию `render`:

```js
import { h } from 'vue'

export default {
  data() {
    return {
      msg: 'привет'
    }
  },
  render() {
    return h('div', this.msg)
  }
}
```

Функция `render()` имеет доступ к экземпляру компонента через `this`.

Помимо возврата одного vnode, можно также возвращать строки или массивы:

```js
export default {
  render() {
    return 'привет мир!'
  }
}
```

```js
import { h } from 'vue'

export default {
  render() {
    // использование массива для возврата нескольких корневых узлов
    return [
      h('div'),
      h('div'),
      h('div')
    ]
  }
}
```

</div>

Если компоненту render-функции не требуется состояние экземпляра, то для краткости он может быть объявлен непосредственно как функция:

```js
function Hello() {
  return 'привет мир!'
}
```

Все верно, это правильный компонент Vue! Подробнее об этом синтаксисе смотрите в разделе [Функциональные компоненты](#functional-components).

### Узлы Vnode должны быть уникальными {#vnodes-must-be-unique}

Все vnode в дереве компонентов должны быть уникальными. Это означает, что следующая функция рендеринга недопустима:

```js
function render() {
  const p = h('p', 'hi')
  return h('div', [
    // Внимание - дублирование vnodes!
    p,
    p
  ])
}
```

Если вы действительно хотите многократно дублировать один и тот же элемент/компонент, это можно сделать с помощью фабричной функции. Например, следующая render-функция является вполне корректным способом вывода 20 одинаковых абзацев:

```js
function render() {
  return h(
    'div',
    Array.from({ length: 20 }).map(() => {
      return h('p', 'hi')
    })
  )
}
```

## JSX / TSX {#jsx-tsx}

[JSX](https://facebook.github.io/jsx/) - это XML-подобное расширение JavaScript, которое позволяет нам писать подобный код:

```jsx
const vnode = <div>привет</div>
```

Внутри JSX-выражений используйте фигурные скобки для вставки динамических значений:

```jsx
const vnode = <div id={dynamicId}>привет, {userName}</div>
```

В `create-vue` и Vue CLI есть опции для создания проектов с предварительно настроенной поддержкой JSX. Если вы настраиваете JSX вручную, обратитесь к документации [`@vue/babel-plugin-jsx`](https://github.com/vuejs/jsx-next) для получения подробной информации.

Хотя JSX впервые появился в React, на самом деле он не имеет определенной семантики во время выполнения и может быть скомпилирован в различные выходные данные. Если вы уже работали с JSX, обратите внимание, что **преобразование Vue JSX отличается от преобразования React JSX**, поэтому вы не можете использовать преобразование React JSX в приложениях Vue. Некоторые заметные отличия от React JSX включают:

- В качестве реквизитов можно использовать HTML-атрибуты, такие как `class` и `for`, - нет необходимости использовать `className` или `htmlFor`.
- Передача дочерних элементов компонентам (т.е. слотов) [работает по-другому](#passing-slots).

Определение типов в Vue также обеспечивает вывод типов для использования TSX. При использовании TSX обязательно укажите `"jsx": "preserve"` в `tsconfig.json`, чтобы TypeScript оставлял синтаксис JSX нетронутым для обработки JSX-преобразованием Vue.

## Рецепты render-функций {#render-function-recipes}

Ниже мы приведем несколько общих рецептов реализации шаблонных функций в виде эквивалентных им функций рендеринга / JSX.

### `v-if` {#v-if}

Шаблон:

```vue-html
<div>
  <div v-if="ok">да</div>
  <span v-else>нет</span>
</div>
```

Эквивалентная render-функция рендеринга / JSX:

<div class="composition-api">

```js
h('div', [ok.value ? h('div', 'да') : h('span', 'нет')])
```

```jsx
<div>{ok.value ? <div>да</div> : <span>нет</span>}</div>
```

</div>
<div class="options-api">

```js
h('div', [this.ok ? h('div', 'да') : h('span', 'нет')])
```

```jsx
<div>{this.ok ? <div>да</div> : <span>нет</span>}</div>
```

</div>

### `v-for` {#v-for}

Шаблон:

```vue-html
<ul>
  <li v-for="{ id, text } in items" :key="id">
    {{ text }}
  </li>
</ul>
```

Эквивалентная render-функция рендеринга / JSX:

<div class="composition-api">

```js
h(
  'ul',
  // предполагается, что `items` - это ссылка со значением массива
  items.value.map(({ id, text }) => {
    return h('li', { key: id }, text)
  })
)
```

```jsx
<ul>
  {items.value.map(({ id, text }) => {
    return <li key={id}>{text}</li>
  })}
</ul>
```

</div>
<div class="options-api">

```js
h(
  'ul',
  this.items.map(({ id, text }) => {
    return h('li', { key: id }, text)
  })
)
```

```jsx
<ul>
  {this.items.map(({ id, text }) => {
    return <li key={id}>{text}</li>
  })}
</ul>
```

</div>

### `v-on` {#v-on}

Реквизиты с именами, начинающимися с `on`, за которым следует заглавная буква, рассматриваются как слушатели событий. Например, `onClick` является эквивалентом `@click` в шаблонах.

```js
h(
  'button',
  {
    onClick(event) {
      /* ... */
    }
  },
  'нажми на меня'
)
```

```jsx
<button
  onClick={(event) => {
    /* ... */
  }}
>
  нажми на меня
</button>
```

#### Модификаторы событий {#event-modifiers}

Для модификаторов событий `.passive`, `.capture`, и `.once` они могут быть объединены после имени события с использованием camelCase.

Например:

```js
h('input', {
  onClickCapture() {
    /* слушатель в режиме захвата */
  },
  onKeyupOnce() {
    /* срабатывает только один раз */
  },
  onMouseoverOnceCapture() {
    /* один раз + захват */
  }
})
```

```jsx
<input
  onClickCapture={() => {}}
  onKeyupOnce={() => {}}
  onMouseoverOnceCapture={() => {}}
/>
```

Для других модификаторов событий и клавиш можно использовать помощник [`withModifiers`](/api/render-function.html#withmodifiers):

```js
import { withModifiers } from 'vue'

h('div', {
  onClick: withModifiers(() => {}, ['self'])
})
```

```jsx
<div onClick={withModifiers(() => {}, ['self'])} />
```

### Компоненты {#components}

Чтобы создать vnode для компонента, первым аргументом, передаваемым в `h()`, должно быть определение компонента. Это означает, что при использовании render-функций нет необходимости регистрировать компоненты - можно просто использовать импортированные компоненты напрямую:

```js
import Foo from './Foo.vue'
import Bar from './Bar.jsx'

function render() {
  return h('div', [h(Foo), h(Bar)])
}
```

```jsx
function render() {
  return (
    <div>
      <Foo />
      <Bar />
    </div>
  )
}
```

Как мы видим, `h` может работать с компонентами, импортированными из файлов любого формата, если это корректный компонент Vue.

Динамические компоненты прямолинейны с помощью render-функций:

```js
import Foo from './Foo.vue'
import Bar from './Bar.jsx'

function render() {
  return ok.value ? h(Foo) : h(Bar)
}
```

```jsx
function render() {
  return ok.value ? <Foo /> : <Bar />
}
```

Если компонент зарегистрирован по имени и не может быть импортирован напрямую (например, глобально зарегистрирован библиотекой), он может быть программно разрешен с помощью помощника [`resolveComponent()`](/api/render-function.html#resolvecomponent).

### Слоты рендеринга {#rendering-slots}

<div class="composition-api">

В render-функциях доступ к слотам осуществляется из контекста `setup()`. Каждый слот объекта `slots` представляет собой **функцию, возвращающую массив vnodes**:

```js
export default {
  props: ['message'],
  setup(props, { slots }) {
    return () => [
      // слот по умолчанию:
      // <div><slot /></div>
      h('div', slots.default()),

      // именованный слот:
      // <div><slot name="footer" :text="message" /></div>
      h(
        'div',
        slots.footer({
          text: props.message
        })
      )
    ]
  }
}
```

JSX эквивалент:

```jsx
// по умолчанию
<div>{slots.default()}</div>

// именованный
<div>{slots.footer({ text: props.message })}</div>
```

</div>
<div class="options-api">

В render-функциях доступ к слотам можно получить из [`this.$slots`](/api/component-instance.html#slots):

```js
export default {
  props: ['message'],
  render() {
    return [
      // <div><slot /></div>
      h('div', this.$slots.default()),

      // <div><slot name="footer" :text="message" /></div>
      h(
        'div',
        this.$slots.footer({
          text: this.message
        })
      )
    ]
  }
}
```

JSX эквивалент:

```jsx
// <div><slot /></div>
<div>{this.$slots.default()}</div>

// <div><slot name="footer" :text="message" /></div>
<div>{this.$slots.footer({ text: this.message })}</div>
```

</div>

### Передача слотов {#passing-slots}

Передача дочерних элементов компонентам работает несколько иначе, чем передача дочерних элементов элементам. Вместо массива нам нужно передать либо слот-функцию, либо объект слот-функции. Слот-функции могут возвращать все, что может вернуть обычная render-функция, которые при обращении к дочернему компоненту всегда будут нормализованы к массивам vnodes.

```js
// единственный слот по умолчанию
h(MyComponent, () => 'привет')

// именованные слоты
// обратите внимание, что `null` требуется для того, чтобы избежать
// того, чтобы объект слотов не рассматривался как props
h(MyComponent, null, {
  default: () => 'слот по умолчанию',
  foo: () => h('div', 'foo'),
  bar: () => [h('span', 'один'), h('span', 'два')]
})
```

JSX эквивалент:

```jsx
// по умолчанию
<MyComponent>{() => 'привет'}</MyComponent>

// named
<MyComponent>{{
  default: () => 'слот по умолчанию',
  foo: () => <div>foo</div>,
  bar: () => [<span>один</span>, <span>два</span>]
}}</MyComponent>
```

Передача слотов в виде функций позволяет дочернему компоненту лениво вызывать их. Это приводит к тому, что зависимости слота отслеживаются дочерним компонентом, а не родительским, что обеспечивает более точное и эффективное обновление.

### Встроенные компоненты {#built-in-components}

[Встроенные компоненты](/api/built-in-components.html) такие как `<KeepAlive>`, `<Transition>`, `<TransitionGroup>`, `<Teleport>` и `<Suspense>` должны быть импортированы для использования в render-функциях:

<div class="composition-api">

```js
import { h, KeepAlive, Teleport, Transition, TransitionGroup } from 'vue'

export default {
  setup () {
    return () => h(Transition, { mode: 'out-in' }, /* ... */)
  }
}
```

</div>
<div class="options-api">

```js
import { h, KeepAlive, Teleport, Transition, TransitionGroup } from 'vue'

export default {
  render () {
    return h(Transition, { mode: 'out-in' }, /* ... */)
  }
}
```

</div>

### `v-model` {#v-model}

При компиляции шаблона директива `v-model` расширяется до реквизитов `modelValue` и `onUpdate:modelValue` - нам придется предоставить эти реквизиты самостоятельно:

<div class="composition-api">

```js
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  setup(props, { emit }) {
    return () =>
      h(SomeComponent, {
        modelValue: props.modelValue,
        'onUpdate:modelValue': (value) => emit('update:modelValue', value)
      })
  }
}
```

</div>
<div class="options-api">

```js
export default {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  render() {
    return h(SomeComponent, {
      modelValue: this.modelValue,
      'onUpdate:modelValue': (value) => this.$emit('update:modelValue', value)
    })
  }
}
```

</div>

### Пользовательские директивы {#custom-directives}

Пользовательские директивы могут быть применены к узлу vnode с помощью функции [`withDirectives`](/api/render-function.html#withdirectives):

```js
import { h, withDirectives } from 'vue'

// собственная директива
const pin = {
  mounted() { /* ... */ },
  updated() { /* ... */ }
}

// <div v-pin:top.animate="200"></div>
const vnode = withDirectives(h('div'), [
  [pin, 200, 'top', { animate: true }]
])
```

Если директива зарегистрирована по имени и не может быть импортирована напрямую, она может быть разрешена с помощью помощника [`resolveDirective`](/api/render-function.html#resolvedirective).

## Функциональные компоненты {#functional-components}

Функциональные компоненты - это альтернативная форма компонентов, не имеющая собственного состояния. Они действуют как чистые функции: props in, vnodes out. Они отображаются без создания экземпляра компонента (т.е. без this) и без обычных хуков жизненного цикла компонента.

Для создания функционального компонента мы используем не объект options, а обычную функцию. Функция фактически является `render` функцией для компонента.

<div class="composition-api">

Сигнатура функционального компонента совпадает с сигнатурой хука `setup()`:

```js
function MyComponent(props, { slots, emit, attrs }) {
  // ...
}
```

</div>
<div class="options-api">

Поскольку для функционального компонента не существует ссылки `this`, Vue передает в качестве первого аргумента `props`:

```js
function MyComponent(props, context) {
  // ...
}
```

Второй аргумент, `context`, содержит три свойства: `attrs`, `emit`, и `slots`. Они эквивалентны свойствам экземпляра [`$attrs`](/api/component-instance.html#attrs), [`$emit`](/api/component-instance.html#emit), и [`$slots`](/api/component-instance.html#slots) соответственно.

</div>

Большинство обычных опций конфигурации компонентов недоступны для функциональных компонентов. Однако можно определить [`props`](/api/options-state.html#props) и [`emits`](/api/options-state.html#emits), добавив их в качестве свойств:

```js
MyComponent.props = ['value']
MyComponent.emits = ['click']
```

Если опция `props` не указана, то объект`props`, передаваемый в функцию, будет содержать все атрибуты, как и `attrs`. Имена реквизитов не будут нормализованы к camelCase, если не указана опция `props`.

Для функциональных компонентов с явными `props`, [наследование атрибутов](/guide/components/attrs.html) работает так же, как и для обычных компонентов. Однако для функциональных компонентов, не имеющих явных `props`, по умолчанию от `attrs` наследуются только `class`, `style`, и слушатели событий `onXxx`. В любом случае для `inheritAttrs` можно установить значение `false`, чтобы отключить наследование атрибутов:

```js
MyComponent.inheritAttrs = false
```

Функциональные компоненты могут быть зарегистрированы и использованы так же, как и обычные компоненты. Если вы передадите функцию в качестве первого аргумента функции `h()`, она будет рассматриваться как функциональный компонент.
