# Desafio Técnico Keyrus — Bank Marketing

Desafio técnico de modelagem em Python para a vaga de Cientista de Dados Sênior na Keyrus. Enunciado completo em [`docs/Teste_Modelagem_Python.pdf`](docs/Teste_Modelagem_Python.pdf).

## Objetivo

Prever se um cliente vai assinar um depósito a prazo (`y`), a partir do dataset [Bank Marketing](https://www.kaggle.com/datasets/abdelazizsami/bank-marketing/data) (Kaggle) — réplica do dataset público do UCI (Moro et al., 2011). Descrição das colunas em [`docs/bank-names.txt`](docs/bank-names.txt).

## Estrutura

```text
.
├── bank_marketing.ipynb   # notebook único com todo o desafio (EDA → modelagem → avaliação)
├── docs/                  # enunciado do desafio e documentação do dataset
├── pyproject.toml         # dependências do projeto (gerenciado com uv)
└── uv.lock
```

## Setup

Dependências gerenciadas com [uv](https://github.com/astral-sh/uv):

```bash
uv sync
```

### Credenciais do Kaggle

O notebook baixa os dados direto do Kaggle via [`kagglehub`](https://github.com/Kaggle/kagglehub) — não há CSVs versionados no repositório. Testado sem nenhuma credencial configurada e funcionou (dataset público). Se o download falhar com erro de autenticação no seu ambiente, configure **um** destes meios:

- Token em `~/.kaggle/access_token`, ou
- Variável de ambiente `KAGGLE_API_TOKEN`, ou
- `kaggle.json` (legado) em `~/.kaggle/kaggle.json`, gerado em Kaggle → Account → Create New API Token.

## Rodando o notebook

```bash
uv run jupyter lab bank_marketing.ipynb
```

Ou abra `bank_marketing.ipynb` no VS Code e selecione o kernel do `.venv` do projeto.
