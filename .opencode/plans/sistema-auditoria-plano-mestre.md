# PLANO MESTRE — Sistema de Auditoria e Conformidade Contabilística e Fiscal (Angola)

> Evolução de **Sistema de Reconciliação de IVA** → **Sistema de Auditoria e Conformidade**
> Documento de análise e roadmap executado **por ordem (fases)**.
> Origem: PROMPT MESTRE (38 secções). Projeto: `sistema-reconciliacao-iva`.

---

## 0. SUMÁRIO EXECUTIVO

O sistema atual é um **protótipo sólido de reconciliação de IVA** com boa base de segurança
(auth, roles, multi-empresa) e um fluxo funcional de upload + análise IA + reconciliação.
O PROMPT MESTRE pede uma plataforma de **15 motores de auditoria** com **Rule Engine,
cobertura mensurada, evidências obrigatórias e auditoria contínua**.

A distância não é de código espalhado, mas de **7 capacidades fundacionais em falta**:

1. Documentos **sem empresa/período/estado** (impossível auditar "Julho/2026")
2. Sem **catálogo de motores nem perfil fiscal** por empresa
3. Sem **matriz de documentos necessários** → sem **cobertura %**
4. Sem **Rule Engine** nem **base legal** versionada
5. Sem modelo de **Ocorrência/Evidência** (alerta com prova)
6. Sem **motores** (só IVA reconciliação + análise IA textual)
7. Sem **relatório/dashboard de auditoria** (cobertura primeiro)

**Estratégia:** 9 fases incrementais, cada uma entregando valor verificável e respeitando a
**Regra de Ouro** (não analisar sem dados; não afirmar sem prova; nunca esconder lacunas).

---

## 1. ESTADO ACTUAL DO SISTEMA (inventário verificado no código)

### 1.1 Backend — 9 modelos (`backend/reconciliacao/models.py`)

| Modelo | O que é | Lacunas face ao PROMPT MESTRE |
|---|---|---|
| `Empresa` | Cadastro completo (NIF, regime IVA, tipo, província, estado) | Falta: repartição fiscal, dados do contabilista, período inicial de análise, **perfil fiscal** |
| `EmpresaUtilizador` | Controlo de acesso por empresa/perfil | ✅ adequado |
| `Contabilidade` | Linhas de balancete por (empresa, ano, mes) | ✅ base para motores contabilísticos |
| `DeclaracaoAGT` | Modelo 7 (IVA liquidado/dedutível/apurar/pagar/recuperar) por período | ✅ base do motor IVA |
| `Reconciliacao` + `ReconciliacaoDetalhe` | Resultado de reconciliação IVA por período, com valores e diferença | Falta ligação a **Ocorrência** com evidência/risco |
| `MapeamentoConta` | Mapeamento conta contabilidade → campo AGT | ✅ insumo do motor IVA |
| `DocumentoUpload` | Upload com `tipo` ∈ {CONTABILIDADE, AGT} + texto extraído | **Crítico:** sem `empresa`, sem `ano/mes`, sem **estado** (Recebido→Validado), só 2 tipos, sem classificação automática |
| `AnaliseIA` | Análise IA (Groq) entre doc contabilidade ↔ doc AGT | Não ligada a empresa/período; IA textual, não regra fiscal |

### 1.2 Motores existentes (2 de 15)

- ✅ **M1 IVA (parcial)** — reconciliação balancete↔Modelo 7 via mapeamento de contas
- ✅ **IA de apoio** — `pdf_service` (pdfplumber + Groq) compara textos e sugere conclusões
- ❌ IRT, Imposto Industrial, Retenções, Seg. Social, Selo, Facturação, Auditoria Contabilística
  (profunda), Reconciliação generalizada, Risco, Obrigações/Prazos, Penalidades/Exposição,
  Anomalias, Histórica, Relatório estruturado

### 1.3 API (`reconciliacao/urls.py`)

