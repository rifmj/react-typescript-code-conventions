*Текущая версия specs актуальна для проектов, которые пишутся отдельно от экосистемы. Экосистема глобально должна включать общие для проектов компоненты и сервисы.*

# Написание кода

## React-приложение

### Типы сущностей

* **Module** *abstract* - структура с файлами, собранными by feature *e.g. модуль настроек пользователя, модуль dashboard, модуль корзины, модуль визуального редактора, модуль ui*
* **Components** - зачастую UI-компоненты; не могут делать запросы к API, не должны использовать контекст для получения информации. Для хранения состояния используют useLocalStore, для того, чтобы компонент реализовывал pattern state-machine, что позволяет добиться конечности состояний компонента, что, в свою очередь, упрощает тестирование компонента. *e.g. компонент ИндикаторОнлайн не должен использовать хук useIsOnline из глобального контекста, а принимать свойство online в качестве prop*;
* **Containers** - собираются из компонентов; могут иметь local context (createLocalContext), могу обращаться к global context.  Внутри containers не должно быть встроенных роутов *e.g. ubscriptionSettings, если находимся внутри page Settings*
* **Pages** - используются в первую очередь для определения структуры конечной страницы и роутинга (могут иметь nested router) *e.g. Settings (в модуле User), которая может состоять из одного component Tab и двух сontainer - GeneralSettings, SubscriptionSettings*
* **Context** - обращаются к сервисам, используются для взаимодействия и хранения состояния компонента.
* **Service** - взаимодействие с внешними API (в том числе и с HTML API необходимо реализовывать через сервисы, т.е. не нужно добавлять лишнюю логику в слой View)

### Соглашения

* В Global Context нельзя передавать конструкторы, все Services должны быть инициализированы.
* Инициализация всех глобальных Services должна происходить в одном файле; в хранилище глобального Context приложения передаются уже инициализированные версии всех необходимых Services. Это делается для использования инстансов сервиса в глобальном хранилище.
* Внешние глобальные зависимости в Local Context должны передаваться через injections *e.g. Logger, BackendAPI*
* "Component scope" services (сервисы, которые создаются отдельные инстансы для каждого Container) должны использовать Local Context. *TODO область применения?*
* Используем глобальное состояние (useStoreState, useStoreActions) только в pages / Не используем глобальное состояние в components
* Использовать что-либо из Module вне его Context нельзя, для подобного взаимодействия должен использоваться Global Context (кроме модулей core и ui)
* Не используем внешние API внутри View и Store, только внутри Services
* Components должны быть написаны в функциональном стиле
* Services должны (могут?) быть написаны в ООП стиле
* Services ответственны за взаимодействие с внешними API-системы (браузер, обращение к API)

## Store

* Store разбит на модули, за исключением структуры app, которая хранит в себе текущие глобальные настройки системы *e.g. время открытия страницы (если необходимо взаимодействовать с подобной информацией, но она не будет отображаться - используем services, если ее нужно отображать, используем store)*
* Данные из модуля в Global Store можно выносить только в случае, если они могут быть использованы в других модулях *e.g. модуль Cart, который должен использоваться в модуле Products*
* В Global Store нельзя хранить информацию, которая относится к определенному локальному Сontext модуля *e.g. если сущность относится к внешней системе, и не влияет на внутреннюю - настройки отправки e-mail сообщений, но браузерные настройки уведомлений; интервал авто-обновления списка клиентов, но интервал проверки количества новых сообщений*

### Примерная структура Global Store

	app - хранилище для GlobalContext
	app.settings
	app.settings.notifications
	app.settings.notifications.isEnabled
	app.user - 
	app.user.id
	app.user.profile
	app.user.profile.first_name
	... модули
	cart
	cart.products
	cart.total

## Сервисы и SDK

### Отличия директории core от sdk

* SDK

В директорию sdk выносится код, который может шэриться между платформами React и React Native.
Код в данной директории не должен использовать специфичные браузерные API (например LocalStorage).
Сервисы в директории SDK должны быть также фреймворк-независимыми.

# Структура приложения

	## Структура директории src:
	### core - ядро React Web

	* = components
	* == Router - система роутинга
	* = constants
	* == routes - описание маршрутов приложения
	* == hooks - браузерные хуки

	### modules - модули

	### sdk - основное API, сервисы
	* = contexts - Глобальные Contexts
	* == AppContext - Основной Context, оборачиваем все приложение
	* === AppContext::logger - сервис логирования
	* == RealtimeContext - Контекст для WebSocket
	* = exceptions - Кастомные исключения
	* = hooks - Глобальные хуки, должны шэриться между платформами (react/react native)
	* = services - Сервисы, должны шэриться между платформами и быть фрэймворк-независимыми
	* == api - Взаимодействие с API
	* == http - Сервис HTTP-запросов (based on Axios)
	* == log - Логирование
	* == ws - Realtime-соединения

	### styles - глобальные стили и SCSS-переменные
	* = variables.scss - глобальные стили для импорта
	  ui - библиотека ui-компонентов

## Структура типичного компонента

	* ./Component
	* Component.stories.mdx
	* Component.styles.module.scss
	* Component.types.ts
	* Component.utils.ts
	* Component.tsx
	* index.ts

