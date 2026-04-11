# Prompt — Minha Jornada 🌸
> App React de acompanhamento de emagrecimento — para replicar no Claude Cowork

---

## Dados iniciais
- Peso inicial: 91,5kg (21/02/2026), meta: 65kg, altura: 1,69m
- Meta de hidratação: 3.150ml/dia
- Storage key: `jornada-emagrecimento-2026`
- Ao carregar, tentar recuperar dados de todas estas chaves do localStorage: `jornada-emagrecimento-2026`, `jornada-emagrecimento`, `jornada-v3`, `jornada-v2`, `jornada-emagrecimento-2025` — ficando com o conjunto que tiver mais registros

---

## Estrutura geral
- App mobile-first, fundo gradiente rose/pink
- Header fixo com título, nome da aba atual, barra de progresso e botão **⚡ Check-in rápido**
- Navegação inferior com 8 abas: 📊 Visão Geral · 📅 Semanas · ⚖️ Peso · 🏋️ Treinos · 💧 Hidratação · 💊 Medicação · 🌿 Suplementos · 🧠 Análise IA
- Todos os campos de data usam `<input type="date">` nativo (não digitação manual)
- Dados salvos automaticamente no localStorage a cada mudança

---

## Abas e funcionalidades

### 📊 Visão Geral
- Cards: peso atual, kg perdidos, IMC atual, próxima dose
- Card de projeção verde: data estimada para 65kg com ritmo atual (kg/semana)
- Card IMC: inicial / atual / meta com classificação colorida
- Card água hoje com barra de progresso
- Gráfico de evolução do peso (linha vermelha = real, linha roxa tracejada = tendência linear projetada até a data de meta)
- Card resumo de treinos (sessões + kcal estimadas)
- Card suplementos ativos

### 📅 Semanas
- Gráfico de barras com variação semanal (kg)
- Cards por semana: variação, peso início→fim, treinos, média hidratação, IMC final
- Verde se perdeu, vermelho se ganhou, cinza se neutro

### ⚖️ Peso
- Form de registro: date picker + campo kg
- Calendário mensal navegável: dias com registro em verde, dias sem registro em cinza neutro, dia de hoje com borda rosa
- Registros agrupados por mês em accordions fechados por padrão
- Dentro de cada mês: mostrar todos os dias a partir do primeiro registro daquele mês; dias passados sem dado mostram ❌ com opções de **+ Add** (pré-preenche o form) e marcação intencional (😴 Descanso / ✈️ Viagem / 📝 Esqueci / 🌿 Folga); dias futuros aparecem em cinza sem ❌
- Cada registro tem botão ✏️ editar e 🗑️ excluir (edição inline)
- Header do accordion mostra quantos dias sem registro

### 🏋️ Treinos
- Botões rápidos de atividade: Musculação · Pilates · Natação
- Select completo com todas as modalidades; se "Outro" selecionado, abre campo de texto livre
- Botões de duração: 40 / 50 / 60min + "outro" (abre campo numérico)
- Registros por mês em accordions fechados; primeira linha do mês mostra resumo de modalidades (ex: "Pilates: 4x · Natação: 3x")
- Todos os dias do mês exibidos; dias com treino em caixa verde numa linha (🏋️ + tipo + duração + kcal); dias sem treino com ❌ e ações de marcação
- Edição e exclusão inline

### 💧 Hidratação
- Date picker + campo ml
- Botões para definir valor: 500ml / 1L / 1,5L
- Botões de adição rápida: +500ml / +1L / +1,5L (adicionam ao total do dia)
- Feedback animado ao registrar: "+Xml registrado! 💧"
- Barra de progresso do dia atual
- Registros por mês em accordions fechados; cada entrada em linha única com 💧 + ml + mini barra de progresso colorida (laranja <70%, azul 70–99%, verde ≥100%) + %
- Edição e exclusão inline

### 💊 Medicação
- Date picker, campo medicamento, botões rápidos de dose: 2,5mg / 5mg / 7,5mg / 10mg / 12,5mg / 15mg + campo livre
- Card de próxima dose (7 dias após a última)
- Registros por mês em accordions fechados
- Edição e exclusão inline

### 🌿 Suplementos
- Botões atalho pré-formatados: Whey Protein (1 scoop, pós-treino) e Creatina (5g, diário) — ficam destacados quando selecionados
- Date picker, nome, dose, horário
- Registros por mês em accordions fechados
- Migração automática do formato antigo `{whey: [...], creatina: [...]}` para array flat

### 🧠 Análise IA
- Botão que chama `https://api.anthropic.com/v1/messages` com header `anthropic-dangerous-direct-browser-access: true`
- Modelo: `claude-sonnet-4-20250514`, max_tokens: 1000
- Analisa últimos 7 registros de cada categoria e gera feedback semanal motivador em português
- Histórico de análises salvas por data

