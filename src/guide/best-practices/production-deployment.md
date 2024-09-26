# Публикация на Production {#production-deployment}

## Разработка vs. Production {#development-vs-production}

В процессе разработки, Vue предоставляет ряд возможностей для улучшения её процесса:

- Предупреждения для частых ошибок и "подводных камней"
- Валидация пропсов / событий
- [Хуки для отладки реактивности](/guide/extras/reactivity-in-depth.html#reactivity-debugging)
- Интеграция инструментов разработки

Однако эти возможности становятся бесполезными в production. Некоторые из проверок предупреждений также могут вызывать небольшие накладные расходы на производительность.

## Без инструментов сборки {#without-build-tools}

Если вы используете Vue без инструмента сборки загружая его из CDN, либо из самостоятельного сценария, убедитесь, что используете production сборку (выходные файлы, что заканчиваются на `.prod.js`) во время внедрения в production. Production сборки предварительно минифицируются, из них удаляются все ветки кода, предназначенные только для разработки.

- При использовании глобальной сборки (получая доступ через глобальный `Vue`): используйте `vue.global.prod.js`
- При использовании ESM сборки (получая доступ через нативные ESM импорты): используйте `vue.esm-browser.prod.js` 

Более подробную информацию можно найти в [руководстве по выходным файлам](https://github.com/vuejs/core/tree/main/packages/vue#which-dist-file-to-use) 

## С инструментами сборки {#with-build-tools}

Проекты собранные через `create-vue` (на базе Vite) или Vue CLI (на базе webpack) предварительно конфигурируются для production сборок. 

При использовании пользовательской настройки, убедитесь, что:

1. `vue` разрешается к `vue.runtime.esm-bundler.js`.
2. [Флаги функций времени компиляции](https://github.com/vuejs/core/tree/main/packages/vue#bundler-build-feature-flags) настроены правильно.
3. <code>process.env<wbr>.NODE_ENV</code> заменён на `"production"` во время сборки.

Дополнительные ссылки:

- [Руководство по production сборке Vite](https://vitejs.dev/guide/build.html)
- [Руководство по развёртыванию через Vite](https://vitejs.dev/guide/static-deploy.html)
- [Руководство по развёртыванию через Vue CLI](https://cli.vuejs.org/guide/deployment.html)

## Отслеживание ошибок среды выполнения {#tracking-runtime-errors}

[Обработчик ошибок на уровне приложения](/api/application.html#app-config-errorhandler) может быть использован для сообщения об ошибках службам отслеживания:

```js
import { createApp } from 'vue'

const app = createApp(...)

app.config.errorHandler = (err, instance, info) => {
  // сообщить об ошибке службам отслеживания
}
```

Службы такие как [Sentry](https://docs.sentry.io/platforms/javascript/guides/vue/) и [Bugsnag](https://docs.bugsnag.com/platforms/javascript/vue/) также предоставляют свои официальные интеграции для Vue.