- CRUD routers: empresas, contabilidade, declaracoes-agt, reconciliacoes, reconciliacao-detalhes, mapeamento-contas
- FBVs: login (token), dashboard, relatorio/resumo, documentos upload/listar, analises (histórico/eliminar)
- ⚠️ `listar_documentos_view` devolve **todos** os documentos (sem filtro de empresa) — corrigir na Fase 1

### 1.4 Frontend (11 páginas)

Home, Dashboard (recém-reditado: 4 KPIs + 3 gráficos), Empresas (read-only), Documentos,
AnaliseDocumentos, HistoricoAnalises, Reconciliacao, Contabilidade, DeclaracoesAGT,
Relatorios, Perfil. i18n pt/en/zh, MUI, axios com token.

### 1.5 Segurança ✅

Token auth + `IsAuthenticated` + `DjangoModelPermissions` + filtro `empresas_acessiveis(user)`
+ 7 grupos de permissões (least privilege). **Manter como camada transversal em todas as fases.**

### 1.6 Stack

Django 4.2.5 / DRF 3.14 / pdfplumber / groq / React 18 + MUI 5 + Recharts (novo).
Venvs: `.venv` (Django 6.1, local) e `venv` (Django 4.2.5, deploy Render).

---

## 2. ANÁLISE DE LACUNAS — secções do PROMPT MESTRE vs realidade

| § | Requisito | Estado | Fase |
|---|---|---|---|
| 1-2 | Assistente de auditoria com evidências | 🟡 Parcial (IA sem evidência estruturada) | F4 |
| 3 | Cadastro simples com contabilista/repartição/período inicial | 🟡 Falta 3 campos | F1 |
| 4 | Perfil fiscal → motores aplicáveis | 🔴 Não existe | F3 |
| 5 | Auditoria por período (ano/mês) sem misturar | 🔴 Documentos sem período | F1 |
| 6-7 | Upload livre + classificação automática + matriz | 🟡 2 tipos, sem auto-classificação, sem matriz | F1, F2, F3 |
| 8-9 | Auditoria por cobertura; níveis limitada/parcial/ampla/completa | 🔴 Não existe | F3 |
| 10 | Índice de cobertura % real por motor | 🔴 Não existe | F3 |
| 11 | Documentos em falta (o que desbloqueia) | 🔴 Não existe | F3 |
| 12 | Motor IVA completo (cativo, caixa, retido, importações…) | 🟡 Só reconciliação de 4 campos | F5 |
| 13-18 | IRT, II, Retenções, SS, Selo, Facturação | 🔴 Não existem | F5 |
| 19 | Auditoria contabilística profunda | 🟡 Só mapeamento IVA | F5 |
| 20 | Reconciliação generalizada | 🟡 Só IVA | F5 |
| 21 | Análise de risco com 4 severidades explicáveis | 🔴 Não existe | F4 |
| 22 | Obrigações e prazos | 🔴 Não existe | F6 |
| 23 | Exposição potencial (nunca "multa definitiva") | 🔴 Não existe | F6 |
| 24 | Anomalias ("Requer revisão", nunca "fraude") | 🔴 Não existe | F5 |
| 25 | Análise histórica entre períodos | 🔴 Não existe | F5 |
| 26 | Relatório começando pela cobertura | 🔴 Não existe | F8 |
| 27 | Evidência encadeada (alerta→regra→docs→valores→cálculo→dif→lei→recomendação) | 🔴 Não existe | F4 |
| 28 | Auditoria contínua no upload | 🔴 Não existe | F7 |
| 29 | Estado de cada documento (Recebido…Utilizado / erros) | 🔴 Não existe | F1 |
| 30 | Rule Engine centralizado e versionado | 🔴 Não existe | F4 |
| 31 | Base legal verificável por regra | 🔴 Não existe | F4 |
| 32 | IA subordinada (legislação→regra→cálculo→resultado→IA explica) | 🟡 IA decide hoje | F9 |
| 33 | Dashboard de auditoria (cobertura, alertas, em falta) | 🟡 Dashboard genérico | F8 |
| 34-35 | Transparência total de limitações | 🟡 Parcial | F3, F8 |
| 36-38 | Fluxo 20 passos + regra de ouro | 🟡 Base parcial | Todas |

