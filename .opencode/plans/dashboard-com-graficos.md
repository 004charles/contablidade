# Plano: Dashboard profissional com gráficos

**Aprovado pelo utilizador (opção "Plano recomendado").**

## Fase 0 — REPARAÇÃO URGENTE do frontend (executar primeiro!)

O utilizador correu `npm audit fix --force`, que danificou o `frontend/package.json`:
- `react-scripts`: `5.0.1` → **`^0.0.0`** (quebrado; binário ausente — `npm start`/`npm run build` não funcionam)
- `react-router-dom`: `^6.16.0` → `^7.18.4` (mudança major)
- `node_modules`: 1217 pacotes removidos (só 140 restam)

Passos:
1. Editar `frontend/package.json`: repor `react-scripts` = `5.0.1` e `react-router-dom` = `^6.16.0` (valores originais confirmados nesta sessão); manter `recharts` e o resto
2. `cd frontend && rm -rf node_modules package-lock.json && npm install`
3. `npm run build` → tem de compilar sem erros
4. Utilizador reinicia `npm start` e confirma `http://localhost:3000` a funcionar
5. **Só depois** executar as Fases 1–4 abaixo (Dashboard)
Projeto: `sistema-reconciliacao-iva` (`/home/sckj-muquissi/Documentos/DADOS CONTABLIDADE/sistema-reconciliacao-iva`)

## Problema
- `frontend/src/pages/Dashboard.jsx` não tem gráficos (só 3 KPIs + lista + atalhos)
- Usa `fetch` **sem token** → desde que a API passou a exigir `IsAuthenticated`, o Dashboard recebe 401 e mostra sempre zeros
- Sem biblioteca de gráficos instalada

## Dados reais (verificado)
7 empresas · 82 documentos · 13 análises (todas Set/2026) · 0 reconciliações · 0 declarações
→ Gráficos **obrigados** a tratar estados vazios com elegância.

## Endpoints usados (backend **sem alterações**)
- `GET /api/empresas/` (autenticado, filtrado por acesso) → KPI Empresas + por estado
- `GET /api/reconciliacoes/` → donut por estado; barras top divergências por empresa
- `GET /api/analises/historico/` → KPI Análises + barras por mês + atividade recente
- `GET /api/relatorios/resumo/` → top 5 empresas com divergências

## Passos de implementação

### 1. Dependência
```bash
cd frontend && npm install recharts
```

### 2. Reescrever `frontend/src/pages/Dashboard.jsx`
- **Correção base**: substituir `fetch` por `api` do `frontend/src/services/api.js` (axios + interceptor de token). Endpoints: `/empresas/`, `/reconciliacoes/`, `/analises/historico/`, `/relatorios/resumo/`. `Promise.all` + try/catch; em caso de erro → toast via `react-toastify` (padrão do projeto) + zeros.
- **Layout (3 secções, grid MUI, paleta do projeto `#003D99` / `#0F172A` / `#64748B`):**
  - **Secção A — 4 KPI cards**: Empresas (total + ativas), Análises (total), Reconciliações (total), Divergências (nº + taxa %). Ícones MUI, cantos 12px, borda `#E2E8F0`.
  - **Secção B — 2 gráficos lado a lado (Recharts):**
    - `PieChart` (donut, innerRadius) — Reconciliações por estado: Conciliado `#10B981`, Divergência `#F59E0B`, Pendente `#003D99`. Sem dados → centro com ícone + `dashboard.charts.noData`.
    - `BarChart` — Análises por mês (últimos 6 meses, meses sem dados = 0), cor `#003D99`, `ResponsiveContainer`.
  - **Secção C:** `BarChart` horizontal — Top 5 empresas com divergências (valores de `divergencias` do `/relatorios/resumo/`, ordenado desc) + mantém card de **Atividade recente** (5 análises) e os 2 atalhos (Nova Análise / Relatório Completo).
- **Estados**: `loading` → `CircularProgress` (padrão atual); vazio → mensagem dentro do gráfico.

### 3. i18n — `frontend/src/i18n/locales/{pt,en,zh}.json`
Novas chaves sob `dashboard.charts.*`: `reconciliationByStatus`, `analysesPerMonth`, `topDivergentCompanies`, `noData`, `statusConciliada`, `statusDivergencia`, `statusPendente`. Reutilizar chaves já existentes (`totalCompanies`, `analysesPerformed`, `recentAnalyses`, `newAnalysis`, `fullReport`, …).

### 4. Verificação
1. `cd frontend && npm run build` → compila sem erros
2. Backend ligado (`./.venv/bin/python manage.py runserver`) + frontend (`npm start`)
3. Login no `http://localhost:3000` → `/dashboard`:
   - Sem 401 (ver consola)
   - 4 KPIs com números reais (7 empresas, 13 análises, 0 reconc., 0 diverg.)
   - Donut mostra estado vazio "Sem dados" (0 reconciliações)
   - Barras de análises/mês mostra 13 em Set/2026
   - Top divergências mostra estado vazio
   - Atividade recente lista as análises
4. Testar também como utilizador não-superuser (ex.: `carlos`) → vê só dados das empresas autorizadas

## Ficheiros afetados
| Ficheiro | Mudança |
|---|---|
| `frontend/package.json` | + `recharts` |
| `frontend/src/pages/Dashboard.jsx` | reescrita (auth + gráficos) |
| `frontend/src/i18n/locales/pt.json` | novas chaves `dashboard.charts.*` |
| `frontend/src/i18n/locales/en.json` | idem |
| `frontend/src/i18n/locales/zh.json` | idem |
| **Backend** | **nenhuma alteração** |