## Стилизация

* Общее правильно при стилизации - отделяем View от стилей. CSS-in-JS спорно, так как для этого необходимо отказаться от типичного подхода к стилизации компонентов, что добавляет сложности системе.
* Стили компонентов находятся в файле Component.styles.module.css.
* Для удобства темизации компонентов и их большей гибкости в root-стиле компонента определяем глобальные для него CSS-variables. Такой подход в стилизации позволит переиспользовать компоненты между продуктами системы; при этом базовая версия компонента внутри системы может отличаться от базовой глобальной, чтобы соответстовать айдентике конкретного продукта.

	  	.inner {
	  	  --button-color: palevioletred;
	  	  --button-text-color: var(--button-color);
	  	  --button-border-color: var(--button-color);
	  	  --button-padding: 0.25em 1em;
	  	  --button-font-size: 1em;
	  	  --button-border-radius: 3px;
	  	  --button-border: 2px solid var(--button-border-color);
	  		
	  	  display: inline-block;
	  	  border-radius: var(--button-border-radius);
	  	  font-size: var(--button-font-size);
	  	  padding: var(--button-padding);
	  	  border: var(--button-border);
	  	  color: var(--button-text-color);
	  	}

* Если в рамках одной страницы необходимо использовать несколько разных тем, то они оборачиваются в новый ThemeContext - div, с переопределением глобальных стилей (в импортируем className нужные стили перезаписываются)
* Если в компонент необходимо добавить дополнительные варианты внешнего вида (variant), то просто перезаписываем Local CSS-variables компонента, не добавляя/заменяя стилей.

	  	&.primary {
	  		--button-color: var(--global-color-primary);
	  	}

* Компоненты могут отдельно экспортировать свои варианты *e.g. PrimaryButton для Button*. При этом экспортируемый компонент должен содержать сигнатуре Omit<P, "variant">.

	  	const DangerButton: React.FC<Omit<ButtonProps, "variant">> = ({
	  	  children,
	  	}: {
	  	  children: React.ReactNode;
	  	}) => {
	  	  return <Button className={css.dangerButton}>{children}</Button>;
	  	};

* Global CSS-variables должны иметь префикс global, *e.g. --global-color-pirmary*
* Компоненты могут использовать глобальные переменные в определении локальных

  		--button-color: var(--global-color-primary);

* Для версионирования компонентов можно использовать Lerna/Abstract/Bit (?)

### Atomic Design

* Atom - Color, Opacity, Spacing, Typography (text, headings), Animation, HTML-тег *e.g. Button, --global-opacity-disabled: .5*. Атомы - отдельные html-теги и css-variables.
* Molecule - *e.g. Field - div(label,input)*
* Organism - *e.g. UserRegistrationForm - form(Field,Button)*
* Template - начинаем добавлять контекст, прокидываем контекст в дочерние сущности в виде Props (в molecules, organisms)
* Page - используется для Layout'инга *e.g. отображаем разные версии Header для страницы Dashboard и UserProfile* [why?](https://github.com/diegohaz/arc/issues/20#issuecomment-265934388)

В TCS предлагается сделать промежуточную сущность - Base organism -> Product -> Unique organism. Это позволяет менять внешний вид организмов в зависимости от продукта.

### Accessibility

В качестве основы для создания Accessible компонентов (Atom и Molecule) может использоваться библиотека Reakit. 

* Следует стандарту WAI-ARIA1.1
* По дефолту компоненты нестилизованы
* Можно использовать отдельные компоненты без импортирования всей библиотеки (с Tree Shaking)

### Примеры UI-kit

* Библиотека компонентов [ivi](https://design.ivi.ru/components/)

### Полезные ссылки

* Чек-лист для создания дизайна/сборки ui-kit'а [здесь](https://www.checklist.design/)
* Каталог российских компонентных [систем](http://designsystemsclub.ru/)

# Используемые библиотеки

* **Jest** - модульное тестирование и тестирование React-компонентов
* **Hygen** - для кодогенерации
* **Cypress** - e2e-тестирование
* **Storybook** - документация компонентов
* **Lodash** - вспомогательные функции
* **Axios** - HTTP-запросы
* **easy-peasy** - хранилище состояния, локальное хранилище состояния
* **i18next** - переводы, интернационализация
* **date-fns/date.io** - работа с датами

### Альтернативные библиотеки

Библиотеки, которые можно использовать в зависимости от поставленных задач

* react-query - HTTP-запросы с кэширование "из коробки"

При использовании пропадает необходимость в Store для компонентов, но при этом размазывается логика между слоями системы (по канону взаимодействие с внешними API должно производиться в сервисах)

* SCSS <=> CSS-in-JS
* SCSS Variables <=> CSS Variables

## Hygen

	hygen init self
	hygen generator new awesome-generator
	hygen awesome-generator new hello

Пример генерации компонента

     npx hygen core ui-component --name Title --path ui/components
     npx hygen core ui-component --name OrdersList --path modules/cart/components
     npx hygen core page --name CoursesListPage --path modules/todo/pages
