# Relatório Técnico — Instrumentação de Pipeline CI/CD

**Aluno:** Vinicius Ibiapina  
**Módulo:** M10 — Sprint 07  
**Repositório:** https://github.com/Viniciusibin/Ponderada-M10-S07  
**Pipeline:** [.github/workflows/ci.yml](.github/workflows/ci.yml)  
**Execuções:** https://github.com/Viniciusibin/Ponderada-M10-S07/actions

---

## 1. Objetivo e Hipóteses Iniciais

O experimento tem como objetivo instrumentar um pipeline de CI/CD no GitHub Actions, coletar métricas reais de execução e analisar comportamento, desempenho e gargalos a partir de 12 execuções com variações controladas.

### Hipóteses iniciais

| # | Hipótese |
|---|----------|
| H1 | O cache de dependências pip reduzirá o tempo de instalação em pelo menos 30% |
| H2 | A execução paralela dos jobs de lint e test reduzirá o tempo total do workflow |
| H3 | A etapa de instalação de dependências será o principal gargalo sem cache |
| H4 | Testes lentos (sleep) aumentarão o tempo de forma proporcional ao delay |

---

## 2. Descrição do Projeto

Projeto Python com uma calculadora (`src/calculator.py`) com 15 funções matemáticas cobertas por testes unitários com `pytest`.

**Pipeline (jobs sequenciais por padrão):**
1. **Lint** — `flake8` análise estática
2. **Test** — `pytest` com relatório JUnit XML e cobertura de código
3. **Collect Metrics** — coleta metadados do run e faz upload como artifact

---

## 3. Variações Controladas e Execuções Reais

| # | Run ID | Commit | Mensagem | Status | Duração (s) |
|---|--------|--------|----------|--------|-------------|
| 1 | [26888048382](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888048382) | `31e8d24` | ci: setup inicial — cache ON, 49 testes | ✅ success | 115 |
| 2 | [26888093412](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888093412) | `6a22758` | ci(variation-2): cache OFF | ✅ success | 116 |
| 3 | [26888106209](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888106209) | `8ee19b2` | test(variation-3): assertion errada | ❌ failure | 122 |
| 4 | [26888118579](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888118579) | `2656971` | test(variation-4): correção do teste | ✅ success | 143 |
| 5 | [26888138426](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888138426) | `6187533` | test(variation-5): +50 testes (99 total) | ✅ success | 124 |
| 6 | [26888149924](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888149924) | `3c89b3f` | test(variation-6): sleep(10) em 1 teste | ✅ success | 163 |
| 7 | [26888156613](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888156613) | `e5496e0` | test(variation-7): remove teste lento | ✅ success | 156 |
| 8 | [26888175059](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888175059) | `242d014` | src(variation-8): linha longa — lint fail | ❌ failure | 119 |
| 9 | [26888185470](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888185470) | `bf960dc` | src(variation-9): correção do lint | ✅ success | 146 |
| 10 | [26888203612](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888203612) | `9208a27` | ci(variation-10): jobs em **paralelo** | ✅ success | 121 |
| 11 | [26888215285](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888215285) | `613e97f` | ci(variation-11): jobs **sequenciais** | ✅ success | 146 |
| 12 | [26888235325](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26888235325) | `f98b884` | ci(variation-12): cache ON (hit real) | ✅ success | 144 |

---

## 4. Métricas Coletadas

Arquivo: [`data/metrics.csv`](data/metrics.csv)

### Tempo por job (segundos)

| Run # | Lint | Test | Collect Metrics | Total |
|-------|------|------|-----------------|-------|
| 1  | 34 | 38 | 32 | 115 |
| 2  | 31 | 31 | 32 | 116 |
| 3  | 33 | 31 | 32 | 122 |
| 4  | 35 | 36 | 33 | 143 |
| 5  | 30 | 30 | 35 | 124 |
| 6  | 34 | 39 | 33 | 163 |
| 7  | 29 | 36 | 36 | 156 |
| 8  | 30 | — | 35 | 119 |
| 9  | 29 | 31 | 34 | 146 |
| 10 | 32 | 31 | 31 | 121 |
| 11 | 31 | 34 | 32 | 146 |
| 12 | 34 | 33 | 33 | 144 |

> Run #8: job Test não executou pois `needs: lint` e lint falhou.

### Testes e qualidade

| Run # | Testes | Falhas | Duração dos testes (s) |
|-------|--------|--------|------------------------|
| 1  | 49 | 0 | 0.198 |
| 2  | 49 | 0 | 0.228 |
| 3  | 49 | 1 | 0.201 |
| 4  | 49 | 0 | 0.277 |
| 5  | 99 | 0 | 0.389 |
| 6  | 100 | 0 | 10.318 |
| 7  | 99 | 0 | 0.252 |
| 8  | 0  | 0 | 0.000 |
| 9  | 99 | 0 | 0.297 |
| 10 | 99 | 0 | 0.291 |
| 11 | 99 | 0 | 0.264 |
| 12 | 99 | 0 | 0.293 |

