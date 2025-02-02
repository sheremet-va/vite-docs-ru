# Почему Vite

## Проблемы

До того, как модули ES стали доступны в браузерах, у разработчиков не было нативных механизмов для создания JavaScript в модульном виде. Вот почему все мы знакомы с концепцией "сборки": использование инструментов, которые сканируют, обрабатывают и объединяют наши исходные модули в файлы, которые можно запускать в браузере.

Со временем мы увидели такие инструменты, как [webpack](https://webpack.js.org/), [Rollup](https://rollupjs.org) и [Parcel](https://parceljs.org/), что значительно улучшило опыт разработки frontend разработчиков.

Однако по мере того, как мы начинаем создавать все более и более амбициозные приложения, количество JavaScript, с которым мы имеем дело, также увеличивалось в геометрической прогрессии. Крупномасштабные проекты нередко содержат тысячи модулей. Мы начинаем сталкиваться с узким местом производительности для инструментов на основе JavaScript: часто может потребоваться неоправданно долгое ожидание (иногда до минут!), чтобы развернуть сервер разработки, и даже с HMR редактирование файлов может занять пару секунд до отражения в браузере. Медленная обратная связь может сильно влиять на продуктивность и настрой разработчиков.

Vite стремится решить эти проблемы, используя новые достижения в экосистеме: доступность нативных ES модулей в браузере и современные инструменты JavaScript, написанные на compile-to-native языках.

### Медленный запуск сервера

При холодном запуске сервера разработки, bundler-based сборка должна охотно просканировать и сбилдить все ваше приложение, прежде чем его можно будет запустить.

Vite улучшает время запуска сервера разработки, разделив сначала модули в приложении на две категории: **зависимости** и **исходный код**.

- **Зависимости** в основном представляют из себя простой JavaScript, который не часто меняется во время разработки. Некоторые большие зависимости (например, библиотеки компонентов с сотнями модулей) также довольно дорого обрабатывать. Зависимости также могут поставляться в различных форматах модулей (например, ESM или CommonJS).

  Vite [создает сборку зависимостей](./dep-pre-bundling) с помощью [esbuild](https://esbuild.github.io/). Esbuild написан на Go и собирает зависимости в 10-100 раз быстрее, чем бандлеры на основе JavaScript.

- **Исходный код** часто содержит непростой JavaScript, который требуется преобразовать (например, JSX, CSS или Vue/Svelte компоненты) и который, впоследствии, будет очень часто редактироваться. Кроме того, не весь исходный код нужно загружать одновременно (например, разделение кода на основе роутера).

  Vite запускает исходный код поверх [нативного ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules). По сути, это позволяет браузеру взять на себя часть работы сборщика пакетов: Vite нужно только преобразовывать и отдать исходный код по запросу, когда браузер запрашивает его. Условный динамический импорт кода программной части обрабатывается только в том случае, если он действительно используется на текущем экране.

  ![сервер разработки на основе сборщика](/images/bundler.png)

  ![сервер разработки на основе esm](/images/esm.png)

### Медленные обновления

Когда файл редактируется в сборке на основе бандлера, то сборщик неэффективно билдит весь пакет по очевидным причинам: скорость обновления будет линейно снижаться с размером приложения.

Некоторые бандлеры дев-сервера запускают сборку в памяти, так что ему нужно только инвалидировать часть своего графа модулей при изменении файла, но ему все равно необходимо заново собрать весь пакет и перезагрузить веб-страницу. Восстановление сборки может быть дорогостоящим, а перезагрузка страницы срывает текущее состояние приложения. Вот почему некоторые сборщики поддерживают горячую замену модулей (HMR): позволяя модулю выполнять "горячую замену", не затрагивая остальную часть страницы. Это значительно улучшает DX - однако на практике мы обнаруживаем, что даже скорость обновления HMR значительно ухудшается по мере роста размера приложения.

В Vite HMR выполняется поверх нативного ESM. Когда файл редактируется, Vite нужно только точечно инвалидировать цепочку между редактируемым модулем и его ближайшую границу HMR (в большинстве случаев только сам модуль), что делает обновления HMR стабильно быстрыми, независимо от размера вашего приложения.

Vite также использует заголовки HTTP для ускорения полной перезагрузки страницы (опять же, дайте браузеру делать больше за нас): запросы модулей исходного кода делаются условными через `304 Not Modified`, а запросы модулей зависимостей строго кэшируются через `Cache-Control: max-age=31536000,immutable`, поэтому они больше не попадают на сервер после кеширования.

Мы очень сомневаемся, что как только вы почувствуете, насколько быстр Vite, вы снова захотите мириться с разработкой на основе бандлов.

## Зачем нужен пакет для production

Несмотря на то, что нативный ESM в настоящее время широко поддерживается, доставка разделенного ESM в production по-прежнему неэффективна (даже с HTTP/2) из-за дополнительных сетевых циклов, вызванных вложенными импортами. Чтобы получить оптимальную производительность загрузки в production среде, все же лучше объединить свой код с tree-shaking, отложенной загрузкой и разделением общих фрагментов (для лучшего кеширования).

Обеспечить оптимальный вывод и согласованность поведения между сервером разработки и production сборкой непросто. Вот почему Vite поставляется с предварительно сконфигурированной [командой сборки](./build), которая "из коробки" включает многие [оптимизации производительности](./features#build-optimizations).

## Почему не использовать бандл с esbuild?

Несмотря на то, что `esbuild` молниеносно быстрый и уже является очень мощным сборщиком для библиотек, некоторые важные функции, необходимые для сборки _приложений_, все еще находятся в стадии разработки - в частности, разделение кода и обработка CSS. На данный момент Rollup более зрелый и гибкий в этом отношении. Тем не менее, мы не исключаем возможность использования `esbuild` для production сборки, когда он стабилизирует эти функции в будущем.

## Чем Vite отличается от X?

Вы можете перейти на страницу [Сравнения](./comparisons) для получения более подробной информации о том, чем Vite отличается от других подобных инструментов.