Legenda: 🔴 ausente · 🟡 parcial · ✅ presente

---

## 3. PRINCÍPIOS INVARIANTES (Regra de Ouro traduzida em critérios)

1. **Nenhuma ocorrência sem evidência** — schema `evidencia` obrigatório: `{fontes[], valores[], calculo, diferenca, referencia_legal}`.
2. **Nenhuma regra sem base legal** — `Regra` só activa com `BaseLegal` preenchida e `vigencia` válida.
3. **Nenhuma % de cobertura arbitrária** — calculada: `docs_disponiveis_necessarios / docs_necessarios` ponderada por motor, só com motores **aplicáveis**.
4. **"Completa" = completa dentro do escopo** — disclaimer obrigatório no relatório.
5. **IA nunca inventa regras** — IA só extrai/classifica/explica; o cálculo é do Rule Engine determinístico.
6. **Nunca "fraude" nem "multa definitiva"** — vocabulário permitido: "Requer revisão", "Exposição potencial".
7. **Nunca auditar fora do período** — todo o acesso é (empresa, ano, mes).
8. **Segurança transversal** — todas as queries filtradas por `empresas_acessiveis(user)`.

---

## 4. ARQUITECTURA PROPOSTA

### 4.1 Novos modelos (resumo — detalhe na fase correspondente)

```
Empresa ─┬─ PerfilFiscal (motores aplicáveis)
         ├─ DocumentoUpload* (empresa, ano, mes, tipo 12×, estado 5×, classificação_auto)
         ├─ AuditoriaPeriodo (empresa, ano, mes → estado, cobertura_%, nivel)
         ├─ Ocorrencia (motor, regra, severidade, valor, evidencia JSON, recomendacao, estado)
         ├─ Obrigacao (tipo, periodo, prazo, estado)
         └─ ExecucaoRegra (regra, periodo, corrigida_em, registos)

Regra (codigo, motor, imposto, vigencia, condicao, formula, severidade, versao)
  └─ BaseLegal (legislacao, artigo, vigencia, fonte_oficial)

Motor (codigo 1..15, nome, documentos_requeridos[])   ← catálogo estático/seed
DocumentoRequerido (motor, tipo_documento, obrigatorio_paraperfil) ← matriz §7
```

### 4.2 Rule Engine (centralizado)

- `regra_engine.py`: carrega regras activas do período → avalia `condicao` (expressão segura
  com whitelist de variáveis) → executa `formula` → grava `Ocorrencia` (ou "sem ocorrência").
- Cada execução regista: entradas, fórmula, resultado, versão da regra → **audit trail**.
- Regras vivem em BD (versionadas), **não** em `if` dispersos no código.

### 4.3 Contrato de Motor (plugin uniforme)

```python
class Motor:
    codigo: str                 # 'IVA', 'IRT', ...
    requer_documentos: [Tipo]   # para a matriz de cobertura
    aplicavel_a(perfil) -> bool
    executar(periodo) -> [Ocorrencia]
```

### 4.4 Níveis de auditoria (§9) — cálculo

| Nível | Critério |
|---|---|
| LIMITADA | cobertura < 40% |
| PARCIAL | 40% ≤ cobertura < 75% |
| AMPLA | 75% ≤ cobertura < 100% |
| COMPLETA | 100% dos documentos dos motores **seleccionados** e válidos (+ disclaimer) |

---

## 5. PLANO POR ORDEM — 9 FASES

### FASE 1 — Fundação: Documentos por Período com Estados  *(§3, §5, §6, §29)*

**Objectivo:** todo o resto depende de saber *de quem*, *de quando* e *em que estado* é cada documento.

