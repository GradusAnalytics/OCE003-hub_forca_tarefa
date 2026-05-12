# Hub Multi-FT — PMI OceanPact × CBO

Ferramenta HTML single-file para gestão e acompanhamento da integração pós-fusão OceanPact × CBO (Projeto América). Centraliza os dados de todas as Forças-Tarefa do PMI em uma interface navegável, conectada a um backend S3/AWS para versionamento de snapshots.

---

## Visão Geral

O Hub é um arquivo `.html` autossuficiente que carrega dados de uma **snapshot JSON** hospedada no S3. Não requer instalação, servidor local ou dependências — abre direto no browser.

Ao abrir, o Hub busca automaticamente a snapshot ativa no bucket S3 e renderiza o painel consolidado de todas as FTs.

---

## Funcionalidades

### Tela inicial (Hub)
- Painel executivo com progresso geral, progresso priorizado e métricas consolidadas
- Cards por Força-Tarefa com avanço, contagem de processos, questões e processos com contraste
- Tabela comparativa entre FTs com colunas de progresso, priorizados, contraste de processos e questões

### Navegação entre FTs
Cada card de FT abre a visão detalhada com 6 abas:

| Aba | Conteúdo |
|-----|----------|
| **Resumo Geral** | KPIs da FT, progresso por status, acompanhamento |
| **Processos & Questões** | Inventário de processos e questões-chave com filtros |
| **Status dos Processos** | Progresso por processo com status e score |
| **Contraste de Práticas** | Comparativo OceanPact × CBO × NewCo por dimensão |
| **Painel de Decisões** | Registro e acompanhamento de decisões por processo |
| **RACI To-be** | Matriz de responsabilidades da NewCo |

### Seletor de Snapshots
- Select no header lista todas as versões disponíveis no S3
- Botão **⬇ Carregar** troca os dados do dashboard sem alterar a snapshot ativa no S3
- Indicador visual da versão ativa (`✓ ativa`)

### Exportação
- **Exportar Base Completa** — CSV com todos os dados da visão atual
- **Exportar Decisões CSV** — CSV com o estado atual das decisões
- **Backup HTML da FT** — gera arquivo HTML offline da FT aberta, com todas as decisões embutidas

---

## Arquitetura

```
Browser (Hub HTML)
    │
    ├── GET /snapshots          → lista versões disponíveis
    ├── GET /snapshots/data     → carrega JSON da snapshot ativa (ou selecionada)
    └── POST /snapshots/apply   → ativa snapshot (via Gerenciador)

API Gateway (HTTP API)
    └── Lambda pmi-list-snapshots / pmi-get-snapshot / pmi-apply-snapshot
            └── S3: gradus-pmi-snapshots
                    ├── snapshots/snapshot_v31.json
                    ├── snapshots/snapshot_v30.json
                    └── active.txt  ← key da snapshot ativa
```

### Variáveis no HTML

```javascript
const PMI_SNAPSHOTS_API = "https://hgelha26b4.execute-api.us-east-1.amazonaws.com";
const PMI_API_TOKEN     = "pmi-token-2026";
```

---

## Estrutura do JSON de Snapshot

```json
{
  "fts": [
    {
      "nome": "Comercial",
      "processos": [ ... ],
      "questoes": [ ... ],
      "acompanhamento": { ... },
      "progresso": 0.325,
      "progresso_priorizados": 0.857
    }
  ],
  "meta": {
    "titulo": "PMI OceanPact × CBO",
    "version_label": "snapshot_v31",
    "updated_at": "2026-05-12T15:17:00Z",
    "total_fts": 14,
    "total_processos": 416,
    "total_questoes": 3121,
    "progresso_geral": 0.28,
    "render_contract": "hub-multi-ft-v53"
  }
}
```

### Campos obrigatórios por processo

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `nome` | string | Nome canônico do processo |
| `ft_nome` | string | Nome da Força-Tarefa |
| `status` | string | `Não Iniciado` / `Iniciado` / `Finalizado` / `Validado` |
| `score_max` | int | Score para ordenação e priorização |
| `prioritario` | bool | Flag de processo prioritário |
| `ppt_rows` | array | Linhas do contraste de práticas |
| `linhas` | array | Etapas detalhadas do processo |
| `questoes` | array | Questões-chave vinculadas |
| `decision_items` | array | Itens de decisão |
| `processo_stable_id` | string | ID estável entre versões |

---

## Forças-Tarefa

O Hub suporta as seguintes FTs do projeto:

- Comercial
- Obras
- Manutenção
- Operação
- Pessoal
- Suprimentos / Logística
- Tecnologia
- SMS
- Finanças / Controladoria
- Jurídico
- Compliance
- Sustentabilidade
- Facilities
- CEOP / CDPA

---

## Gerenciamento de Versões

O versionamento de snapshots é feito pela ferramenta **pmi-snapshot-manager.html**, que permite:

- Listar todas as snapshots no bucket S3
- Fazer upload de novas versões (via presigned URL — sem limite de tamanho)
- Tornar qualquer versão a ativa
- Excluir versões antigas

Para gerar uma nova snapshot a partir de inputs de PPT/Excel, use a skill **pmi-snapshot-generator** instalada no Claude.

---

## Histórico de Versões

| Versão | Data | Principais mudanças |
|--------|------|---------------------|
| v53 | Mai/2026 | Seletor de snapshots no header, botão Hub corrigido, bugs de renderização de script corrigidos |
| v51 | Mai/2026 | Integração com API S3, carregamento dinâmico de snapshot ativa |
| v46+ | Abr/2026 | Painel de decisões, RACI To-be, backup HTML offline |

---

## Desenvolvimento

O Hub é mantido pela **Gradus Consultoria** no contexto do projeto de integração Projeto América (OceanPact × CBO). Alterações no código devem seguir o padrão de versionamento (`v53 → v54`) com backup da versão funcional antes de qualquer modificação.
