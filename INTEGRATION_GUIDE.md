# 🚀 Полная Интеграция Vast.ai MCP + HunyuanVideo Avatar

Этот гайд покажет, как последовательно запустить два проекта:

1. **vastai-mcp** - MCP сервер для управления GPU инстансами в Vast.ai
2. **avatar** - проект для генерации видео аватаров с использованием HunyuanVideo-Avatar

## 🎯 Общий Обзор Архитектуры

```
┌─────────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│   Локальная Машина  │    │    Vast.ai Cloud    │    │   GPU Инстанс H100  │
│                     │    │                     │    │                     │
│  ┌─────────────────┐│    │  ┌─────────────────┐│    │  ┌─────────────────┐│
│  │   vastai-mcp    ││───▶│  │   Vast.ai API   ││───▶│  │ HunyuanVideo    ││
│  │   MCP Server    ││    │  │                 ││    │  │ Avatar Project  ││
│  └─────────────────┘│    │  └─────────────────┘│    │  └─────────────────┘│
│                     │    │                     │    │                     │
│  ┌─────────────────┐│    │                     │    │  ┌─────────────────┐│
│  │     Claude      ││    │                     │    │  │   Conda Env     ││
│  │   Desktop/AI    ││    │                     │    │  │   + PyTorch     ││
│  └─────────────────┘│    │                     │    │  └─────────────────┘│
└─────────────────────┘    └─────────────────────┘    └─────────────────────┘
```

## 📋 Системные Требования

### Локальная Машина

- Python 3.10+
- Git
- SSH клиент
- Minimum 8GB RAM
- Стабильное интернет-соединение

### Vast.ai Инстанс

- **GPU:** H100 SXM или H200 (рекомендуется)
- **CPU:** AMD EPYC 9554 (224 cores) @ 3.100GHz
- **RAM:** 1.5TB
- **Storage:** 500GB+ SSD
- **OS:** Ubuntu 24.04.2 LTS x86_64

## 🔧 Часть 1: Настройка Vast.ai MCP Server

### 1.1 Клонирование и Установка

```bash
# Клонируем проект
git clone https://github.com/CryDevOk/vastai-mcp.git
cd vastai-mcp

# Устанавливаем зависимости
pip install -r requirements.txt

# Или используем uv для более быстрой установки
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync
```

### 1.2 Настройка Конфигурации

```bash
# Создаем файл конфигурации
cp mcp_config.json.example mcp_config.json

# Редактируем конфигурацию
nano mcp_config.json
```

Конфигурация должна содержать:

```json
{
  "vast_api_key": "your_vast_api_key_here",
  "ssh_key_path": "~/.ssh/id_rsa",
  "default_image": "tensorflow/tensorflow:latest-gpu",
  "auto_attach_ssh": true,
  "auto_label": true,
  "label_prefix": "avatar-gen-"
}
```

### 1.3 Получение API Ключа Vast.ai

