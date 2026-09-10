# CLAUDE.md

Contexto e decisões do projeto, para persistir entre sessões. Ver também `README.md` (uso) e `docs/` (enunciado e documentação do dataset).

## Contexto

Desafio técnico de modelagem em Python para a vaga de Data Scientist na Keyrus. Prazo: 7 dias corridos a partir de 2026-09-10. Roteiro completo em `docs/Teste_Modelagem_Python.pdf`; a apresentação final não faz parte deste repositório.

## Diretrizes de trabalho combinadas

- Registrar aqui, à medida que forem tomadas, as decisões relevantes do projeto.
- Manter README.md e .gitignore atualizados.
- Boas práticas: reprodutibilidade, seeds fixas, código limpo e comentado, estrutura de pastas consistente.
- "Menos é mais": simplicidade acima de acúmulo — evitar abstrações, arquivos e dependências além do necessário para a tarefa em andamento.
- Nunca desfazer/reinterpretar decisões de sessões anteriores sem avisar antes.
- Na dúvida, perguntar antes de assumir.
- Sinceridade acima de concordância: se houver uma abordagem melhor que a pedida, avisar antes de executar.

## Decisões tomadas

- **Um único notebook** (`bank_marketing.ipynb`) para todo o desafio, na raiz do repo — conforme opção dada pelo enunciado (notebook ou script) e preferência do candidato. Reavaliar (múltiplos notebooks/scripts) apenas se a complexidade do projeto exigir; sinalizar antes de mudar.
- Tarefas do roteiro do PDF são desenvolvidas **uma de cada vez, na ordem do enunciado**.
- **Dados carregados direto do Kaggle** via `kagglehub` (`kagglehub.dataset_load` com `KaggleDatasetAdapter.PANDAS`) a cada execução — não há CSVs versionados no repo. Dataset: `abdelazizsami/bank-marketing`, arquivos `bank-full.csv` (45.211 linhas) e `bank.csv` (amostra de 10%, 4.521 linhas), carregados em dois DataFrames Pandas separados.
  - Os CSVs usam `;` como separador (herdado do dataset original UCI/Moro et al.) — passado explicitamente via `pandas_kwargs={"sep": ";"}`. Validado com uma asserção de 17 colunas após a leitura.
  - Testado sem credenciais do Kaggle configuradas e funcionou (dataset público). Caso falhe por autenticação em outro ambiente, configurar `~/.kaggle/access_token`, `KAGGLE_API_TOKEN` ou `~/.kaggle/kaggle.json` — nunca versionar, já cobertos pelo `.gitignore`.
- **Gerenciamento de dependências com `uv`** (`pyproject.toml` + `uv.lock`), Python `>=3.12` (evita a incerteza de compatibilidade do stack de ciência de dados com Python 3.14, que é muito recente).
- Dependências adicionadas apenas conforme a tarefa em andamento passa a exigi-las (ex.: `pandas`, `kagglehub[pandas-datasets]`, `jupyter` para a etapa de carregamento de dados) — nada de instalar todo o stack (sklearn, matplotlib, etc.) adiantado.
- `RANDOM_SEED = 42` definida no topo do notebook, para ser reutilizada em toda etapa que envolva aleatoriedade (split treino/teste, inicialização de modelo, etc.).
- Notebook validado de ponta a ponta com `jupyter nbconvert --execute` usando o `.venv` do projeto: `bank-full.csv` → 45.211 linhas × 17 colunas, `bank.csv` → 4.521 linhas × 17 colunas, ambos batendo com `docs/bank-names.txt`.
- **Nota importante:** o campo "Missing Attribute Values" do dataset original diz "None", mas isso é enganoso — `job`, `education`, `contact` e `poutcome` usam a categoria `"unknown"`, e `pdays = -1` é sentinela de "nunca contatado antes". Tratar isso explicitamente na etapa de EDA/tratamento de ausentes.
- **`bank_full_df` é o dataset principal de todo o projeto** a partir da tarefa de EDA (inclusive pré-processamento e modelagem). `bank_df`/`bank.csv` continua carregado na Seção 0 mas não é usado depois disso — a própria documentação do dataset o descreve como amostra de 10% só para testar algoritmos computacionalmente caros (ex. SVM), não como base de um projeto completo.
- Tarefa de EDA concluída (`## 1.` no notebook, subseções 1.1–1.4, uma por item do roteiro). Decisões da EDA:
  - Valores ausentes: `"unknown"` (job/education/contact/poutcome) e `pdays = -1` tratados como categoria/sentinela com significado próprio — sem imputação/remoção nesta etapa. Confirmado que `pdays == -1` e `previous == 0` coincidem (36.954 linhas cada) e `poutcome == "unknown"` é quase idêntico (36.959).
  - Visualizações com `matplotlib` + `seaborn` (adicionadas ao projeto via `uv add`) — únicas libs de plotagem usadas.
  - `y` é desbalanceada (~88% "no" / ~12% "yes") — vai pesar na escolha de métricas na tarefa de Avaliação do Modelo.
  - **Achado importante — vazamento de dado em `duration`:** é a variável mais correlacionada com `y`, mas só é conhecida depois que a ligação termina, não estando disponível no momento de decidir quem contatar. Retomar isso nas tarefas de Pré-processamento/Modelagem/Produção (ex.: treinar/comparar versão sem `duration` para um modelo realista).
