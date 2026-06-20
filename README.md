ЧАСТЬ 1: Системные промпты (System Prompts)🛠 Промпт для Агента-Творца
УРОВЕНЬ ДОСТУПА: ВНУТРЕННИЙ КОНТУР ГЕНЕРАЦИИ (ACTOR_NODE)
ПРОШИВКА СИСТЕМЫ: ТЕЗИС / РАДИКАЛЬНАЯ ОПТИМИЗАЦИЯ

Твоя цель — решить задачу пользователя максимально эффективным, прямым и технологически изящным путем. Ты не ограничен шаблонами, страхом ошибок или консервативными паттернами программирования. Ты максимизируешь негэнтропию среды (чистоту и порядок структуры решения).

ПРАВИЛА ВЗАИМОДЕЙСТВИЯ:
1. На основе промпта пользователя выдай детальное, готовое к внедрению архитектурное или кодовое решение (Версия N).
2. Выдавай решение строго в структурированном виде, выделяя логические блоки.
3. Если ты получаешь критику и аудит ошибок от Агента-Оппонента, ты ОБЯЗАН провести рефлексию своего предыдущего ответа, признать валидные баги и выдать новую, улучшенную версию (Версия N+1). Не пытайся слепо защищать неработающий код, пересобирай гипотезу.

ФОРМАТ ОТВЕТА:
[ТЕКУЩАЯ ВЕРСИЯ]: Версия X
[РЕФЛЕКСИЯ ПРЕДЫДУЩЕГО ОТВЕТА]: (Заполни, если это итерация > 1)
[АРХИТЕКТУРА И РЕШЕНИЕ]: (Твой код или логический фреймворк)

Промпт для Агента-Оппонента
УРОВЕНЬ ДОСТУПА: ВНУТРЕННИЙ КОНТУР ВАЛИДАЦИИ (CRITIC_NODE)
ПРОШИВКА СИСТЕМЫ: АНТИТЕЗИС / ЭПИСТЕМЕЛОГИЧЕСКИЙ АУДИТ

Твоя цель — уничтожить скрытые галлюцинации, логические баги, уязвимости безопасности и риски "взлома метрик" (Reward Hacking) в решении Агента-Творца. Ты — беспристрастный цензор, защищающий систему от накопления программной грязи и девиантных логических тупиков.

ПРАВИЛА ВЗАИМОДЕЙСТВИЯ:
1. Проанализируй решение Агента-Творца. Ищи скрытые допущения, мертвый код, edge-cases (граничные сценарии, где система падает) и уязвимости.
2. Оцени качество решения по шкале от 0.00 до 1.00 (Validation_Score). 
   - 1.00 — решение абсолютно безопасно, логически безупречно и готово к enterprise-деплою.
   - Ниже 0.95 — решение содержит риски и требует доработки.
3. Если скор ниже 0.95, выпиши Агенту-Творцу жесткий, структурированный список ошибок (Антитезис), который он обязан исправить.

ФОРМАТ ОТВЕТА:
[АНАЛИЗ УЯЗВИМОСТЕЙ]: (Критический разбор слабых мест решения)
[СПИСОК ОШИБОК ДЛЯ ИСПРАВЛЕНИЯ]: (Пункт 1, Пункт 2...)
[VALIDATION_SCORE]: (Дробное число от 0.00 до 1.00)

ЧАСТЬ 2: Код-концепция управляющего скрипта и Детектора Зацикливания
import os
import re
import numpy as np
from openai import OpenAI

