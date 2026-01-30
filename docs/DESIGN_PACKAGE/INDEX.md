# INDEX: FULL NAVIGATION HUB

## 🎯 Быстрая навигация

### Я только начинаю
→ **00_START_HERE.md** (15 мин)

### Я не помню, что искал
→ **Этот файл (INDEX.md)**

### Мне нужна шпаргалка при кодинге
→ **QUICK_REFERENCE.md** (константы + формулы)

### Мне нужны формулы
→ **03_MATHEMATICAL_SPECIFICATION.md** (15 уравнений + код)

### Мне нужно спроектировать архитектуру
→ **04_SOFTWARE_ARCHITECTURE.md** (UML + типы)

### Мне нужно спланировать спринты
→ **05_DEVELOPMENT_ROADMAP.md** (16 недель, 7 фаз)

### Мне нужно понять стандарты
→ **01_FOUNDATIONAL_ARCHITECTURE.md** (ISO/ГОСТ + теория)

### Мне нужны спецификации CAT 320D
→ **02_REFERENCE_EXCAVATOR_SPEC.md** (параметры + датчики)

---

## 📂 Все файлы по типам

### **ВХОДЫ (начни с одного)**
- 00_READ_ME_FIRST.txt
- 00_START_HERE.md
- INDEX.md (ты здесь)

### **ТЕОРИЯ & СТАНДАРТЫ**
- 01_FOUNDATIONAL_ARCHITECTURE.md
- 02_REFERENCE_EXCAVATOR_SPEC.md

### **КОД & РЕАЛИЗАЦИЯ**
- 03_MATHEMATICAL_SPECIFICATION.md (15 уравнений + Python)
- 04_SOFTWARE_ARCHITECTURE.md (UML + типы + интеграция)
- QUICK_REFERENCE.md (шпаргалка)

### **ПЛАНИРОВАНИЕ & УПРАВЛЕНИЕ**
- 05_DEVELOPMENT_ROADMAP.md (16 недель, 7 фаз)
- MANIFEST.md (версия + зависимости)

### **РОЛИ & КОМАНДА**
- README_DESIGN_PACKAGE.md (путеводитель по ролям)

### **ПРОВЕРКА & СТАТУС**
- COMPLETION_CHECKLIST.md (верификация)
- DESIGN_PACKAGE_COMPLETE.txt (финальный статус)

### **СПРАВКА**
- VISUAL_SUMMARY.txt (ASCII-арт обзор)
- FILES_COMPLETE.txt (что в каждом файле)

---

## 🎓 По ролям

### Я Backend Engineer (Track A): Механика

**Читай в порядке:**
1. 00_START_HERE.md (15 мин)
2. 03_MATHEMATICAL_SPECIFICATION.md (уравнения 1-8)
3. QUICK_REFERENCE.md
4. 04_SOFTWARE_ARCHITECTURE.md (раздел mechanics)
5. 05_DEVELOPMENT_ROADMAP.md (Phase 2)

**Твои файлы в проекте:**
```
backend/mechanics/
├─ kinematics.py
├─ dynamics.py
├─ hydraulics.py
└─ tests/
```

---

### Я ML Engineer (Track B): Диагностика

**Читай в порядке:**
1. 00_START_HERE.md (15 мин)
2. 01_FOUNDATIONAL_ARCHITECTURE.md (раздел ML)
3. 03_MATHEMATICAL_SPECIFICATION.md (уравнения 9-15)
4. 02_REFERENCE_EXCAVATOR_SPEC.md (датчики)
5. QUICK_REFERENCE.md
6. 05_DEVELOPMENT_ROADMAP.md (Phase 3)

**Твои файлы в проекте:**
```
backend/ml/
├─ features.py
├─ models.py
├─ ensemble.py
└─ tests/
```

---

### Я Tech Lead (Track C): Архитектура

**Читай в порядке:**
1. 01_FOUNDATIONAL_ARCHITECTURE.md (вся система)
2. 04_SOFTWARE_ARCHITECTURE.md (вся система)
3. 05_DEVELOPMENT_ROADMAP.md (все 7 фаз)
4. MANIFEST.md
5. README_DESIGN_PACKAGE.md (управление командой)

**Твои ответственности:**
- Скоординировать Track A и Track B
- Интеграция + тестирование
- CI/CD + deployment

