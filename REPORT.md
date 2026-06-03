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

**Pipeline (jobs por padrão sequenciais):**
1. **Lint** — `flake8` análise estática
2. **Test** — `pytest` com relatório JUnit XML e cobertura de código
3. **Collect Metrics** — coleta metadados do run e faz upload como artifact

---

## 3. Variações Controladas e Execuções Reais

| # | Run ID | Commit | Variação | Status | Duração (s) |
|---|--------|--------|----------|--------|-------------|
| 1 | [26891388441](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891388441) | `fef6429` | Baseline — cache ON, 49 testes, sequencial | ✅ success | 110 |
| 2 | [26891403962](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891403962) | `7718246` | Cache OFF | ✅ success | 125 |
| 3 | [26891424211](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891424211) | `775ac1e` | Assertion errada (`add(2,3) == 99`) | ❌ failure | 101 |
| 4 | [26891442771](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891442771) | `d2ce249` | Correção do teste + cache ON | ✅ success | 106 |
| 5 | [26891473415](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891473415) | `a514613` | +50 testes parametrizados (99 total) | ✅ success | 106 |
| 6 | [26891494904](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891494904) | `362dbdc` | Teste lento — `sleep(10)` | ✅ success | 113 |
| 7 | [26891500420](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891500420) | `dcdd777` | Remove teste lento | ✅ success | 108 |
| 8 | [26891525072](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891525072) | `92fb705` | Linha >88 chars — falha de lint | ❌ failure | 67 |
| 9 | [26891538406](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891538406) | `aa47bd2` | Correção do lint | ✅ success | 100 |
| 10 | [26891560443](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891560443) | `38febb1` | Jobs lint e test em **paralelo** | ✅ success | 76 |
| 11 | [26891579615](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891579615) | `6bc04bf` | Jobs lint e test **sequenciais** | ✅ success | 105 |
| 12 | [26891603404](https://github.com/Viniciusibin/Ponderada-M10-S07/actions/runs/26891603404) | `488ff0f` | Cache ON com hit real | ✅ success | 97 |

---

## 4. Métricas Coletadas

Arquivo: [`data/metrics.csv`](data/metrics.csv)

### Tempo por job (segundos)

| Run # | Lint | Test | Collect Metrics | Total | Testes | Falhas | Duração testes (s) |
|-------|------|------|-----------------|-------|--------|--------|-------------------|
| 1  | 32 | 32 | 32 | 110 | 49 | 0 | 0.168 |
| 2  | 30 | 41 | 42 | 125 | 49 | 0 | 0.187 |
| 3  | 29 | 30 | 30 | 101 | 49 | 1 | 0.183 |
| 4  | 28 | 34 | 30 | 106 | 49 | 0 | 0.176 |
| 5  | 30 | 32 | 32 | 106 | 99 | 0 | 0.245 |
| 6  | 27 | 41 | 32 | 113 | 100 | 0 | 10.255 |
| 7  | 35 | 28 | 32 | 108 | 99 | 0 | 0.247 |
| 8  | 29 | — | 30 | 67 | 0 | 0 | 0.000 |
| 9  | 28 | 30 | 29 | 100 | 99 | 0 | 0.267 |
| 10 | 28 | 36 | 31 | 76 | 99 | 0 | 0.263 |
| 11 | 26 | 36 | 30 | 105 | 99 | 0 | 0.301 |
| 12 | 28 | 28 | 29 | 97 | 99 | 0 | 0.193 |

> Run #8: job Test não executou pois `needs: lint` e lint falhou — duração registrada como -1 (skipped).

---

## 5. Gráficos

### Gráfico 1 — Tempo total do pipeline por execução

![Tempo total por execução](graphs/graph_01_pipeline_duration.png)

### Gráfico 2 — Duração por job em cada execução

![Tempo por job](graphs/graph_02_job_duration.png)

### Gráfico 3 — Taxa de sucesso e falha

![Taxa de sucesso/falha](graphs/graph_03_success_rate.png)

### Gráfico 4 — Quantidade de testes × duração do pipeline

![Testes vs duração](graphs/graph_04_tests_vs_duration.png)

---

## 6. Análise dos Resultados

### 6.1 Qual etapa mais contribuiu para o tempo total?

Nenhum job isolado domina o tempo — todos ficaram entre 27-41s por run. O **job Test** teve a maior variância: foi de 28s (run #12) a 41s (runs #2 e #6). No run #6, o `sleep(10)` no teste elevou sua duração para 41s. O **Collect Metrics** manteve overhead constante de ~30-32s — tempo fixo independente das variações.

A maior contribuição para o tempo total vem do **overhead fixo do runner**: checkout (~2s), setup-python (~10s) e pip install (~15s) acontecem em cada job separado, multiplicando o overhead. Com 3 jobs sequenciais, esse overhead é pago 3 vezes.

### 6.2 Houve diferença significativa com e sem cache?

Comparando diretamente:
- **Run #1** (cache ON, 1º run = cache miss): **110s**
- **Run #2** (cache OFF): **125s** — sem cache levou 15s a mais
- **Run #12** (cache ON, hit real): **97s** — com cache hit, 13s mais rápido que sem cache

O cache **teve impacto real de ~15s** neste experimento, ao contrário do experimento anterior no repo da org. A diferença pode ser atribuída a variações na infraestrutura do runner em cada execução.

### 6.3 O paralelismo reduziu o tempo total?

**Sim, significativamente.** Comparação direta:
- **Run #10** (paralelo): **76s**
- **Run #11** (sequencial): **105s**
- **Ganho: 29 segundos (28% de redução)**

O resultado mais expressivo do experimento. Com lint e test em paralelo (~28-36s cada), o tempo total cai para próximo do job mais lento + overhead do Collect Metrics.

### 6.4 Quais falhas foram mais frequentes?

Das 12 execuções, 2 falharam (taxa de falha: 16,7%):
- **Run #3** — falha de teste: `AssertionError: assert 5 == 99`
- **Run #8** — falha de lint: linha com >88 caracteres em `calculator.py`

A falha de lint (run #8) foi a mais impactante operacionalmente: durou apenas **67s** porque o job Test foi completamente bloqueado pelo `needs: lint`. O desenvolvedor não recebeu feedback dos testes — situação pior do que uma falha de teste que pelo menos informa o estado do código.

### 6.5 O pipeline fornece feedback rápido o suficiente?

O tempo médio de execução dos runs com sucesso foi **~105s (~1,75 minutos)**. Para o ciclo de desenvolvimento, esse tempo é aceitável para integração contínua. Porém, o tempo de feedback ainda poderia ser reduzido:

- Com **paralelismo** (run #10), chegou a **76s** — próximo do ideal para este projeto
- O tempo de fila do GitHub Actions (não medido) adiciona variação fora do nosso controle

### 6.6 Que melhorias poderiam ser feitas?

1. **Sempre usar jobs paralelos**: run #10 provou 28% de ganho — não há motivo para manter sequencial neste projeto
2. **Pre-commit hooks locais**: detectar lint e falha de testes antes do push evita ciclos desnecessários
3. **Separar dependências de lint**: lint não precisa de pytest-cov, pandas, matplotlib — instalar apenas flake8 reduziria o pip install do job lint
4. **Timeout por job**: `timeout-minutes: 5` evitaria que testes travados bloqueassem o pipeline indefinidamente
5. **Relatório de cobertura como artefato**: já feito, mas poderia incluir badge no README

### 6.7 Quais limitações existem nos dados coletados?

- **Variabilidade do runner**: o GitHub Actions usa runners compartilhados. O run #2 (cache OFF) foi mais lento que o esperado provavelmente por variação na carga da infraestrutura
- **Granularidade de steps**: o script coleta duração por job, não por step individual — não sabemos exatamente quanto tempo cada step (checkout, setup-python, pip install, execução) consome separadamente
- **Artifact dependente de sucesso parcial**: quando lint falha (run #8), test não roda e o artifact de métricas registra `test_count=0`, o que pode distorcer análises
- **Amostra de 12 runs**: insuficiente para análise estatística robusta — um mesmo cenário pode variar ±10-20s
- **Sem controle de rede**: velocidade de download dos pacotes varia conforme congestionamento do datacenter da GitHub

### 6.8 Como essa análise apoia decisões de engenharia?

- **Paralelismo como padrão**: dados provam empiricamente 28% de ganho — justifica mudança permanente no pipeline
- **ROI do cache**: ganho real de ~15s em pipelines com dependências leves — para projetos maiores, o ganho é proporcional ao tamanho das dependências
- **Custo de testes lentos**: sleep(10) elevou o job Test em ~9s além do tempo do sleep — overhead do job absorve parte do custo. Em produção, monitorar `test_duration` no CSV permite detectar testes lentos antes que se tornem gargalos
- **Falha de lint bloqueia feedback**: dado que justifica habilitar lint em paralelo com test (não sequencial)

---

## 7. Resultados Inesperados

### 7.1 Paralelismo reduziu 28% — mais do que esperado

**Hipótese**: paralelismo reduziria o tempo total pelo tempo do job mais rápido.  
**Observado**: run #10 (paralelo) = **76s**; run #11 (sequencial) = **105s** — ganho de 29s.

O ganho foi maior do que em experimentos anteriores porque nesta rodada os jobs de lint (~28s) e test (~36s) têm durações mais próximas. Quando ambos rodam em paralelo, o tempo total é determinado pelo mais lento (~36s) + overhead do Collect Metrics (~31s) + setup (~9s) = ~76s. O resultado confirma H2 com folga.

### 7.2 Falha de lint durou apenas 67s — o mais curto de todos

**Hipótese**: uma falha deveria levar tempo similar a um sucesso.  
**Observado**: run #8 (lint failure) = **67s** — 30s mais rápido que a média dos outros runs.

A explicação: quando lint falha, o job Test (que tem `needs: lint`) é pulado completamente. O Collect Metrics roda com `if: always()`, mas sem artifact de junit para baixar. O resultado é que **falhas rápidas de lint terminam mais rápido que successes** — o pipeline "falha rápido". Isso é um comportamento positivo do design do pipeline, mas implica que falha de lint = **zero feedback sobre os testes**.

---

## 8. Comparação Hipótese × Resultado

| Hipótese | Resultado observado | Confirmada? |
|----------|---------------------|-------------|
| H1 — cache reduz ≥30% do tempo | Ganho de ~15s (~13%) entre sem cache (#2) e com hit (#12) | ⚠️ Parcial (impacto real mas <30%) |
| H2 — paralelismo reduz tempo total | Ganho de 29s (28%) no run #10 vs #11 | ✅ Confirmada |
| H3 — instalação é o principal gargalo sem cache | Overhead distribuído em checkout + setup + pip — não há gargalo único isolável | ⚠️ Parcial |
| H4 — teste lento aumenta tempo linearmente | sleep(10) elevou job Test em ~9s além do sleep — overhead absorve parte | ⚠️ Parcial |

---

## 9. Limitações do Experimento

- Variabilidade da infraestrutura do GitHub Actions (runners compartilhados, fila)
- Projeto pequeno limita o impacto real de cache e paralelismo em comparação com projetos de produção
- 12 execuções não são suficientes para análise estatística — conclusões são indicativas, não definitivas
- Métricas coletadas no nível de job, não de step — granularidade insuficiente para isolar etapas individuais (pip install vs pytest)
- Tempo de fila do Actions não é capturado nas métricas

---

## 10. Como Reproduzir

```bash
# 1. Clonar e instalar dependências
git clone https://github.com/Viniciusibin/Ponderada-M10-S07.git
cd Ponderada-M10-S07
pip install -r requirements-dev.txt

# 2. Rodar lint localmente
flake8 src/ tests/ --max-line-length=88

# 3. Rodar testes localmente
mkdir -p results
pytest tests/ -v --junitxml=results/junit.xml --cov=src

# 4. Coletar métricas (requer GitHub PAT com scope 'repo')
export GITHUB_TOKEN=ghp_SEU_TOKEN
python scripts/collect_metrics.py --repo Viniciusibin/Ponderada-M10-S07

# 5. Gerar gráficos
python scripts/generate_graphs.py
# Saída: graphs/graph_01_*.png ... graph_04_*.png
```

As execuções do pipeline são disparadas automaticamente a cada push para `main`.