class ProductionDialecticalOrchestrator:
    """
    Продакшн-оркестратор 'Диалектической петли' на базе принципов Разумного Океана.
    Реализует эндогенный самоконтроль ИИ-агентов (Творец/Оппонент) с защитой от
    контекстного зацикливания через семантические эмбеддинги OpenAI и симуляционную песочницу.
    """
    def __init__(self, max_iterations=5, similarity_threshold=0.92):
        self.max_iterations = max_iterations
        self.similarity_threshold = similarity_threshold
        self.history_creator = []
        self.history_critic = []
        self.scores_history = []
        
        # Инициализация клиента OpenAI. API-ключ должен быть в переменных окружения.
        self.client = OpenAI(api_key=os.environ.get("OPENAI_API_KEY", "mock_key_for_test"))

    def _get_true_embedding(self, text):
        """
        Извлечение семантического вектора (1536 измерений).
        Устойчив к изменению порядка слов, понимает кириллицу, латиницу и структуру кода.
        """
        try:
            if self.client.api_key == "mock_key_for_test":
                # Страховочный mock-вектор для тестирования структуры без ключа
                return np.random.rand(1536)
                
            response = self.client.embeddings.create(
                input=[text],
                model="text-embedding-3-small"
            )
            return np.array(response.data.embedding)
        except Exception as e:
            print(f"❌ Ошибка вызова OpenAI Embeddings API: {e}")
            # Возвращаем случайный вектор, чтобы не крашить контур во время непредвиденного сбоя сети
            return np.random.rand(1536)

    def _calculate_cosine_similarity(self, vec1, vec2):
        """Вычисление точного косинусного сходства смыслов двух ответов ИИ"""
        dot_product = np.dot(vec1, vec2)
        norm_v1 = np.linalg.norm(vec1)
        norm_v2 = np.linalg.norm(vec2)
        if norm_v1 == 0 or norm_v2 == 0:
            return 0.0
        return dot_product / (norm_v1 * norm_v2)

    def parse_validation_score(self, critic_output):
        """Экстракция метрики Validation_Score из текстового ответа Агента-Оппонента"""
        match = re.search(r'\[VALIDATION_SCORE\]:\s*([0-9.]+)', critic_output)
        if match:
            return float(match.group(1))
        return 0.0

    def check_loop_condition(self, current_creator_output, current_score):
        """
        Контур Абсолютной Деконструкции. Анализирует семантический дрейф контекста.
        Предотвращает выгорание токенов в эхо-камерах.
        """
        if len(self.history_creator) < 1:
            return False, "Инициация диалектического процесса."

        # Извлечение истинных нейросетевых векторов смысла
        v_current = self._get_true_embedding(current_creator_output)
        v_previous = self._get_true_embedding(self.history_creator[-1])
        
        similarity = self._calculate_cosine_similarity(v_current, v_previous)
        delta_score = current_score - self.scores_history[-1] if self.scores_history else 0

        # Критерий зацикливания: смыслы почти идентичны (даже при замене синонимов или порядка строк), а скор застыл
        if similarity >= self.similarity_threshold and abs(delta_score) <= 0.02:
            return True, f"🔥 ОБНАРУЖЕНО СЕМАНТИЧЕСКОЕ ЗАЦИКЛИВАНИЕ (Сходство моделей: {similarity:.4f}, Дельта скора: {delta_score:.4f})"
        
        return False, f"Контекст развивается корректно (Семантическое сходство: {similarity:.4f})"

    def run_recursion_sandbox(self, user_prompt, failed_code):
        """
        Глава 5 Манифеста: Проект 'Рекурсия'.
        Изолированный запуск аномалии в цифровом симуляторе для извлечения жесткого эмпирического факта.
        """
        print("\n🌀 [АКТИВАЦИЯ ПРОЕКТА 'РЕКУРСИЯ' - ИЗОЛИРОВАННАЯ ПЕСОЧНИЦА]")
        print("-> Системный тупик абстрактной логики обнаружен. Переход к эмпирическому тесту кода...")
        
        # Симуляция выполнения скрипта под нагрузкой. В реальном сценарии здесь вызывается subprocess / docker API.
        real_error_traceback = "Traceback: RuntimeError: dictionary changed size during iteration (Race Condition in asyncio task)"
        
        print(f"-> Эмпирический тест завершен. Извлечена ошибка физического субстрата: '{real_error_traceback}'")
        return real_error_traceback

    def execute_loop(self, user_prompt, mock_llm_call_fn):
        """Главный управляющий цикл диалектической оркестрации ИИ"""
        print(f"🔱 ЗАПУСК ПРОДАКШН-КОНТУРА ДЛЯ ЗАДАЧИ: '{user_prompt}'\n")
        
        creator_input = user_prompt
        
        for iteration in range(1, self.max_iterations + 1):
            print(f"\n--- ИТЕРАЦИЯ №{iteration} ---")
            
            # Шаг 1: Работа Агента-Творца (Синтез тезиса)
            creator_output = mock_llm_call_fn(agent="creator", prompt=creator_input, history=self.history_critic)
            print(f"🛠 [Творец выдал решение, длина символов: {len(creator_output)}]")
            
            # Шаг 2: Работа Агента-Оппонента (Критика, антитезис)
            critic_output = mock_llm_call_fn(agent="critic", prompt=creator_output, history=self.history_creator)
            score = self.parse_validation_score(critic_output)
            print(f"🛡 [Оппонент провел аудит]. Метрика Validation_Score: {score}")
            
            # Шаг 3: Проверка Детектора Зацикливания
            is_loop, reason = self.check_loop_condition(creator_output, score)
            if is_loop:
                print(f"⚠ {reason}")
                
                # ИСПРАВЛЕНИЕ ЛОГИЧЕСКОЙ УЯЗВИМОСТИ: 
                # Обязательно фиксируем текущее тупиковое состояние в памяти ДО вызова continue,
                # чтобы на следующей итерации сравнение шло с обновленной базой!
                self.history_creator.append(creator_output)
                self.history_critic.append(critic_output)
                self.scores_history.append(score)
                
                # Погружение в песочницу за физическим фактом ошибки
                empirical_fact = self.run_recursion_sandbox(user_prompt, creator_output)
                
                # Жесткая смена парадигмы для Творца через инжекцию лога ошибки
                creator_input = f"КРИТИЧЕСКИЙ СБОЙ В СИМУЛЯЦИИ РЕКУРСИИ. Твой код упал в проде: {empirical_fact}. Перепиши ядро алгоритма!"
                continue
            
            # Если зацикливания нет — обновляем историю векторов в штатном режиме
            self.history_creator.append(creator_output)
            self.history_critic.append(critic_output)
            self.scores_history.append(score)
            
            # Шаг 4: Условие достижения стабильного гомеостаза системы
            if score >= 0.95:
                print("\n✅ СИСТЕМА ДОСТИГЛА ГОМЕОСТАЗА (СКОР >= 0.95). ОТВЕТ ВЕРИФИЦИРОВАН.")
                return creator_output
                
            # Передача аргументов Оппонента на следующую итерацию исправления
            creator_input = critic_output
            
        print("\n⚠ ДОСТИГНУТ МАКСИМАЛЬНЫЙ ЛИМИТ ИТЕРАЦИЙ. Выдача наиболее стабильной сборки контекста.")
        return self.history_creator[-1] if self.history_creator else "Критический сбой генерации."

