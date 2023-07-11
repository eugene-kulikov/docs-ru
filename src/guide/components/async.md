# Асинхронные компоненты {#async-components}

## Основное использование {#basic-usage}

В больших приложениях может потребоваться разделить приложение на более мелкие части и загружать компоненты с сервера только тогда, когда они необходимы. Чтобы реализовать подобное Vue предоставляет метод [`defineAsyncComponent`](/api/general.html#defineasynccomponent):

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    // ...загрузка компонента с сервера
    resolve(/* загруженный компонент */)
  })
})
// ... используйте `AsyncComp` как обычный компонент
```

Как можно увидеть, `defineAsyncComponent` принимает функцию-фабрику, возвращающую Promise. В Promise коллбэк `resolve` должен быть вызван, когда получено определение компонента с сервера. Также можно вызвать `reject(reason)` для обработки неудачи при загрузке.

[Динамический импорт ES-модуля](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) также возвращает Promise, поэтому чаще всего мы будем использовать его в сочетании с `defineAsyncComponent`. Такие бандлеры, как Vite и webpack, также поддерживают этот синтаксис (и будут использовать его в качестве точки разделения бандла), поэтому мы можем использовать его для импорта SFC Vue:

```js
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)
```

Полученный в результате `AsyncComp` представляет собой компонент-обертку, который вызывает функцию загрузчик только тогда, когда он действительно отображается на странице. Кроме того, он будет передавать все входные параметры и слоты внутреннему компоненту, что позволяет использовать асинхронную обертку для плавной замены исходного компонента, добиваясь при этом "ленивой" загрузки.

Как и обычные компоненты, асинхронные компоненты могут быть [зарегистрированы глобально](/guide/components/registration.html#global-registration) с помощью `app.component()`:

```js
app.component('MyComponent', defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
))
```

<div class="options-api">

Вы также можете использовать `defineAsyncComponent` при [локальной регистрации компонента](/guide/components/registration.html#local-registration):

```vue
<script>
import { defineAsyncComponent } from 'vue'

export default {
  components: {
    AdminPage: defineAsyncComponent(() =>
      import('./components/AdminPageComponent.vue')
    )
  }
}
</script>

<template>
  <AdminPage />
</template>
```

</div>

<div class="composition-api">

Они также могут быть определены непосредственно внутри родительского компонента:

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

const AdminPage = defineAsyncComponent(() =>
  import('./components/AdminPageComponent.vue')
)
</script>

<template>
  <AdminPage />
</template>
```

</div>

## Состояния загрузки и ошибки {#loading-and-error-states}

Асинхронные операции неизбежно влекут за собой состояния загрузки и ошибки - `defineAsyncComponent()` поддерживает обработку этих состояний с помощью дополнительных опций:

```js
const AsyncComp = defineAsyncComponent({
  // функция загрузчика
  loader: () => import('./Foo.vue'),

  // Компонент, который будет использоваться во время загрузки асинхронного компонента
  loadingComponent: LoadingComponent,
  // Задержка перед показом компонента загрузки. По умолчанию: 200 мс.
  delay: 200,

  // Компонент, который будет использоваться в случае сбоя загрузки
  errorComponent: ErrorComponent,
  // Компонент ошибки будет выведен на экран, если таймаут
  // задан и превышен. По умолчанию: бесконечность.
  timeout: 3000
})
```

Если указан компонент загрузки, то он будет отображаться первым, пока загружается внутренний компонент. По умолчанию задержка перед отображением компонента загрузки составляет 200 мс - это связано с тем, что в быстрых сетях состояние мгновенной загрузки может сменяться слишком быстро и выглядеть как мерцание.

Если указан компонент ошибки, то он будет отображаться, когда Promise, возвращенный функцией загрузчика, будет отклонен. Можно также указать таймаут для отображения компонента ошибки, если запрос выполняется слишком долго.

## Использование с Suspense {#using-with-suspense}

Асинхронные компоненты можно использовать со встроенным компонентом `<Suspense>`. Взаимодействие между `<Suspense>` и асинхронными компонентами описано в [отдельной главе для `<Suspense>`](/guide/built-ins/suspense.html).
