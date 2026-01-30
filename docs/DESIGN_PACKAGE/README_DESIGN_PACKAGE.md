# README DESIGN PACKAGE: ROLE-BASED GUIDE

## 🎓 Твоя роль: Backend Engineer (Track A)

Ты создаёшь **механику** экскаватора.

### Что ты сейчас делаешь?
- Реализуешь уравнения 1-8 из `03_MATHEMATICAL_SPECIFICATION.md`
- Пишешь Python код в `backend/mechanics/`
- Ежедневно коммитишь с префиксом `feat:` или `fix:`

### Файлы для чтения (в порядке):
1. **00_START_HERE.md** (15 мин) — быстрый обзор
2. **03_MATHEMATICAL_SPECIFICATION.md** (уравнения 1-8) — формулы + код
3. **QUICK_REFERENCE.md** — шпаргалка при кодинге
4. **04_SOFTWARE_ARCHITECTURE.md** — как интегрировать
5. **05_DEVELOPMENT_ROADMAP.md** — недельные планы

### Твой первый коммит (TRACK A):

```bash
git checkout -b feat/kinematics
# Реализуешь уравнения 1-2 (forward kinematics)
# Пишешь тесты
# Коммитишь:
git commit -m "feat: implement forward kinematics (eq. 1-2)

- Boom FK with link length 6.45m
- Full 3-link FK (boom + arm + bucket)
- Unit tests with ±0.5° tolerance vs CAT specs
- Fixes #123"
```

### Checklist перед каждым коммитом:

```
☐ Код проходит Ruff (ruff check + ruff format)
☐ Bandit нет критических: bandit -r backend/
☐ Тесты зелёные: pytest backend/mechanics/tests/
☐ Покрытие ≥85%: pytest --cov=backend/mechanics
☐ Коммит с информативным сообщением
☐ Тикет в GitHub Issues (fixes #NNN)
```

---

## 🤖 Если ты ML Engineer (Track B)

### Что ты сейчас делаешь?
- Реализуешь уравнения 9-15 из `03_MATHEMATICAL_SPECIFICATION.md`
- Пишешь Python код в `backend/ml/`
- Готовиш ML модель к продакшену

### Файлы для чтения:
1. **00_START_HERE.md** (15 мин)
2. **03_MATHEMATICAL_SPECIFICATION.md** (уравнения 9-15) — feature engineering + models
3. **QUICK_REFERENCE.md** — sensor specs + constants
4. **02_REFERENCE_EXCAVATOR_SPEC.md** — какие датчики есть
5. **04_SOFTWARE_ARCHITECTURE.md** — как интегрировать в API

### Твой первый коммит (TRACK B):

```bash
git checkout -b feat/feature-extraction
# Реализуешь уравнения 9-12 (feature extraction)
# Генерируешь синтетические данные
# Коммитишь:
git commit -m "feat: implement feature extraction pipeline (eq. 9-12)

- Pressure derivative dP/dt detection
- Variance-based cavitation indicator
- Thermal gradient for runaway detection
- Correlation matrix for multivariate anomalies
- Unit tests on synthetic data
- Fixes #456"
```

---

## 👨‍💼 Если ты Tech Lead / Architect (Track C)

### Что ты сейчас делаешь?
- Спроектировал архитектуру (уже готова)
- Управляешь двумя потоками (Track A + Track B)
- Обеспечиваешь интеграцию + тестирование

### Файлы для чтения:
1. **01_FOUNDATIONAL_ARCHITECTURE.md** — стандарты + механика
2. **04_SOFTWARE_ARCHITECTURE.md** — вся система
3. **05_DEVELOPMENT_ROADMAP.md** — недельные спринты
4. **MANIFEST.md** — версионирование + зависимости
5. **COMPLETION_CHECKLIST.md** — что проверять перед release

### Твоя задача этой недели:

```
☐ Убедись, что Track A разработчик читает нужные файлы
☐ Убедись, что Track B разработчик читает нужные файлы
☐ Сегодня оба должны создать первые PR
☐ PR должны пройти: Ruff + Bandit + Tests
☐ Plan integration точки между механикой и ML
```

### Как запускать локально:

```bash
# 1. Clone
git clone https://github.com/hydrosim-dev/hydrosim.git
cd hydrosim

# 2. Docker
docker-compose up

# 3. Тесты
pytest backend/

# 4. Lint + format
ruff check backend/
ruff format backend/

# 5. Security scan
bandit -r backend/
```

---

## 📊 Team Allocation (примерно)

```
Track A (Mechanics):        1 Backend Engineer
Track B (ML):               1 Backend Engineer  
Track C (Architecture):     1 Tech Lead
Infrastructure/Testing:    1 DevOps Engineer (part-time)
─────────────────────────────────────
Total:                      3-4 человека
Duration:                   16 недель
Budget:                     312 часов
```

---

## 🔄 Communication Protocol

**Daily Standup** (15 min, 10 AM):
- Что сделал вчера?
- Что делаю сегодня?
- Есть ли блокеры?

**Weekly Sync** (1 hour, Friday):
- Review PRs
- Planning next sprint
- Address blockers

**Async communication:**
- GitHub Issues (для задач)
- GitHub Discussions (для вопросов)
- Slack #hydrosim-dev (для срочного)

---

## 🎯 Success Criteria (по неделям)

| Week | Milestone | Track | Success |
|------|-----------|-------|---------|
| 1-2 | Foundation | All | `docker-compose up` зелёный |
| 3-4 | Mechanics | A | FK/IK полностью реализованы |
| 5-6 | ML | B | Feature extraction + модели готовы |
| 7-8 | Integration | C | API endpoints работают end-to-end |
| 9-12 | Hardening | All | p90 <100ms, 85% test coverage |
| 13-16 | Deploy | C | Production ready |

---

**Last updated:** 2026-01-30
