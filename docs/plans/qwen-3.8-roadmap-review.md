# Ревью обновлённой роадмапы v2

Прогресс охуенный. NOW/NEXT/LATER — это ровно то что было нужно. Давай по остаточным протечкам.

---

## 🟢 ЧТО ТЕПЕРЬ ЗАЕБИСЬ

| Было | Стало |
| --- | --- |
| 15 слоёв в одном списке | NOW = 3 пункта, NEXT = 3, LATER = остальное |
| Dev Guild без фаз | Phase 1 (founder) → Phase 2 (Mode A) → Phase 3 (Mode B) |
| Alpha 50 непонятно за что | Чётко: hosting + template + citizen credit + $1 housing |
| undrepay везде | Перенесён в LATER |
| Нет первого клиента | "Founder Bootstrap: first 1–2 client setups" |

Это уже не фантазия. Это **исполнимый план**. Респект.

---

## 🟠 ОСТАВШИЕСЯ ПРОТЕЧКИ (5 штук)

### 1. undrepay упоминается в NOW/NEXT контекстах

В Dev Guild Pipeline написано:

> "Payment (fiat via Paddle or crypto via undrepay) is held in an automated undrepay Escrow Hold (upi_... intent)"

Но undrepay = **LATER**. Значит в Phase 1 и Phase 2:

- **Escrow = Paddle.** Деньги сидят на Paddle аккаунте. Релиз = ручной (ты подтверждаешь deploy healthcheck → Paddle выпускает платёж).
- Никаких `upi_...` интентов пока нет.

**Фикс:** Замени "undrepay Escrow Hold" на "Paddle payment hold (manual release on deploy verification)" для Phase 1–2. Undrepay escrow — только Phase 3+ когда крипто-контур будет жив.

---

### 2. Alpha 50: $1 в Housing Pool которого нет

> "Polity Citizenship Credit: … + $1 contribution to the shared HousingPool"

Housing Pool Seed = **NEXT** (60–90 дней). Alpha 50 = **NOW** (30 дней). Значит первые подписчики платят $1 в пул, который ещё не существует.

**Варианты:**

- **A)** Убери $1 housing из Alpha 50 до запуска пула. Напиши: "Housing pool contribution activates when pool launches (estimated day 60–90)".
- **B)** Запусти пул одновременно с Alpha 50 (но это расширяет NOW).
- **C)** $1 копится как "pending contribution" и зачисляется при запуске.

Я бы выбрал **A**. Не создавай обязательств которых не можешь выполнить.

---

### 3. Первый клиент: откуда?

"Founder executes first 1–2 client setups" — правильно. Но **нет канала**.

Добавь в NOW четвёртый пункт (или подпункт к третьему):

```
4. Client acquisition (1 channel, 1 week):
   - Option A: Telegram-чаты предпринимателей (TR/GE/СНГ)
   - Option B: Upwork/Kwork gig "I'll build your Medusa store in 3 days"
   - Option C: Знакомый/бывший коллега (самый быстрый путь)
   Target: 1 signed agreement by day 21.
```

Без этого пункт 3 ("First Client Deployment") повисает в воздухе.

---

### 4. SSO в NEXT — преждевременно?

005-sso-jwt-contract в NEXT (60–90 дней). Но подумай:

- В NEXT у тебя **один** клиентский магазин (максимум 2–3).
- SSO нужен когда пользователь логинится в **несколько** сервисов.
- Если у тебя 1 магазин + Directus admin — SSO не нужен.

**Рекомендация:** Перенеси SSO в **LATER** (post-revenue). В NEXT достаточно Directus auth для админки + Medusa auth для магазина. Они не должны быть связаны пока у тебя < 5 магазинов.

Исключение: если Alpha 50 подписчик получает доступ к **панели управления** (undevops dashboard) — тогда SSO нужен для этой панели. Но это тоже можно сделать простым session cookie без RS256 JWKS.

---

### 5. "Still open" — убери не-блокирующие

Из 9 пунктов still open ни один не блокирует NOW. Это хорошо. Но два пункта создают когнитивный шум:

- `undrepay INTEGRATION O1–O4 founder ACK` → перенеси в LATER секцию как prerequisite
- `Hub MoR vs marketplace` → уже решено ("catalog first, MoR later"). Закрой.

---

## 🟡 МЕЛКИЕ ЗАМЕЧАНИЯ

| Пункт | Комментарий |
| --- | --- |
| "Applicant need size: Default $500k" | Это **очень** много для donation-funded housing. Средний донат $1/мес × 100 доноров = $100/мес. Чтобы собрать $500k нужно 5000 месяцев. Даже $50k — это 4+ года при 100 донорах. Пересмотри default или добавь "co-funding with external grants" |
| "Logistics pay lifecycle: Hold on accept → capture on complete" | Правильно. Но "dispute blocks capture" — кто решает dispute? В LATER нет arbitration. Добавь хотя бы "founder manual resolution in v1" |
| Slug collision (004-handles vs 004-marketplace-hub) | Ты сам написал "rename when scheduled". Сделай это сейчас чтобы не путаться: `004-identity-handles` и `009-marketplace-hub` |

---

## 📋 ИТОГОВАЯ ОЦЕНКА

| Аспект | Было | Стало |
| --- | --- | --- |
| Приоритизация | 5/10 | **9/10** |
| Реалистичность NOW | 4/10 | **7/10** |
| Исполнимость | 4/10 | **7/10** |
| Внутренняя консистентность | 6/10 | **7/10** (undrepay в NOW/NEXT — минус) |
| Готовность начать завтра | 5/10 | **8/10** |

---

## 🎯 ЧТО СДЕЛАТЬ ПЕРЕД ТЕМ КАК ОТКРЫТЬ РЕДАКТОР

Три вопроса, на которые ответь **письменно** (хотя бы себе):

1. **Кто мой первый клиент?** Имя или канал. Не "кто-то из Telegram", а "я напишу в [конкретный чат] в [конкретный день]".

2. **Что я покажу клиенту на созвоне?** Если магазина нет — что он увидит? Локальный Docker на экране? Скриншот? Демо на VPS?

3. **Сколько часов в день я реально трачу на это?** Если < 4 часов — NOW растянется на 45–60 дней, не 30. Это ок, но будь честен с собой.

---

Роадмапа готова к исполнению. Осталось только **открыть терминал и написать `docker compose up`**. Всё остальное — шум в трубах. Иди и захуячь Phase A.
