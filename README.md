# Google Sheets — Automação de Planilhas Escolares

Scripts em Google Apps Script para criação, atualização e controle de acesso de planilhas de acompanhamento de alunos no Google Sheets, integrados ao Google Drive.

---

## Visão Geral

Este projeto automatiza o gerenciamento de planilhas de avaliação escolar. A partir de uma planilha central de configuração, os scripts criam e mantêm planilhas individuais por turma — com dados de alunos, níveis de leitura, escrita e controle de acesso mensal.

---

## Estrutura do Projeto

```
Planilha de Configuração (Google Sheets)
│
└── Abas por Turma (ex: 1A, 2B, 3C...)
    ├── Escola, Sala, Professor, Alunos
    └── Nível de leitura, Etnia, Origem

Pasta Base (Google Drive)
└── 1° ANO /
│   ├── 1° ANO A.xlsx
│   └── 1° ANO B.xlsx
└── 2° ANO /
    └── ...
```

---

## Scripts

### 1. `Create` — Criação de Planilhas

> **Arquivo:** `Create.gs`

Lê cada aba da planilha de configuração e cria uma planilha de turma no Google Drive a partir de um modelo pré-definido, caso ainda não exista.

**O que faz:**
- Seleciona o modelo correto conforme o ano letivo (1°, 2° ou 3° ao 5°)
- Cria a pasta do ano no Drive se necessário
- Pula turmas sem alunos cadastrados
- Preenche cabeçalhos (escola, sala, professor, bimestre)
- Popula as abas **Leitura** e **Níveis de Escrita** com os dados dos alunos
- Instala o trigger de `onEdit` na planilha criada
- Aplica proteção de colunas via `LockArquivo`

**Uso:** Execute manualmente no início do ano letivo ou ao cadastrar novas turmas.

> Utiliza `LockService` para evitar execuções simultâneas.

---

### 2. `UpdateClass` — Atualização Manual de Turmas

> **Arquivo:** `UpdateClass.gs`

Reescreve completamente os dados de alunos nas planilhas já existentes.

**O que faz:**
- Limpa e reescreve os dados nas abas **Leitura** e **Níveis de Escrita**
- Atualiza nome, nível, etnia e origem (1° ano)

**Uso:** Execute manualmente para correções completas de uma turma.

> Este script **sobrescreve** todos os dados existentes. Use com cautela.

---

### 3. `onEdit` — Sincronização em Tempo Real

> **Arquivo:** `onEdit.gs`

Trigger instalável que detecta edições na planilha de configuração e reflete as alterações automaticamente na planilha de destino da turma correspondente.

**Escuta edições nas colunas:**

| Coluna | Dado |
|--------|------|
| B | Nome do aluno |
| C | Nível de leitura |
| D | Etnia |
| E | Origem *(apenas 1° ano)* |

**Comportamento:**
- **Edição:** atualiza o aluno na linha correspondente das abas **Leitura** e **Níveis de Escrita**
- **Exclusão (nome apagado):** limpa as células do aluno na planilha de destino

**Instalação:** O trigger é criado automaticamente pelo `Create`, mas pode ser instalado manualmente em *Extensões → Apps Script → Gatilhos*.

---

### 4. `SmartLock` / `LockArquivo` — Proteção de Planilhas

> **Arquivo:** `Lock.gs`

Dois mecanismos complementares de proteção de colunas por mês.

**`LockArquivo(planilha)`**
- Chamada automaticamente pelo `Create` após criar cada planilha
- Protege as colunas de meses anteriores ao atual
- Libera apenas o mês corrente para edição

**`SmartLock()`**
- Varre todas as planilhas na pasta base
- Reaaplica a proteção em arquivos editados nos últimos **40 minutos**
- Indicada para ser usada via trigger por tempo (ex: a cada hora)

---

### 5. `UnlockMonth` — Controle Mensal de Acesso

> **Arquivo:** `UnlockMonth.gs`

Atualiza as permissões de edição mensalmente, liberando o mês atual e bloqueando os anteriores.

**O que faz:**
- Identifica o mês atual e localiza a proteção com descrição `MES_XXX` correspondente
- Concede acesso de edição ao admin e aos editores do arquivo para o mês atual
- Mantém os demais meses acessíveis apenas ao admin
- Atua somente na aba **Níveis de Escrita**

**Uso:** Configure um trigger mensal (ex: todo dia 1° do mês) em *Extensões → Apps Script → Gatilhos*.

> Não executa ação em janeiro (`mesReal === 0`).

---

## Configuração

### Pré-requisitos

- Conta Google com acesso ao Google Drive e Google Sheets
- Planilhas-modelo já criadas no Drive com as abas: **Leitura**, **Matemática**, **Produção** e **Níveis de Escrita**
- IDs dos modelos e da pasta base configurados nas constantes de cada script

### Constantes a configurar

Nos scripts, substitua os valores das constantes:

```javascript
const CONFIG = {
  MODELO_1:      'ID_DA_PLANILHA_MODELO_1ANO',
  MODELO_2:      'ID_DA_PLANILHA_MODELO_2ANO',
  MODELO_3a5:    'ID_DA_PLANILHA_MODELO_3A5ANO',
  ID_PASTA_BASE: 'ID_DA_PASTA_NO_DRIVE'
};
```

### Estrutura esperada na planilha de configuração

| Célula | Conteúdo |
|--------|----------|
| A5 | Nome da sala (ex: `1° ANO A`) |
| B2 | Nome da escola |
| B4 | Nome do professor(a) |
| D63 | Ano letivo atual |
| B8:B62 | Nomes dos alunos |
| C8:C62 | Nível de leitura |
| D8:D62 | Etnia |
| E8:E62 | Origem *(apenas 1° ano)* |

---

##  Como usar

1. Abra a planilha de configuração no Google Sheets
2. Acesse **Extensões → Apps Script**
3. Cole os scripts nos arquivos correspondentes
4. Atualize as constantes com os IDs corretos do seu Drive
5. Execute `Create` para gerar as planilhas das turmas
6. Configure os triggers:
   - `SmartLock` → a cada hora (trigger por tempo)
   - `UnlockMonth` → dia 1° de cada mês (trigger por tempo)
   - `onEdit` → criado automaticamente pelo `Create`

---

## 📄 Licença

Este projeto é de uso livre para fins educacionais.
