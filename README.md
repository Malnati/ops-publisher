<!-- README.md -->
# ⚙️ Ops Publisher

[![GitHub Release](https://img.shields.io/github/v/release/Malnati/ops-publisher?style=for-the-badge&color=blue)](https://github.com/Malnati/ops-publisher/releases)

**Publicador GitOps de arquivos via branch dedicada + Pull Request derivada + comentário no timeline.**

O **Ops Publisher** recebe um arquivo do repositório (Markdown/JSON/CSV/etc), comita esse arquivo em uma branch padronizada e cria (ou reutiliza) uma **Pull Request derivada** apontando para a **branch de origem** da PR informada. Em seguida, publica uma mensagem no timeline dessa PR usando template.

---

## ✅ O que esta Action faz

- Valida inputs obrigatórios (`token`, `pr_number`, `attached_file_path`, `pr_template_path`, `timeline_template_path`)
- Lê metadados da PR fonte (`head`, `base`, SHAs, URL, título) via `gh`
- Calcula uma branch de publicação no padrão:  
  `branch_convention_prefix/<ultimos-10-do-sha-do-HEAD>`
- Faz commit do arquivo em `attached_file_path` nessa branch e `push`
- Cria (ou reaproveita) uma PR derivada com:
  - `base = headRefName` da PR fonte
  - `head = branch_convention` (branch de publicação)
- Renderiza:
  - corpo da PR via `templateer` (template do PR)
  - comentário do timeline via `templateer` (template do timeline)
- Publica o comentário no timeline via `pr-comment`
- Centraliza logs de erro no arquivo configurado via `errors` (via `ops-errors`)

---

## 🔐 Permissions necessárias

No workflow que chama esta Action:

```yaml
permissions:
  contents: write
  pull-requests: write
```

---

## 📦 Inputs (v3.0.0)

| Input | Obrigatório | Padrão | Descrição |
|---|:---:|---|---|
| `token` | Sim |  | Token GitHub (ex.: `secrets.GITHUB_TOKEN`) |
| `pr_number` | Sim |  | Número da PR fonte (aceita `N` ou `#N`) |
| `attached_file_path` | Sim |  | Caminho do arquivo a anexar/commitar no repo (ex.: `.reports/report.md`) |
| `branch_convention_prefix` | Não | `ops/files` | Prefixo para branches de publicação |
| `pr_title` | Não | `🛡️ Automated Pull Request` | Título da PR derivada |
| `pr_template_path` | Sim |  | Caminho do template do corpo da PR |
| `timeline_template_path` | Sim |  | Caminho do template do comentário no timeline |
| `bot_name` | Não | `git-pr-ops-bot` | Nome do autor do commit |
| `bot_email` | Não | `git-pr-ops@users.noreply.github.com` | Email do autor do commit |
| `errors` | Não | `.github/workflows/errors.log` | Arquivo para centralizar logs de erro |

---

## 🧩 Variáveis disponíveis nos templates

A Action renderiza templates via **Malnati/templateer** com variáveis de ambiente.

### Template do corpo da PR (`pr_template_path`)
Disponíveis:
- `ATTACHED_FILE_PATH` (basename do arquivo)
- `PR_NUMBER`
- `BRANCH_CONVENTION`

### Template do timeline (`timeline_template_path`)
Disponíveis:
- `ATTACHED_FILE_PATH` (basename do arquivo)
- `PR_NUMBER`
- `BRANCH_CONVENTION`
- `PR_URL` (URL da PR derivada criada/reutilizada)

---

## 🛠️ Exemplo de uso

Exemplo de workflow acionado por comentário em PR, gerando um arquivo e publicando via Ops Publisher:

```yaml
name: "Publish report"
on:
  issue_comment:
    types: [created]

permissions:
  contents: write
  pull-requests: write

jobs:
  publish:
    if: github.event.issue.pull_request != null
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate report
        shell: bash
        run: |
          mkdir -p .reports
          printf '%s
' "hello report" > .reports/report.md

      - name: Publish
        uses: Malnati/ops-publisher@v3.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.issue.number }}
          attached_file_path: .reports/report.md
          pr_title: "📋 Report"
          branch_convention_prefix: ops/files
          pr_template_path: .github/templates/report-pr.md
          timeline_template_path: .github/templates/report-timeline.md
          bot_name: mbra-com-br
          bot_email: accounts@mbra.com.br
          errors: .errors/ops-publisher.log
```

---

## 🧾 Exemplos de templates

### `.github/templates/report-pr.md`

```md
# 📎 Published file

- File: `${ATTACHED_FILE_PATH}`
- Source PR: `#${PR_NUMBER}`
- Branch: `${BRANCH_CONVENTION}`

> This PR is generated automatically by **Ops Publisher**.
```

### `.github/templates/report-timeline.md`

```md
A nova branch do arquivo `${ATTACHED_FILE_PATH}` é `${BRANCH_CONVENTION}` e a Pull Request derivada é ${PR_URL}.

---
<div align="right">
  <sub>Processado por <b>Ops Publisher</b></sub>
</div>
```

---

## ⚠️ Observações importantes

- O arquivo em `attached_file_path` é commitado **como está** no repositório. Evite anexar dados sensíveis.
- A PR derivada é criada para integrar a branch `ops/files/...` **na branch de origem** da PR informada (cadeia de PRs).
- Para evitar branches conflitantes, mantenha `branch_convention_prefix` dedicado (ex.: `ops/files`).

No fim, mantenha os templates em paths versionados no repositório (ex.: `.github/templates/*`) e use `fetch-depth: 0` no checkout para garantir consistência do `git` e do cálculo de branch.
</readme>