- **Modelos:** `DocumentoUpload` += `empresa (FK)`, `ano`, `mes`, `estado`
  (`RECEBIDO→PROCESSANDO→PROCESSADO→VALIDADO→UTILIZADO` | `ERRO/ILEGIVEL/PERIODO_ERRADO/TIPO_DESCONHECIDO`),
  `tipo` estendido a 12 (`BALANCETE, MODELO7, FACTURA_VENDA, FACTURA_COMPRA, FOLHA_SALARIAL,
  EXTRACTO_BANCARIO, RETENCOES, COMPROVATIVOS, IMPOSTO_INDUSTRIAL, DECLARACAO_IRT, FOLHA_SS, OUTRO`
  — manter `CONTABILIDADE/AGT` como compat/aliases na migração).
- **`Empresa`** += `reparticao_fiscal`, `contabilista_nome`, `contabilista_contacto`, `periodo_inicial_analise`.
- **Views:** upload exige `empresa_id, ano, mes`; **corrigir** `listar_documentos_view` para filtrar
  por empresa acessível + período (query params).
- **Frontend:** selector empresa+período antes do upload; lista com estado por documento; tags de estado.
- **Migração:** data-safe (`RunPython` copia doc sem empresa para empresa "default"/superuser).
- **Aceite:** não é possível listar/upload fora do (empresa, período) acessível; estados visíveis; testes de API.

### FASE 2 — Extração e Classificação Automática  *(§6, §7)*

**Objectivo:** aceitar PDF, Excel, CSV, XML e classificar o tipo automaticamente.

- Extractors: `pdfplumber` (já há), **openpyxl/pandas** (xlsx), `csv` nativo, `xml.etree` (XML AGT).
- Classificador determinístico primeiro (nome/folha/cabeçalhos/palavras-chave: "Modelo 7", "Balancete",
  "NIF", colunas de factura); **IA só como desempate** (F9), nunca decisive.
- Upload: gravar `tipo_sugerido` vs `tipo_confirmado` (utilizador pode corrigir → alimenta melhoramento).
- **Aceite:** um xlsx de balancete e um CSV de facturas são extraídos e classificados sem intervenção manual.

### FASE 3 — Perfil Fiscal, Matriz e Cobertura  *(§4, §7, §8, §9, §10, §11)*

**Objectivo:** saber **o que se pode e o que não se pode auditar**, e dizer-lo sempre.

- Seed dos **15 motores** (catálogo) + `DocumentoRequerido` (matriz §7) + `PerfilFiscal`
  (motores aplicáveis por regime/actividade — default: todos os aplicáveis ao regime).
- `cobertura_service.calcular(empresa, ano, mes)` →
  `% geral`, `% por motor`, `nivel`, `documentos_em_falta[]` (cada um: *"desbloqueia motor X; período Y"*).
- **Endpoints:** `GET /api/auditoria/{empresa}/{ano}/{mes}/cobertura/`.
- **Frontend:** painel de cobertura (barra % + grelha por motor 🟢🟡⚪ + docs em falta com checkbox),
  sempre no topo da página de auditoria.
- **Aceite:** subir só Balancete+Modelo 7 → cobertura ≈ % esperado, nível "LIMITADA", facturação
  listada como "não analisada — em falta"; nunca "completa" com lacunas.

### FASE 4 — Rule Engine + Base Legal + Ocorrência  *(§21, §27, §30, §31, §35)*

**Objectivo:** motor de regras central e alertas com prova encadeada.

- Modelos `BaseLegal`, `Regra` (código único tipo `IVA-002`, motor, imposto, vigência, severidade,
  `condicao`/`formula` — expressões com variáveis whitelisted, versão, activa, recomendação) e
  `Ocorrencia` (empresa, período, regra, severidade `INFORMATIVO|BAIXO|MEDIO|ALTO`, valor_envolvido,
  `evidencia JSON` obrigatório, `recomendacao`, estado `ABERTA|REVISADA|IGNORADA|RESOLVIDA`).
- `regras/seed_inicial.py`: **conjunto inicial conservador de regras IVA** (ex.: balancete sem
  correspondência no Modelo 7 — IVA-002) com `BaseLegal` **marcada como "requer validação jurídica"**
  até validação oficial (Regra de Ouro: não publicar regra sem fundamento).
