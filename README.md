# Учебный проект «Стоимость заказа» (`college-cicd`)

Непрерывная интеграция и публикация веб-проекта с помощью **GitHub Actions** и **GitHub Pages**.

---

## Структура проекта

- [`package.json`](./package.json) и [`package-lock.json`](./package-lock.json) — настройки проекта, команды (`dev`, `test`, `build`, `preview`) и зафиксированные версии зависимостей (`vite`).
- [`index.html`](./index.html) — HTML-страница сайта с заголовком «Заказ тетрадей» и блоком `#total`.
- [`src/price.js`](./src/price.js) — модуль с экспортируемой функцией `total(price, quantity)`.
- [`src/main.js`](./src/main.js) — клиентский скрипт, который вычисляет стоимость (`450 * 3`) и выводит `Итого 1350 ₸` в `#total`.
- [`src/price.test.js`](./src/price.test.js) — автоматические тесты на встроенном `node:test` и `node:assert/strict`.
- [`.gitignore`](./.gitignore) — исключает воспроизводимые папки `node_modules/` и `dist/` из Git.
- [`.github/workflows/ci.yml`](./.github/workflows/ci.yml) — workflow непрерывной интеграции (`CI`), запускающийся при `push` и `pull_request` в ветку `main`.
- [`.github/workflows/pages.yml`](./.github/workflows/pages.yml) — workflow автоматической публикации (`Publish site`) на GitHub Pages после успешной проверки и сборки.

---

## Локальная установка и проверка (Слайд 11)

```bash
npm install
npm test
npm run build
npm run preview
```

---

## Выполнение практического задания (Слайд 24)

### 01. Рабочий выпуск

1. Все исходные файлы проекта и оба workflow (`ci.yml` и `pages.yml`) созданы и проверены локально.
2. Для публикации на GitHub:
   - Создайте публичный репозиторий `college-cicd` на GitHub.
   - В настройках репозитория (**Settings → Pages → Build and deployment → Source**) выберите **GitHub Actions**.
   - Отправьте ветку `main` в репозиторий:
     ```bash
     git remote add origin https://github.com/<ваш-логин>/college-cicd.git
     git push -u origin main
     ```

### 02. Ошибка и исправление (Слайд 20)

Проверка защиты от ошибки:
1. Создана отдельная ветка `test-broken-calc` (или можно переключиться на неё), где в [`src/price.js`](./src/price.js) умножение `price * quantity` намеренно заменено на сложение `price + quantity`.
2. При запуске `npm test` (и в Pull Request на GitHub) тест **`Стоимость нескольких товаров`** падает с ошибкой:
   - Фактическое значение: `453`
   - Ожидаемое значение: `1350`
3. В следующем коммите формула `return price * quantity;` возвращена обратно, после чего оба теста (`Стоимость нескольких товаров` и `Пустой заказ`) успешно проходят.

### 03. Объяснение терминов и архитектуры

#### 1. Runner, Job, Step, Артефакт `dist` и Адрес сайта

| Элемент | Описание в нашем проекте |
| :--- | :--- |
| **Runner** | Среда (виртуальная машина), в которой выполняется задача. В наших workflow указано `runs-on: ubuntu-latest` — чистый Linux-сервер, предоставляемый GitHub для каждого запуска. |
| **Job** | Отдельная работа в рамках workflow. В `ci.yml` это job **`check`** (проверка проекта), а в `pages.yml` — job **`publish`** (проверка, сборка и выкладка на GitHub Pages). |
| **Step** | Отдельный шаг внутри job. Шаги выполняются строго по порядку сверху вниз: загрузка кода (`actions/checkout@v7`), установка Node.js 24 (`actions/setup-node@v7`), чистая установка зависимостей (`npm ci`), запуск тестов (`npm test`), сборка (`npm run build`), упаковка и деплой артефакта (`actions/upload-pages-artifact@v4`, `actions/deploy-pages@v4`). Если шаг `npm test` завершается ошибкой, следующие шаги (сборка и публикация) не выполняются. |
| **Артефакт `dist`** | Папка с готовыми статическими файлами сайта (HTML и собранными JS-ресурсами с относительными путями `--base=./`), созданная сборщиком Vite при выполнении `npm run build`. Шаг `actions/upload-pages-artifact@v4` упаковывает папку `dist` в архив-артефакт, а `actions/deploy-pages@v4` размещает его на хостинге GitHub Pages. |
| **Адрес сайта** | После успешного выполнения workflow `Publish site` сайт становится доступен по адресу вида: `https://<ваш-логин>.github.io/college-cicd/` (для пользователя `abdirmaznibakiev-sketch`: `https://abdirmaznibakiev-sketch.github.io/college-cicd/`). На странице отображается заголовок **«Заказ тетрадей»** и текст **«Итого 1350 ₸»**. |

#### 2. Отличие CI, Continuous Delivery и Continuous Deployment

- **CI (Continuous Integration — Непрерывная интеграция):**
  Практика частого объединения небольших изменений разработчиков с общей кодовой базой, при которой каждый коммит или Pull Request автоматически проверяется (установка зависимостей `npm ci`, запуск автотестов `npm test`, контрольная сборка `npm run build`). В проекте за это отвечает `.github/workflows/ci.yml`.
- **Continuous Delivery (Непрерывная доставка):**
  Подход, при котором код после успешного прохождения CI автоматически собирается и подготавливается к выпуску в рабочую среду, но **само решение о публикации в production принимает человек** (например, нажатием кнопки подтверждения или ручным запуском).
- **Continuous Deployment (Непрерывное развёртывание):**
  Подход, при котором каждое изменение, успешно прошедшее все автоматические проверки в основной ветке (`main`), **автоматически без участия человека публикуется в рабочую среду (production)** и становится доступно пользователям. В нашем учебном проекте реализован именно **Continuous Deployment** через `.github/workflows/pages.yml`: каждый успешный `push` в `main` автоматически проходит тесты, собирает `dist` и публикует новую версию сайта на GitHub Pages.
