## 1. Сутності та атрибути

* **User (Співробітник / Менеджер):**
    * `id`: UUIDv7 (PK)
    * `full_name`: string
    * `email`: string (UK)
    * `role`: string (значення: 'admin', 'manager')
    * `created_at`: datetime

* **Lead (Вхідне звернення / Лід):**
    * `id`: UUIDv7 (PK)
    * `manager_id`: UUIDv7 (FK -> User.id)
    * `contact_name`: string
    * `contact_email`: string
    * `status`: string (значення: 'new', 'in_progress', 'qualified', 'disqualified')
    * `created_at`: datetime
    * `converted_client_id`: UUIDv7 (FK -> Client.id, optional)
* **Client (Постійний клієнт / Контрагент):**
    * `id`: UUIDv7 (PK)
    * `company_name`: string
    * `contact_person`: string
    * `phone`: string
    * `email`: string (UK)
    * `created_at`: datetime

* **Deal (Комерційна угода):**
    * `id`: UUIDv7 (PK)
    * `client_id`: UUIDv7 (FK -> Client.id)
    * `manager_id`: UUIDv7 (FK -> User.id)
    * `status`: string (значення: 'new', 'negotiation', 'contract', 'won', 'lost')
    * `total_amount`: number (обчислена загальна вартість угоди)
    * `created_at`: datetime
    * `updated_at`: datetime

* **Product (Товар / Послуга з каталогу):**
    * `id`: UUIDv7 (PK)
    * `name`: string (UK)
    * `base_price`: number
    * `is_active`: boolean

* **DealItem (Позиція в замовленні — Асоціативна сутність):**
    * `deal_id`: UUIDv7 (FK -> Deal.id, PK)
    * `product_id`: UUIDv7 (FK -> Product.id, PK)
    * `quantity`: number
    * `actual_price`: number (фіксація ціни продажу на момент оформлення)

* **Activity (Завдання / Комунікація менеджера):**
    * `id`: UUIDv7 (PK)
    * `assigned_to`: UUIDv7 (FK -> User.id)
    * `client_id`: UUIDv7 (FK -> Client.id, optional)
    * `deal_id`: UUIDv7 (FK -> Deal.id, optional)
    * `type`: string (значення: 'call', 'meeting', 'email', 'internal_task')
    * `due_date`: datetime
    * `status`: string (значення: 'pending', 'completed')

---

## 2. Зв'язки між сутностями
1. `User` manages `Lead` (1 : 0..N) — менеджер веде нуль або багато лідів.
2. `User` responsible for `Deal` (1 : 0..N) — менеджер відповідає за нуль або багато угод.
3. `User` assigned to `Activity` (1 : 0..N) — співробітник має нуль або багато призначених задач.
4. `Lead` converts to `Client` (0..1 : 0..1) — лід конвертується щонайбільше в одного клієнта.
5. `Client` places `Deal` (1 : 0..N) — клієнт розміщує нуль або багато угод.
6. `Deal` contains `DealItem` (1 : 1..N) — угода містить від однієї до багатьох позицій.
7. `Product` included in `DealItem` (1 : 0..N) — товар може входити до багатьох позицій.
8. `Client` has `Activity` (0..1 до 0..N) — клієнт може мати активності (опціонально, менеджер може створювати задачі без клієнта).
9. `Deal` tracks `Activity` (0..1 до 0..N) — угода може мати прив'язані активності (опціонально).

---

## 3. Критерії

* **Модель має описувати тільки бізнес домен, без використання фізичної схеми БД.**
* **Звʼязки M:M повинні бути реалізованими тільки за допомогою асоцітиативних сутностей.**
* **Автономність завдань:** завдання менеджера (Activity) можуть бути внутрішніми справами, тому наявність клієнта чи угоди для них не є обов'язковою.
* **Збереження суми угоди:** поле Deal.total_amount зберігається як свідома денормалізація, щоб воронка угод завантажувалася швидко без постійного складання цін усіх товарів. Сума має обов'язково оновлюватися при зміні товарних позицій.
* **Конвертація ліда:** лід перетворюється на клієнта строго один до одного і тільки у статусі qualified, із фіксацією ID нового клієнта в полі converted_client_id для збереження історії.