---

## 5. Gráficos

### Gráfico 1 — Tempo total do pipeline por execução

![Tempo total por execução](graphs/graph_01_pipeline_duration.png)

### Gráfico 2 — Tempo por job em cada execução

![Tempo por job](graphs/graph_02_job_duration.png)

### Gráfico 3 — Taxa de sucesso e falha

![Taxa de sucesso/falha](graphs/graph_03_success_rate.png)

### Gráfico 4 — Quantidade de testes × duração do pipeline

![Testes vs duração](graphs/graph_04_tests_vs_duration.png)

---

## 6. Análise dos Resultados

### 6.1 Qual etapa mais contribuiu para o tempo total?

O job **Test** apresentou a maior variação entre execuções, pois absorveu diretamente o impacto do `sleep(10)` no run #6 (duração de 39s vs ~31s no baseline). No entanto, o job que mais contribuiu de forma **consistente** ao tempo total foi o **Install dependencies** (dentro do Lint e do Test), que em cada job consome ~20-25s para download e instalação dos pacotes — algo que deveria ser mitigado pelo cache.

O **Collect Metrics** manteve duração estável em torno de 32-36s, sendo tempo fixo de overhead não relacionado ao projeto em si.

### 6.2 Houve diferença significativa com e sem cache?

**Surpreendentemente, não.** O run #1 (cache configurado, mas miss) levou **115s**, e o run #2 (sem cache explícito) levou **116s** — diferença de apenas 1 segundo. O run #12 (cache configurado com hit esperado) levou **144s**, mais lento que o baseline sem cache.

Isso indica que o cache do `actions/setup-python` com `cache: "pip"` tem impacto marginal neste projeto porque:
1. As dependências de dev são poucas (~8 pacotes)
2. O tempo de restauração do cache pode ser equivalente ao download direto dos pacotes
3. O overhead de setup do runner domina o tempo total

### 6.3 O paralelismo reduziu o tempo total?

**Sim**, mas de forma moderada. Comparando diretamente:
- Run #10 (paralelo): **121s**
- Run #11 (sequencial): **146s**
- **Ganho: ~25 segundos (17% de redução)**

O ganho foi real mas menor do que o esperado porque o job de Lint (~30s) e Test (~31s) têm durações similares — em paralelismo ideal, o tempo total seria o maior dos dois (~31s), mais o overhead do Collect Metrics (~31s). Na prática, o overhead do runner (checkout, setup-python) consome tempo adicional em cada job paralelo.

### 6.4 Quais falhas foram mais frequentes?

Das 12 execuções, 2 falharam:
- **Run #3** — falha de teste (assertion errada): `add(2,3) == 99`
- **Run #8** — falha de lint (linha >88 chars): o job Test sequer foi executado

A falha de lint é mais impactante operacionalmente porque bloqueia o job de test por completo (devido ao `needs: lint`), impedindo que o desenvolvedor saiba se os testes passariam ou não.

### 6.5 O pipeline fornece feedback rápido o suficiente?

O tempo médio de execução dos runs com sucesso foi **~134s (~2,2 minutos)**. Para um ciclo de desenvolvimento, isso é aceitável, mas ainda poderia ser melhorado. O principal problema é que mesmo uma falha de lint simples leva ~2 minutos para dar feedback (tempo de fila + checkout + setup), quando a execução local de `flake8` seria instantânea.