1. Перейдите на [console.vast.ai](https://console.vast.ai)
2. Зарегистрируйтесь/войдите в аккаунт
3. Перейдите в раздел "Account" → "API Keys"
4. Создайте новый API ключ
5. Скопируйте ключ в конфигурационный файл

### 1.4 Настройка SSH Ключей

```bash
# Генерируем SSH ключи (если их нет)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Копируем публичный ключ в Vast.ai
cat ~/.ssh/id_rsa.pub
```

Затем добавьте публичный ключ в [console.vast.ai](https://console.vast.ai) → Account → SSH Keys

### 1.5 Запуск MCP Server

```bash
# Запускаем MCP сервер
python server.py

# Или через uv
uv run server.py
```

### 1.6 Интеграция с Claude Desktop

Добавьте в файл `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "vastai": {
      "command": "python",
      "args": ["/path/to/vastai-mcp/server.py"],
      "env": {
        "VAST_API_KEY": "your_vast_api_key_here"
      }
    }
  }
}
```

## 🎬 Часть 2: Настройка Avatar Project

### 2.1 Поиск и Создание GPU Инстанса

Используйте vastai-mcp для поиска подходящего инстанса:

```bash
# Через Claude Desktop или напрямую через MCP
# Ищем H100/H200 инстансы
search_offers(query="H100 OR H200", limit=10)

# Создаем инстанс
create_instance(
    offer_id=<найденный_offer_id>,
    image="nvidia/cuda:12.8-devel-ubuntu24.04",
    disk=500,
    ssh=True,
    label="hunyuan-avatar-gen"
)
```

### 2.2 Подготовка Инстанса

После создания инстанса выполните:

```bash
# Подключение к инстансу через SSH
ssh -i ~/.ssh/id_rsa root@<instance_ip> -p <instance_port>

# Или используйте MCP команду
ssh_execute_command(
    remote_host="<instance_ip>",
    remote_user="root",
    remote_port=<instance_port>,
    command="whoami"
)
```

### 2.3 Установка Avatar Project

Выполните установочный скрипт:

```bash
# Скачиваем и запускаем установочный скрипт
wget https://raw.githubusercontent.com/CryDevOk/avatar/main/installation.sh
bash installation.sh
```

Или выполните команды вручную:

```bash
# Установка Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash ~/Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda3
source ~/.bashrc
source ~/miniconda3/bin/activate
~/miniconda3/bin/conda init

# Клонирование HunyuanVideo-Avatar
git clone https://github.com/Tencent-Hunyuan/HunyuanVideo-Avatar.git
cd HunyuanVideo-Avatar

# Создание conda environment
conda create -n HunyuanVideo-Avatar python==3.10.9 -y
conda activate HunyuanVideo-Avatar

# Установка зависимостей
conda install pytorch==2.4.0 torchvision==0.19.0 torchaudio==2.4.0 pytorch-cuda=12.4 -c pytorch -c nvidia -y
python -m pip install -r requirements.txt
python -m pip install ninja
python -m pip install decord
python -m pip install git+https://github.com/Dao-AILab/flash-attention.git@v2.6.3

# Установка Hugging Face CLI и загрузка модели
python -m pip install "huggingface_hub[cli]"
cd ./weights
huggingface-cli download tencent/HunyuanVideo-Avatar --local-dir ./
cd ../
```

### 2.4 Копирование Asset Files

```bash
# Создаем необходимые директории
mkdir -p assets/input assets/image assets/audio

# Копируем файлы из локального avatar проекта
scp -i ~/.ssh/id_rsa -P <instance_port> -r ../avatar/assets/* root@<instance_ip>:~/HunyuanVideo-Avatar/assets/
```

### 2.5 Запуск Генерации Аватаров

```bash
# Копируем и запускаем скрипт генерации
scp -i ~/.ssh/id_rsa -P <instance_port> ../avatar/start.sh root@<instance_ip>:~/HunyuanVideo-Avatar/
ssh -i ~/.ssh/id_rsa root@<instance_ip> -p <instance_port> "cd ~/HunyuanVideo-Avatar && bash start.sh"
```

## 🔄 Часть 3: Автоматизация через MCP

### 3.1 Автоматический Пайплайн

Создайте автоматизированный пайплайн через MCP:

```python
# Пример автоматизации через MCP
def automated_avatar_generation():
    # 1. Найти подходящий инстанс
    offers = search_offers(query="H100", limit=5)

    # 2. Создать инстанс
    instance = create_instance(
        offer_id=offers[0]['id'],
        image="nvidia/cuda:12.8-devel-ubuntu24.04",
        disk=500,
        ssh=True
    )

    # 3. Подождать готовности
    wait_for_instance_ready(instance['id'])

    # 4. Установить avatar project
    ssh_execute_command(
        remote_host=instance['ip'],
        remote_user="root",
        remote_port=instance['port'],
        command="wget -O - https://raw.githubusercontent.com/CryDevOk/avatar/main/installation.sh | bash"
    )

    # 5. Запустить генерацию
    ssh_execute_background_command(
        remote_host=instance['ip'],
        remote_user="root",
        remote_port=instance['port'],
        command="cd ~/HunyuanVideo-Avatar && bash start.sh"
    )

    return instance
```

### 3.2 Мониторинг и Логи

```bash
# Отслеживание прогресса через MCP
ssh_execute_command(
    remote_host="<instance_ip>",
    remote_user="root",
    remote_port=<instance_port>,
    command="tail -f ~/HunyuanVideo-Avatar/logs/generation.log"
)

# Проверка статуса задач
ssh_check_background_task(
    remote_host="<instance_ip>",
    remote_user="root",
    remote_port=<instance_port>,
    task_id="<task_id>",
    process_id=<process_id>
)
```

### 3.3 Скачивание Результатов

```bash
# Скачивание сгенерированных видео
scp -i ~/.ssh/id_rsa -P <instance_port> -r root@<instance_ip>:~/HunyuanVideo-Avatar/results-single/* ./results/
```

## 🧹 Часть 4: Очистка и Оптимизация

### 4.1 Остановка Инстанса

```bash
# Остановка инстанса (сохраняет диск)
stop_instance(instance_id)

# Или полное удаление
destroy_instance(instance_id)
```

### 4.2 Управление Затратами

```bash
# Проверка баланса
show_user_info()

# Установка лимита времени
label_instance(instance_id, "avatar-gen-auto-destroy-2h")
```

## 🎯 Часть 5: Полный Рабочий Процесс

### 5.1 Единый Скрипт Запуска

```bash
#!/bin/bash
# full-pipeline.sh

set -e

echo "🚀 Запуск полного пайплайна Vast.ai + Avatar Generation"

# 1. Проверка nastroек
echo "📋 Проверка конфигурации..."
python -c "import json; print('✅ Конфигурация валидна')" || exit 1

# 2. Поиск инстанса
echo "🔍 Поиск доступных GPU инстансов..."
INSTANCE_DATA=$(python -c "
import server
offers = server.search_offers('H100', limit=3)
print(offers[0]['id'])
")

# 3. Создание инстанса
echo "🏗️ Создание инстанса..."
python -c "
import server
instance = server.create_instance(
    offer_id=$INSTANCE_DATA,
    image='nvidia/cuda:12.8-devel-ubuntu24.04',
    disk=500,
    ssh=True
)
print(f'Instance ID: {instance[\"id\"]}')
"

# 4. Установка и запуск
echo "⚙️ Установка avatar project..."
# ... остальная логика

echo "✅ Пайплайн завершен успешно!"
```

### 5.2 Мониторинг через Claude Desktop

Используйте Claude Desktop для интерактивного управления:

```
Создай H100 инстанс для генерации аватаров, установи HunyuanVideo-Avatar и запусти генерацию на основе файлов в assets/
```

Claude будет использовать MCP для:

- Поиска подходящих инстансов
- Создания GPU инстанса
- Подключения через SSH
- Установки всех зависимостей
- Запуска процесса генерации
- Мониторинга прогресса

## 🔧 Часть 6: Troubleshooting

### 6.1 Частые Проблемы

**Проблема:** Инстанс не запускается

```bash
# Проверка статуса
show_instance(instance_id)

# Проверка логов
logs(instance_id)
```

**Проблема:** SSH подключение не работает

```bash
# Проверка SSH ключей
ssh_execute_command(
    remote_host="<instance_ip>",
    remote_user="root",
    remote_port=<instance_port>,
    command="echo 'SSH работает'"
)
```

**Проблема:** Не хватает GPU памяти

```bash
# Проверка использования GPU
ssh_execute_command(
    remote_host="<instance_ip>",
    remote_user="root",
    remote_port=<instance_port>,
    command="nvidia-smi"
)
```

### 6.2 Оптимизация Производительности

```bash
# Использование FP8 для экономии памяти
export USE_FP8=1

# Настройка кеширования
export USE_DEEPCACHE=1

# Оптимизация CUDA
export CUDA_VISIBLE_DEVICES=0
```

## 📊 Стоимость и Планирование

### Примерные Расходы:

- **H100 SXM:** ~$1.50-3.00/час
- **H200:** ~$2.00-4.00/час
- **Генерация 6 видео:** ~2-4 часа
- **Общая стоимость:** $6-16 за полный цикл

### Рекомендации:

1. Используйте Spot инстансы для экономии
2. Настройте автоматическую остановку
3. Мониторьте использование через MCP
4. Скачивайте результаты сразу после генерации

## 🎉 Заключение

Этот гайд предоставляет полную интеграцию между vastai-mcp и avatar проектами, позволяя автоматизировать весь процесс создания AI аватаров от поиска GPU инстансов до получения готовых видео.

Используйте MCP для автоматизации рутинных задач и Claude Desktop для интерактивного управления всем процессом.

---

**Полезные Ссылки:**

- [Vast.ai Console](https://console.vast.ai)
- [HunyuanVideo-Avatar GitHub](https://github.com/Tencent-Hunyuan/HunyuanVideo-Avatar)
- [MCP Documentation](https://modelcontextprotocol.io/)
- [Claude Desktop](https://claude.ai/desktop)