---

## Check-in rápido (modal bottom sheet)
- Peso (campo numérico)
- Treino: checkbox opcional; se marcado, mostra botões rápidos Musculação/Pilates/Natação + select + botões 40/50/60min + "outro"
- Água: campo + botões 500ml / 1L / 1,5L / 2L / 3L / 3,5L / 4L / 4,5L
- Suplementos: botões toggle (Whey Protein, Creatina, + outros cadastrados) — ficam verdes ao selecionar
- Botão salvar registra tudo com a data de hoje

---

## Componentes principais
- **`MesAccordion`**: fecha por padrão, mostra nome do mês + contagem, aceita `resumo` (primeira linha interna), `getItemDia` (para renderizar todos os dias do mês), `onAddDia` (callback para pré-preencher form), `marcadoresField` (objeto com marcações intencionais por data)
- **`DateInput`**: wrapper de `<input type="date">` convertendo entre `DD/MM/YYYY` e `YYYY-MM-DD`
- **`MARCADORES`**: objeto com emoji + label para descanso/viagem/esqueci/folga — salvo em localStorage separado (`_marc`)
- Marcações intencionais persistidas em chave separada no localStorage

---

## Atividades e cálculo de calorias

| Atividade | kcal/min |
|---|---|
| Pilates | 4 |
| Natação | 8 |
| Musculação | 6 |
| Cardio | 9 |
| Caminhada | 4,5 |
| Corrida | 11 |
| Yoga | 3 |
| Hidroginástica | 5,5 |
| Dança | 6 |
| Bicicleta | 7 |
| Funcional | 8 |
| Outro | 5 |

---

## Dados históricos pré-carregados (DEFAULT_DATA)

### Registros de peso

| Data | Peso (kg) |
|---|---|
| 21/02/2026 | 91,5 |
| 23/02/2026 | 92,3 |
| 24/02/2026 | 91,9 |
| 25/02/2026 | 91,5 |
| 26/02/2026 | 91,0 |
| 27/02/2026 | 90,8 |
| 28/02/2026 | 90,5 |
| 01/03/2026 | 90,5 |
| 02/03/2026 | 90,2 |
| 03/03/2026 | 89,5 |
| 04/03/2026 | 89,2 |
| 05/03/2026 | 89,4 |
| 06/03/2026 | 88,8 |
| 07/03/2026 | 88,8 |
| 15/03/2026 | 87,5 |
| 17/03/2026 | 86,2 |
| 19/03/2026 | 85,2 |
| 21/03/2026 | 86,6 |
| 22/03/2026 | 86,1 |
| 23/03/2026 | 85,2 |
| 28/03/2026 | 85,4 |
| 29/03/2026 | 85,3 |
| 31/03/2026 | 86,2 |
| 01/04/2026 | 84,1 |
| 02/04/2026 | 85,6 |

### Treinos

| Data | Tipo | Duração |
|---|---|---|
| 26/02/2026 | Pilates | 60min |
| 27/02/2026 | Pilates | 60min |
| 28/02/2026 | Natação | 60min |
| 02/03/2026 | Hidroginástica | 35min |
| 03/03/2026 | Pilates | 40min |
| 04/03/2026 | Natação | 40min |
| 05/03/2026 | Natação | 45min |
| 06/03/2026 | Pilates | 40min |

### Hidratação

| Data | ml |
|---|---|
| 26/02/2026 | 3450 |
| 27/02/2026 | 4370 |
| 28/02/2026 | 3450 |
| 01/03/2026 | 4150 |
| 02/03/2026 | 4500 |
| 03/03/2026 | 3900 |
| 04/03/2026 | 3220 |

### Medicação

| Data | Medicamento | Dose | Nota |
|---|---|---|---|
| 22/02/2026 | Tirzepatida | 2,5mg | 1ª dose |
| 01/03/2026 | Tirzepatida | 2,5mg | 2ª dose |

### Suplementos

| Data | Nome | Dose | Horário |
|---|---|---|---|
| 01/03/2026 | Whey Protein | 1 scoop (~25g) | pós-treino |
| 01/03/2026 | Creatina | 5g | diário |

---

## Stack
- **React** com hooks (useState, useMemo, useEffect)
- **Tailwind CSS** para estilo
- **recharts** para gráficos (LineChart, BarChart, Legend)
- **lucide-react** para ícones (Scale, Flame, Pill, Trash2, Brain, RefreshCw, Pencil, X, ChevronLeft, ChevronRight, Zap, ChevronDown, ChevronUp)
