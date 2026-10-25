# Сборка ImagineBook в IntelliJ IDEA (Minecraft 1.21.11)

Инструкция для компиляции мода на **Fabric** и **NeoForge** для Minecraft **1.21.11**.
Проект использует мульти-версионную сборку через плагин [Stonecutter](https://github.com/kikugie/stonecutter).

## 1. Требования

| Компонент | Версия |
|---|---|
| JDK (Java) | **21** (рекомендуется Temurin / Microsoft OpenJDK) |
| IntelliJ IDEA | Community или Ultimate (поддержка Gradle встроена) |
| RAM | минимум **4 ГБ** свободной памяти для Gradle |
| Интернет | для загрузки зависимостей (Maven Fabric/NeoForge, Yarn, ModMenu) |

Gradle ставить не нужно — проект использует Gradle Wrapper (`gradlew`), который сам скачает нужную версию.

## 2. Открытие проекта в IntelliJ IDEA

1. `File → Open…` → выберите папку репозитория (где лежит `settings.gradle.kts`).
2. Дождитесь завершения индексации Gradle (первый загрузочный sync может занять несколько минут — скачиваются Loom, Yarn-Mappings, Fabric API и т.д.).
3. Проверьте JDK: `File → Project Structure → Project → SDK` → выберите **JDK 21**, Language level **21**.
   Также: `Settings → Build, Execution, Deployment → Build Tools → Gradle → Gradle JVM` → **JDK 21**.

> В репозитории активной версией стоит `1.21.11-fabric` (см. `stonecutter.gradle.kts`).
> Все остальные версии Minecraft активны в `settings.gradle.kts`. Полная настройка проекта
> (все ~23 подсборки) требует значительной памяти: при проблемах увеличите `org.gradle.jvmargs`
> в `gradle.properties` (например, до `-Xmx8G`) или временно закомментируйте ненужные версии
> в `settings.gradle.kts`, оставив только `mc("fabric", "1.21.11")` и `mc("neoforge", "1.21.11")`.

### ⚠️ Ошибка «Gradle 8.x несовместим / extendsFrom» — быстрое решение

Если IDEA показывает **«Ошибка совместимости Gradle»** (сообщение про `extendsFrom` или
несовместимость версии Gradle с API), это почти всегда **не реальная ошибка сборки**, а следствие
того, что IDEA по умолчанию пытается импортировать проект собственным встроенным Gradle
(«Use Gradle from → Specified location» / Gradle distribution bundled with IDEA).

Решение — заставить IDEA использовать **Gradle Wrapper** (тот же Gradle 8.14, что и в консоли):

1. `Settings → Build, Execution, Deployment → Build Tools → Gradle`
2. **Distribution Source** → выбрать **`Wrapper`** (в старых версиях IDEA: *Use Gradle from → gradle-wrapper.properties file*).
3. **Gradle JVM** → **JDK 21** (проект не соберётся на JDK 17).
4. Нажмите OK и выполните `File → Sync Project with Gradle Files` (иконка «бабочки»/🔄 в панели Gradle).

Дополнительно можно снять галочку *«Configure on demand»* и включить
*「Build and run using: Gradle」* в том же окне настроек Gradle.

После этого ошибка совместимости исчезнет, а проект будет синхронизироваться той же
конфигурацией, что и `./gradlew` из терминала.

## 3. Компиляция

### Вариант А: через панель Gradle в IDEA
Откройте правую панель **Gradle** и выполните задачи:

- `ImagineBook → Tasks → project → chiseledBuildFabric` — собрать **все Fabric-версии**
- `ImagineBook → Tasks → project → chiseledBuildNeoForge` — собрать **все NeoForge-версии**
- `ImagineBook → Tasks → project → chiseledBuild` — собрать **вообще все активные версии**

### Вариант Б: через встроенный терминал IDEA (`Help → Show Log… / Terminal`)
```bash
# Только активная версия (1.21.11-fabric):
./gradlew build            # Linux/macOS  (в Windows: gradlew.bat build)

# Все Fabric-версии:
./gradlew chiseledBuildFabric

# Все NeoForge-версии:
./gradlew chiseledBuildNeoForge

# Всё сразу:
./gradlew chiseledBuild
```

Если нужен только релизный JAR без лишних шагов (test и т.п.):
```bash
./gradlew :1.21.11-fabric:buildAndCollect
./gradlew :1.21.11-neoforge:buildAndCollect
```

## 4. Где искать готовые файлы (.jar)

После сборки моды лежат здесь:

```
versions/1.21.11-fabric/build/libs/imaginebook-<версия>-fabric.jar
versions/1.21.11-neoforge/build/libs/imaginebook-<версия>-neoforge.jar
```

(при сборке через Stonecutter артефакты также собираются в `build/libs/<версия>/<loader>/`).

Готовый JAR можно положить в папку `.minecraft/mods`. Для игры нужны также:
- **Fabric**: [Fabric Loader](https://fabricmc.net/use/) ≥ 0.20.1 + [Fabric API](https://modrinth.com/mod/fabric-api) 0.141.6+1.21.11 (+ ModMenu 17.0.1 опционально)
- **NeoForge**: [NeoForge](https://neoforged.net/) ≥ 21.11.48

## 5. Запуск тестового клиента из IDE

Необязательно, но удобно для проверки:
```bash
./gradlew :1.21.11-fabric:runClient      # запустит Minecraft 1.21.11 с модом (Fabric)
./gradlew :1.21.11-neoforge:runClient    # то же для NeoForge
```
В панели Gradle это задачи `application → runClient` внутри соответствующей версии,
либо `runActiveClientFabric` / `runActiveClientNeoForge` в корневом проекте.

## 6. Частые проблемы

| Проблема | Решение |
|---|---|
| Ошибка памяти при sync/сборке | Увеличьте heap: `Settings → Gradle → Gradle daemon memory` или отредактируйте `org.gradle.jvmargs=-Xmx4G` в `gradle.properties` |
| «Unsupported class file major version» | В Project Structure и в Gradle JVM выбран JDK < 21 — переключите на 21 |
| Долгая первая сборка | Нормально: Loom скачивает и декомпилирует ядро Minecraft; последующие сборки быстрее |
| Зависимости не скачиваются | Проверьте доступ к `maven.fabricmc.net`, `maven.neoforged.net` (VPN/файрвол) |
| Нужна другая версия MC | Выполните `./gradlew stonecutter switch <версия>` (например `1.21.11-neoforge`) или поменяйте строку `stonecutter active` в `stonecutter.gradle.kts` |

## 7. Автосборка на GitHub (без IDE)

В репозитории настроен GitHub Actions (`.github/workflows/build.yml`): при каждом push/PR
проект собирается автоматически, а JAR-файлы доступны во вкладке **Actions → последний run → Artifacts**.