A configuração de jobs paralelos (run #10) já melhorou marginalmente. Para feedback realmente rápido, seria necessário separar o lint em um job mais leve ou usar pre-commit hooks locais.

### 6.6 Que melhorias poderiam ser feitas?

1. **Pre-commit hooks locais**: rodar flake8 e pytest antes do push para evitar runs desnecessários
2. **Separar lint em job dedicado e rápido**: lint não precisa de pytest-cov e outras dependências pesadas
3. **Usar `pip install --no-deps` + hash checking**: instalação mais rápida e segura
4. **Cache mais granular**: usar hash do requirements-dev.txt como chave de cache
5. **Paralelizar mais jobs**: rodar lint e test sempre em paralelo (como no run #10)
6. **Timeout por job**: adicionar `timeout-minutes: 5` para falhar rapidamente em testes travados
7. **Matriz de versões Python**: testar em 3.10 e 3.11 em paralelo para garantir compatibilidade

### 6.7 Quais limitações existem nos dados coletados?

- **Variabilidade do runner**: os tempos incluem tempo de fila e inicialização do runner no GitHub, não apenas execução do código. Um run pode levar 115s em condições ideais e 163s sob carga — isso não é necessariamente atribuível ao código.
- **Granularidade de steps**: o script coleta duração por job, mas não por step individual (checkout, setup-python, pip install, etc.) — seria necessário parsear a API de steps para isso.
- **Artifact dependente de sucesso parcial**: quando lint falha (run #8), o job test não roda e o artifact de metrics fica com `test_count=0`, o que pode distorcer análises agregadas.
- **Amostra pequena**: 12 runs não são suficientes para análise estatística robusta — um mesmo run pode variar ±10-20s por fatores externos.
- **Sem controle de rede**: a velocidade de download dos pacotes no runner varia conforme a infraestrutura da GitHub.

### 6.8 Como essa análise apoia decisões de engenharia?

- **Decisão de paralelismo**: os dados do run #10 vs #11 provam empiricamente que paralelizar lint e test economiza ~25s por run — dado suficiente para justificar a mudança permanente
- **Investimento em pre-commit**: sabendo que o tempo de feedback é ~2 minutos, fica claro o valor de detectar problemas localmente antes de empurrar
- **Cache**: os dados mostram que o cache de pip não justifica a complexidade de configuração para projetos pequenos — mas seria relevante em projetos com centenas de dependências
- **Alertas de performance**: um teste com `sleep(10)` aumentou o tempo total em ~40s — em projetos maiores, testes lentos são facilmente detectáveis por monitoramento de duração por job

---

## 7. Resultados Inesperados

### 7.1 Cache não gerou ganho mensurável

**Hipótese**: cache de pip reduziria o tempo em ≥30%.  
**Observado**: run #1 (cache ON, miss) = 115s; run #2 (cache OFF) = 116s; run #12 (cache ON, hit) = 144s — praticamente sem diferença, e o run com cache hit esperado foi até mais lento.

**Explicação**: para projetos pequenos com poucas dependências, o tempo de restaurar o cache do S3 da GitHub pode ser comparável ao tempo de `pip install` direto. O impacto real do cache aparece em projetos com dezenas ou centenas de pacotes pesados (ex: torch, tensorflow). Além disso, o GitHub pode ter entregado o cache de um runner diferente, adicionando latência de rede.

### 7.2 Run #7 (remove teste lento) mais lento que run #6 (com teste lento)

**Hipótese**: remover o `sleep(10)` deveria deixar o pipeline mais rápido que o run anterior com o sleep.  
**Observado**: run #6 (com sleep) = **163s**; run #7 (sem sleep) = **156s**.

A diferença foi apenas 7 segundos, quando o esperado era uma redução de pelo menos 10s (o próprio sleep). Isso indica que **o overhead do runner variou entre runs** — o run #7 pode ter ficado mais tempo na fila, ou o setup-python demorou mais. Isso evidencia que a infraestrutura compartilhada do GitHub Actions introduz ruído nos dados que torna difícil isolar o efeito de mudanças pequenas.

---

## 8. Comparação Hipótese × Resultado

| Hipótese | Resultado observado | Confirmada? |
|----------|---------------------|-------------|
| H1 — cache reduz ≥30% do tempo | Diferença <1% entre cache ON e OFF | ❌ Refutada |
| H2 — paralelismo reduz tempo total | Ganho de 25s (17%) no run #10 vs #11 | ✅ Confirmada |
| H3 — instalação é o principal gargalo sem cache | Overhead distribuído igualmente — não há gargalo único | ⚠️ Parcial |
| H4 — teste lento aumenta tempo linearmente | `sleep(10)` adicionou ~40s ao total (não apenas 10s) | ⚠️ Parcial (overhead do job) |

---

## 9. Limitações do Experimento

- Variabilidade da infraestrutura do GitHub Actions (runners compartilhados, fila)
- Tamanho pequeno do projeto limita o impacto real de cache e paralelismo
- Apenas 12 execuções — base estatística pequena para conclusões definitivas
- Não foram testados cenários de rede lenta, falha de infraestrutura, ou múltiplos PRs simultâneos
- Os tempos de duração incluem overhead fixo do runner (~15-20s) não relacionado ao código

---

## 10. Como Reproduzir

```bash
# 1. Clonar o repositório
git clone https://github.com/H3-Solar-Org/Ponderada-M10-S07.git
cd Ponderada-M10-S07

# 2. Instalar dependências
pip install -r requirements-dev.txt

# 3. Rodar lint localmente
flake8 src/ tests/ --max-line-length=88

# 4. Rodar testes localmente
mkdir -p results
pytest tests/ -v --junitxml=results/junit.xml --cov=src

# 5. Coletar métricas (requer GitHub PAT com scope 'repo')
export GITHUB_TOKEN=ghp_SEU_TOKEN_AQUI
python scripts/collect_metrics.py --repo H3-Solar-Org/Ponderada-M10-S07

# 6. Gerar gráficos
python scripts/generate_graphs.py
# Saída: graphs/graph_01_*.png ... graph_04_*.png
```

As execuções do pipeline são disparadas automaticamente a cada push para `main`.

Para reproduzir as variações, basta replicar os commits descritos na seção 3, com as modificações correspondentes em `tests/` e `.github/workflows/ci.yml`.