- **Endpoints:** `GET /ocorrencias/?empresa&ano&mes&severidade`, `GET /ocorrencias/{id}` (evidência completa),
  `PATCH /ocorrencias/{id}` (estado/revisão humana).
- **Frontend:** página **Alertas** com filtros + drill-down mostrando a cadeia
  *Regra → Documentos → Valores → Cálculo → Diferença → Referência legal → Recomendação*.
- **Aceite:** nenhum alerta sem evidência completa (teste automático que valida o schema).

### FASE 5 — Motores de Auditoria (por ordem de dados disponíveis)  *(§12-20, §24, §25)*

**Ordem de implementação** (valor decrescente de dados já disponíveis):

| # | Motor | Dados de que precisa | Estado dos dados |
|---|---|---|---|
| 5.1 | **M1 IVA completo** — evoluir reconciliação actual (liquidado, dedutível, a pagar/recuperar, taxas, operações tributáveis/isentas) | Balancete + Modelo 7 | ✅ existem |
| 5.2 | **M8 Contabilidade** — saldos anormais/negativos, transitórios, duplicados, fora do período, razão de impostos | Balancete | ✅ existe |
| 5.3 | **M9 Reconciliação generalizada** — factura→contab→balancete→declaração→pagamento (onde houver) | parcial | 🟡 |
| 5.4 | **M14 Histórica** — variações mês-a-mês (receitas, IVA, custos) | Balancete × N períodos | ✅ (múltiplos meses) |
| 5.5 | **M13 Anomalias** — variações bruscas, concentração de fornecedores (rotulagem: "Requer revisão") | Balancete + facturas | 🟡 |
| 5.6 | **M7 Facturação** + **M4 Retenções** — duplicações, saltos de numeração, NIF, taxas | Facturas/ret. | 🔴 depende F1/F2 (docs novos) |
| 5.7 | **M2 IRT**, **M5 Seg. Social** | Folha salarial | 🔴 docs novos |
| 5.8 | **M3 Imposto Industrial**, **M6 Selo** | Balancete + ajustamentos + contratos | 🟡/🔴 |
| 5.9 | **M10 Risco** (agregador) — combina ocorrências → nível de risco por área | Ocorrências | depende F4 |

Cada motor: classe + seeds de regras + testes com fixtures + cartão de motor no frontend
(estado: ✅ executado / 🟡 parcial / ⚪ por documentação).

### FASE 6 — Obrigações/Prazos e Exposição Potencial  *(§22, §23)*

- `Obrigacao`: tipo (Modelo 7 mensal, IRT, SS, II anual…), empresa, período, prazo, estado
  (`CUMPRIDA | PROXIMO | ATRASADO | NAO_APLICAVEL`), prova de pagamento quando existir.
- `ExporicaoPotencial`: aggregado por ocorrência com `valor_envolvido`, `base_legal`,
  `consequencia_potencial`, `estimativa` (quando calculável) e `limitacoes_calculo` (texto obrigatório).
- Frontend: calendário/estado de obrigações + cartão "Exposição potencial" **com disclaimer**.
- Aceite: nunca aparece "multa definitiva"; toda exposição mostra limitações.

### FASE 7 — Auditoria Contínua  *(§28)*

- No `upload_documento_view`: classificar → validar → (se período aberto) correr regras
  aplicáveis daquele motor → criar ocorrências → notificar (toast/in-app) imediatamente.
- Re-execução idempotente por (regra, documento, período) — sem duplicar ocorrências.
- Aceite: carregar uma factura com NIF inválido → alerta antes do fecho do período.

### FASE 8 — Dashboard de Auditoria + Relatório  *(§26, §33, §34)*

- **Relatório por período** (estrutura fixa): cobertura primeiro (%, docs 8/11, motores
  executados/parciais/não executados) → contagem de severidades → reconciliações →
  divergências → anomalias → obrigações → exposição → docs em falta → recomendações →
  **disclaimer** ("completa = dentro do escopo").
