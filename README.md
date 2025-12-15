<!-- README.md -->
<div align="center">

# ⚙️ Ops Publisher

**Automatize a publicação de artefatos GitOps via Pull Requests Derivadas**

[![GitHub Release](https://img.shields.io/github/v/release/Malnati/ops-publisher?style=for-the-badge&color=0052CC&logo=github)](https://github.com/Malnati/ops-publisher/releases)
[![License](https://img.shields.io/github/license/Malnati/ops-publisher?style=for-the-badge&color=grey)](LICENSE)

<p align="center">
  <a href="#-como-funciona">Como Funciona</a> •
  <a href="#-uso-rápido">Uso Rápido</a> •
  <a href="#-configuração">Configuração</a> •
  <a href="#-templates">Templates</a>
</p>

</div>

---

## 🚀 Sobre

O **Ops Publisher** é uma GitHub Action projetada para fluxos de GitOps avançados. Ela captura um arquivo gerado no seu workflow (Markdown, JSON, CSV, etc.), commita em uma branch isolada e gerencia uma **Pull Request derivada** que aponta de volta para a branch da PR original.

É a solução ideal para anexar relatórios de CI, planos do Terraform ou artefatos de build diretamente no contexto da PR, sem poluir o histórico principal de imediato.

## 🧠 Como Funciona

A action executa uma lógica de "Sidecar PR":

1.  **Validação:** Verifica inputs e metadados da PR de origem.
2.  **Branching:** Calcula uma branch única baseada no SHA do commit (`ops/files/<sha-hash>`).
3.  **Commit:** Envia o arquivo selecionado para esta nova branch.
4.  **PR Derivada:** Cria (ou atualiza) uma PR que propõe merge da branch de publicação para a branch da PR original.
5.  **Notificação:** Comenta no timeline da PR original com o link para o artefato gerado.

---

## ✨ Funcionalidades

* ✅ **Gestão Automática de PRs:** Criação e reutilização inteligente de Pull Requests.
* ✅ **Templating Dinâmico:** Renderiza corpo da PR e comentários usando variáveis de ambiente.
* ✅ **Rastreabilidade:** Logs de erro centralizados e links diretos no timeline.
* ✅ **Segurança:** Suporte a tokens personalizados e permissões granulares.

---

## ⚡ Uso Rápido

Adicione este passo ao seu workflow. Certifique-se de configurar as permissões necessárias.

### Pré-requisitos
```yaml
permissions:
  contents: write
  pull-requests: write
````

### Exemplo de Workflow

Este exemplo gera um relatório e o publica sempre que um comentário é feito na PR.

```yaml
name: "Publish Report"
on:
  issue_comment:
    types: [created]

jobs:
  publish:
    if: github.event.issue.pull_request != null
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Essencial para cálculo de git history

      - name: 📝 Gerar Relatório
        run: |
          mkdir -p .reports
          echo "# Relatório de Execução" > .reports/report.md
          date >> .reports/report.md

      - name: ⚙️ Ops Publisher
        uses: Malnati/ops-publisher@v3.0.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.issue.number }}
          attached_file_path: .reports/report.md
          pr_template_path: .github/templates/report-pr.md
          timeline_template_path: .github/templates/report-timeline.md
          # Opcionais
          pr_title: "📋 Relatório Automatizado"
          branch_convention_prefix: ops/reports
          bot_name: ops-bot
          bot_email: bot@company.com
```

-----

## 📦 Configuração (Inputs)

| Input | Obrigatório | Padrão | Descrição |
| :--- | :---: | :--- | :--- |
| `token` | **Sim** | - | Token GitHub (ex.: `secrets.GITHUB_TOKEN`). |
| `pr_number` | **Sim** | - | Número da PR fonte (aceita `N` ou `#N`). |
| `attached_file_path` | **Sim** | - | Caminho do arquivo a ser publicado. |
| `pr_template_path` | **Sim** | - | Caminho do template Markdown para o corpo da PR derivada. |
| `timeline_template_path` | **Sim** | - | Caminho do template Markdown para o comentário na PR original. |
| `branch_convention_prefix` | Não | `ops/files` | Prefixo para organização das branches. |
| `pr_title` | Não | `🛡️ Automated PR` | Título da PR derivada. |
| `bot_name` | Não | `git-pr-ops-bot` | Nome do autor do commit git. |
| `bot_email` | Não | `...` | Email do autor do commit git. |
| `errors` | Não | `.github/...` | Arquivo para centralizar logs de erro. |

-----

## 🎨 Personalizando Templates

A Action utiliza o **Malnati/templateer** para renderizar variáveis nos seus arquivos Markdown.

### Variáveis Disponíveis

| Variável | Descrição | Disponível em |
| :--- | :--- | :--- |
| `${ATTACHED_FILE_PATH}` | Nome/Caminho do arquivo | Ambos |
| `${PR_NUMBER}` | Número da PR original | Ambos |
| `${BRANCH_CONVENTION}` | Nome da branch gerada | Ambos |
| `${PR_URL}` | URL da PR derivada | **Timeline** apenas |

### Exemplos

#### `.github/templates/report-pr.md` (Corpo da PR)

```markdown
# 📎 Arquivo Publicado
Este Pull Request contém a atualização automática do arquivo:
- **Arquivo:** `${ATTACHED_FILE_PATH}`
- **Origem:** PR #${PR_NUMBER}

> *Gerado automaticamente por Ops Publisher*
```

#### `.github/templates/report-timeline.md` (Comentário)

```markdown
✅ **Relatório Gerado com Sucesso!**

Uma nova versão do arquivo `${ATTACHED_FILE_PATH}` está disponível para revisão.
🔗 **Ver Pull Request Derivada:** ${PR_URL}
```

-----

## ⚠️ Notas Importantes

1.  **Dados Sensíveis:** O arquivo em `attached_file_path` é commitado **como está**. Não utilize para segredos ou chaves privadas.
2.  **Cadeia de PRs:** A PR derivada tenta integrar a branch `ops/...` de volta na branch da PR de origem.
3.  **Fetch Depth:** Sempre use `fetch-depth: 0` no checkout para garantir que a action consiga calcular corretamente a árvore do git.

-----

<div align="right"> <sub>Mantido por <a href="https://github.com/Malnati">Malnati</a></sub> </div>
