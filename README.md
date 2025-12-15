<!-- README.md -->
# ⚙️ Ops Publisher

[![GitHub Release](https://img.shields.io/github/v/release/Malnati/ops-publisher?style=for-the-badge&color=blue)](https://github.com/Malnati/ops-publisher/releases)

**A engine de publicação para suas ferramentas de governança.**

Esta Action abstrai a complexidade de **GitOps** para relatórios de auditoria. Ela recebe um arquivo Markdown gerado por qualquer ferramenta, aplica validações de idempotência (para evitar re-trabalho), gerencia branches dedicadas e cria/atualiza Pull Requests automaticamente.

## 🚀 Funcionalidades Core

1.  **Assinatura de Conteúdo:** Calcula um hash do código-fonte (`ts, js, py...`). Se o código não mudou, o relatório não é republicado.
2.  **Proteção Anti-Loop:** Detecta automaticamente se o commit foi feito pelo bot ou se está rodando na branch de relatório, interrompendo o ciclo infinito.
3.  **Gestão de PRs:** Cria branches órfãs ou derivadas (`audit/report/...`) e mantém a PR de relatório sempre atualizada com `git push -f`.

## 📦 Inputs

| Input | Obrigatório | Descrição |
| :--- | :---: | :--- |
| `token` | Sim | Token com `contents:write` e `pull-requests:write`. |
| `report_file` | Sim | Caminho do arquivo Markdown a ser publicado. |
| `scan_extensions` | Não | Extensões consideradas para a assinatura de código. |
| `report_branch_prefix` | Não | Prefixo da branch (Padrão: `audit/report`). |

## 🛠️ Como usar (Criando sua própria ferramenta)

Exemplo: Criando um scanner de TODOs que usa esta engine para publicar.

```yaml
steps:
  - uses: actions/checkout@v4
    with: { fetch-depth: 0 }

  # 1. Sua Ferramenta gera o relatório
  - name: Generate TODO Report
    run: |
      grep -r "TODO" . > todo_report.md

  # 2. A Engine publica (se necessário)
  - name: Publish
    id: ops
    uses: Malnati/ops-publisher@v1
    with:
      token: ${{ secrets.GITHUB_TOKEN }}
      report_file: "todo_report.md"
      pr_title: "📝 TODOs Audit"
      
  # 3. Você usa o output
  - name: Notify
    if: steps.ops.outputs.status == 'PUBLISHED'
    run: echo "Relatório novo em: ${{ steps.ops.outputs.pr_url }}"