- **Dashboard** (substituir/evoluir o actual): cabeçalho `AUDITORIA — MÊS/ANO`, barra de cobertura,
  grelha de motores por estado, contadores 🔴🟠🟡🟢, docs em falta; manter KPIs globais actuais.
- Export: **HTML print** (fase imediata) + PDF (opção `weasyprint`/`wkhtmltopdf` — decidir).
- Frontend: página Relatórios reescrita + selector de período em todas as vistas de auditoria.

### FASE 9 — IA Subordinada  *(§32)*

- Reordenar a IA para o papel correcto: **extrair** (campos de facturas/folhas), **classificar**
  (desempate), **explicar/resumir** ocorrências e relatório.
- Guardrail explícito: prompts never inventam regras; a resposta da IA só referência regras
  existentes no Rule Engine (`codigo_regra` deve existir em BD — validação).
- Já existe Groq + `pdf_service` → estender, não substituir.

---

## 6. FLUXO MVP (quando as Fases 1-4 + 5.1-5.2 estiverem prontas)

```
Cadastrar empresa (F1) → Perfil fiscal (F3) → Escolher período (F1)
→ Carregar Balancete + Modelo 7 (F1/F2) → Classificação automática (F2)
→ Validar docs (F1) → Cobertura calculada + docs em falta (F3)
→ Executar motores IVA + Contabilidade (F5) → Ocorrências com evidência (F4)
→ Dashboard de auditoria + Relatório (F8) → Recomendações → Correcção → Nova auditoria
```

---

## 7. RISCOS E CAUTELAS

| Risco | Mitigação |
|---|---|
| **Validade jurídica das regras** (Lei Geral Tributária, Código do IVA, etc.) | Seeds marcadas `pendente_validacao_juridica`; só regras validadas ficam activas; documento de referências oficiais |
| Extracção ruim em PDFs digitados (erros de OCR) | Estado `ILEGIVEL` + reprocessamento; nunca calcular sobre texto < confiança mínima |
| Escopo gigante (15 motores) | Fases entregam valor desde a F3 (cobertura) mesmo com 2 motores |
| Consistência de migrações (`.venv` Django 6.1 vs `venv` Django 4.2.5) | Correr `makemigrations/migrate/check` em **ambas** as venvs sempre |
| IA alucinar conclusões fiscais | Guardrail F9: validação código-regra contra BD; IA nunca escreve Ocorrência |
| Performance com muitos documentos | Processamento assíncrono futuro (celery/threads) — só se necessário |

---

## 8. DECISÕES A CONFIRMAR COM O UTILIZADOR

1. **Ordem dos motores** — confirma a prioridade 5.1→5.9 (IVA→Contabilidade→Histórica→…)? Algum motor urgente para o teu caso real?
2. ~~**Validação jurídica**~~ ✅ **DECIDIDO (2026-09-30): o utilizador valida as regras.** Protocolo da Fase 4: todas as regras nascem com estado `POR_VALIDAR` (informativas, sem alertas oficiais); o utilizador valida uma a uma e só então passam a `VALIDADA` (podem gerar alertas de exposição). Cada regra guarda quem/ quando validou.
3. **Relatório em PDF** — HTML para imprimir chega? ou PDF gerado no servidor?
4. **Período aberto/fechado** — bloquear upload de períodos antigos fechados, ou permitir com aviso?
5. **Dados de teste** — criar fixtures de exemplo (balancete+Modelo 7 de Julho/2026) para demonstrar cobertura e alertas desde já?

---

## 9. CRITÉRIOS GERAIS DE ACEITAÇÃO (todas as fases)

- [ ] `manage.py check` + `migrate` + testes OK em `.venv` **e** `venv`
- [ ] `npm run build` sem erros; fluxo manual por fase verificado no browser
- [ ] Nenhum endpoint expõe dados de empresa não acessível (teste 404/403)
- [ ] i18n pt/en/zh completo para cada novo texto
- [ ] Nenhuma ocorrência/regra/relatório sem evidência/base legal/disclaimer conforme o caso