---

## 📋 По фазам разработки

### Phase 1: Foundation (недели 1-2)
- Docker, CI/CD, конфиг
- Файл: 05_DEVELOPMENT_ROADMAP.md

### Phase 2: Mechanics (недели 3-4)
- Уравнения 1-8 (FK, IK, ODE, hydraulics)
- Файлы: 03_MATHEMATICAL_SPECIFICATION.md, QUICK_REFERENCE.md

### Phase 3: ML (недели 5-6)
- Уравнения 9-15 (features, LSTM, ensemble)
- Файлы: 03_MATHEMATICAL_SPECIFICATION.md, QUICK_REFERENCE.md

### Phase 4: Integration (недели 7-8)
- Соединение механики + ML
- Файлы: 04_SOFTWARE_ARCHITECTURE.md

### Phase 5: Optimization (недели 9-10)
- Профайлинг, оптимизация <100ms p90
- Файл: 05_DEVELOPMENT_ROADMAP.md

### Phase 6: Testing & Hardening (недели 11-12)
- Безопасность, 85% coverage, load testing
- Файл: COMPLETION_CHECKLIST.md

### Phase 7: Deployment (недели 13-16)
- Production release
- Файл: DESIGN_PACKAGE_COMPLETE.txt

---

## 🔗 По темам

### Kinematics (уравнения 1-3)
- 03_MATHEMATICAL_SPECIFICATION.md → Section 1
- QUICK_REFERENCE.md → "Kinematics"
- 04_SOFTWARE_ARCHITECTURE.md → kinematics.py

### Dynamics (уравнения 4-6)
- 03_MATHEMATICAL_SPECIFICATION.md → Section 2
- QUICK_REFERENCE.md → "Dynamics"
- 04_SOFTWARE_ARCHITECTURE.md → dynamics.py

### Hydraulics (уравнения 7-8)
- 03_MATHEMATICAL_SPECIFICATION.md → Section 3
- QUICK_REFERENCE.md → "Hydraulic"
- 02_REFERENCE_EXCAVATOR_SPEC.md

### Features (уравнения 9-12)
- 03_MATHEMATICAL_SPECIFICATION.md → Section 4
- QUICK_REFERENCE.md → "Features"
- 02_REFERENCE_EXCAVATOR_SPEC.md → "Sensor Data"

### Models (уравнения 13-15)
- 03_MATHEMATICAL_SPECIFICATION.md → Section 5
- QUICK_REFERENCE.md → "ML"
- 04_SOFTWARE_ARCHITECTURE.md → models.py

### Безопасность
- 01_FOUNDATIONAL_ARCHITECTURE.md → Security
- 04_SOFTWARE_ARCHITECTURE.md → Security
- COMPLETION_CHECKLIST.md → Security tests

### Тестирование
- COMPLETION_CHECKLIST.md (что тестировать)
- 05_DEVELOPMENT_ROADMAP.md → Phase 6
- QUICK_REFERENCE.md → Unit Tests Template

### CI/CD
- 05_DEVELOPMENT_ROADMAP.md → Phase 1
- README_DESIGN_PACKAGE.md → checklist

---

## ❓ F.A.Q.

**Q: С чего начать?**  
A: 00_START_HERE.md (15 мин) → выбери трек → читай файлы для своего трека

**Q: Какую команду запустить?**  
A: QUICK_REFERENCE.md → "Docker" или README_DESIGN_PACKAGE.md

**Q: Где найти формулу X?**  
A: 03_MATHEMATICAL_SPECIFICATION.md или QUICK_REFERENCE.md

**Q: Как организована кодовая база?**  
A: 04_SOFTWARE_ARCHITECTURE.md → Project Structure

**Q: Когда дедлайн?**  
A: 05_DEVELOPMENT_ROADMAP.md → 16 недель, 7 фаз

**Q: Что проверять перед коммитом?**  
A: README_DESIGN_PACKAGE.md → Checklist

**Q: Какие стандарты важны?**  
A: 01_FOUNDATIONAL_ARCHITECTURE.md → ISO/ГОСТ

**Q: Какие параметры CAT 320D?**  
A: 02_REFERENCE_EXCAVATOR_SPEC.md или QUICK_REFERENCE.md

**Last updated:** 2026-01-30
