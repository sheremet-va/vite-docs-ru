# Env переменные и режимы

## Env переменные

Vite предоставляет env переменные в специальном объекте **`import.meta.env`**. Некоторые встроенные переменные доступны во всех случаях:

- **`import.meta.env.MODE`**: {string} [режим](#modes), в котором работает приложение.

- **`import.meta.env.BASE_URL`**: {string} базовый URL-адрес, на котором работает приложение. Это определяется [параметром `base` конфигурации](/config/#base).

- **`import.meta.env.PROD`**: {boolean} работает ли приложение в production среде.

- **`import.meta.env.DEV`**: {boolean} работает ли приложение в development среде (всегда противоположно `import.meta.env.PROD`)

### Production замена

Во время production эти env переменные **статически заменяются**. Поэтому необходимо всегда ссылаться на них, используя полную статическую строку. Например, динамический доступ к ключу, как `import.meta.env[key]`, не будет работать.

Замена также произойдет для этих переменных, появляющихся в JavaScript строках и Vue шаблонах. Это должно быть редким случаем, но он может быть и непреднамеренным. Есть способы обойти это поведение:

- Для JavaScript строк вы можете разбить строку с помощью unicode zero-width пробела, например `'import.meta\u200b.env.MODE'`.
- Для Vue шаблонов или другого HTML, который компилируется в JavaScript строки, вы можете использовать [тег `<wbr>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/wbr), например `import.meta.<wbr>env.MODE`.

## `.env` файлы

Vite использует [dotenv](https://github.com/motdotla/dotenv) для загрузки дополнительных переменных среды из следующих файлов в вашем [каталоге среды](/config/#envDir):

```
.env                # загрузится во всех случаях
.env.local          # загрузится во всех случаях, игнорируется git'ом
.env.[mode]         # загрузится только в указанном режиме
.env.[mode].local   # загрузится только в указанном режиме, игнорируется git'ом
```

Загруженные env переменные также доступны в вашем клиентском исходном коде через `import.meta.env`.

Чтобы предотвратить случайную утечку env переменных на клиент, Vite обработает только переменные с префиксом `VITE_`, например следующий файл:

```
DB_PASSWORD=foobar
VITE_SOME_KEY=123
```

Только `VITE_SOME_KEY` будет отображаться как `import.meta.env.VITE_SOME_KEY` для клиентского исходного кода, а `DB_PASSWORD` - нет.

:::warning ЗАМЕТКИ ПО БЕЗОПАСНОСТИ

- Файлы `.env.*.local` являются только локальными и могут содержать конфиденциальные переменные. Вы должны добавить `.local` в ваш `.gitignore`, чтобы избежать их регистрации в git.

- Поскольку любые переменные, представленные вашему исходному коду Vite, попадут в вашу клиентскую сборку, переменные `VITE_*` _не_ должны содержать какую-либо конфиденциальную информацию.
  :::

### IntelliSense

По умолчанию Vite предоставляет определение типов для `import.meta.env`. Хотя вы можете определить кастомные env переменные в `.env.[mode]` файлах, вам может потребоваться TypeScript IntelliSense для user-defined env переменных с префиксом `VITE_`.

Для этого вы можете создать `env.d.ts` в каталоге `src`, а затем дополнить `ImportMetaEnv` следующим образом:

```typescript
interface ImportMetaEnv {
  VITE_APP_TITLE: string
  // другие env переменные...
}
```

## Режимы

По умолчанию сервер разработки (команда `serve`) запускается в `development` режиме, а команда `build` – в `production` режиме.

Это означает, что при запуске `vite build` он загрузит env переменные из `.env.production`, если они есть:

```
# .env.production
VITE_APP_TITLE=My App
```

В вашем приложении вы можете отрендерить заголовок с помощью `import.meta.env.VITE_APP_TITLE`.

Однако важно понимать, что **режим** - это более широкое понятие, чем просто development и production. Типичный пример: вам может потребоваться "staging" режим, в котором должно быть поведение, подобное production, но с немного отличающимися env переменными от production среды.

Вы можете перезаписать режим по умолчанию, используемый для команды, передав флаг опции `--mode`. Например, если вы хотите собрать свое приложение для нашего гипотетического staging режима:

```bash
vite build --mode staging
```

И чтобы получить желаемое поведение, нам нужен `.env.staging` файл:

```
# .env.staging
NODE_ENV=production
VITE_APP_TITLE=My App (staging)
```

Теперь ваше staging приложение должно иметь поведение, подобное production, но отображать заголовок, отличный от production.