# daju1-articles-memberl

## Итог работающих методов

### 1. Мониторинг памяти 
```python
import os, gc, tracemalloc
tracemalloc.start()

def mem_report(label=""):
    with open(f"/proc/{os.getpid()}/status") as f:
        for line in f:
            if line.startswith("VmRSS:"):
                rss_kb = int(line.split()[1])
                print(f"[MEM {label}] RSS: {float(rss_kb) / 1024:.1f} MB")
                return
```

### 2. Конвертация символьных объектов в числа в `calc_thrust`
Перед `return result` — преобразует тяжёлые символьные деревья SageMath в Python `float`/`complex`, освобождая память сразу после выхода из функции.

### 3. Удаление тензоров после всех ветвей
```python
for result_list in results_omega:
    for result in result_list:
        for tensor_name in ['stress_tensor', 'maxwell_stress_tensor',
                            'convective_stress_tensor_PE', 'convective_stress_tensor_IH']:
            result.pop(tensor_name, None)
plt.close('all')
gc.collect()
```

### 4. Мониторинг памяти в цикле `scan_parameter`
```python
rss = get_rss_mb()
print(f"[MEM] RSS: {rss:.1f} MB")
if rss > MEM_THRESHOLD_MB:
    # сохранить checkpoint и break
```

### 5. Режим тестирования
```python
TEST_MODE = True
if TEST_MODE:
    use_phase_y   = False  # отключает integrate() в avg_over_y
    omega_values  = [1.1*10^5]
    nk_run        = 10
    ns_run        = 10
    steps_run     = 2
    make_plot_run = False
    log_trust_run = False
```

### 6. Очистка Out[] после сканирования
```python
forget()
get_ipython().user_ns.get('Out', {}).clear()
gc.collect()
```

### 7. Защита от несуществующей ветви
```python
if best_branch_id >= n_branches:
    best_branch_id = 0
```

### 8. Исправление `get_rss_mb()` для SageMath
```python
def get_rss_mb():
    with open(f"/proc/{os.getpid()}/status") as f:
        for line in f:
            if line.startswith("VmRSS:"):
                return float(int(line.split()[1])) / 1024.0
    return 0.0
```

---

**Что НЕ сработало и было отменено:**
- `forget()` внутри цикла по кластерам — ломал допущения Maxima
- Конвертация тензоров в `complex` — ломала `.subs()` в последующих расчётах
- `restore_assumptions()` в цикле — усугубляла накопление

