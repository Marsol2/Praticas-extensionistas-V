### 6. Ajustes (Semana 7)

# Documentação de Validação, Tour e Ajustes — PulseVida (MVP v2)

Este documento consolida o tour funcional da aplicação **PulseVida**, os usuários ativos validados no banco de dados (**TiDB Cloud**), os registros reais coletados na tabela `pesquisas` via pesquisa de satisfação (`/feedback`) e os ajustes entregues na **Versão 2 do MVP** (Semana 7).

> **Diretriz de privacidade:** Conforme orientação exibida na página `/feedback`, os registros visam avaliar a experiência de uso e navegação sem expor informações clínicas ou dados pessoais sensíveis.

---

## 1. Tour pelo Aplicativo (Fluxo Ponta a Ponta)

### 1.1. Porta de Entrada e Identidade Visual
Ao acessar o **PulseVida**, o usuário encontra a interface principal da plataforma (com comportamento PWA e telas voltadas ao apoio à saúde e organização de atendimentos). O desenho prioriza navegação direta e separação clara entre quem apenas consulta/avalia e quem opera rotinas internas.

### 1.2. Autenticação e Estabilidade de Sessão (`JWT_SECRET`)
No login (contas administrativas, médicas ou de pacientes/testadores):
- O backend valida as credenciais e emite um token assinado com a chave **`JWT_SECRET`**.
- O token acompanha as requisições protegidas (`Authorization: Bearer <token>`), mantendo a sessão estável e impedindo alterações indevidas de perfil no cliente.

### 1.3. Portais por Perfil
- **Fluxo do Paciente:** acompanhamento simples do acesso e do andamento da fila/atendimento, sem complexidade técnica.
- **Portal Médico (`portal_medico`):** visualização da fila e registro de atendimentos integrados à base de dados.
- **Painel Administrativo (`portal_admin`):** gestão operacional de usuários, organização do sistema e acompanhamento das respostas da pesquisa.

### 1.4. Camada de Dados na Nuvem (`TiDB Cloud`) e Segurança de Ambiente
- O backend (porta `3000` local ou ambiente publicado) conecta-se ao **TiDB Cloud** via `mysqls://` na porta `4000` (base `pulsevida`), centralizando usuários, fluxos e avaliações.
- As credenciais reais e o `JWT_SECRET` ficam isolados no arquivo `.env` local e nas variáveis de ambiente da nuvem, enquanto o `.env.example` público no branch `main` mantém apenas placeholders sanitizados.

### 1.5. Pesquisa de Satisfação (`/feedback`)
Canal de coleta rápida de percepções de uso composto por quatro campos gravados na tabela `pesquisas`:
1. **Como prefere se identificar? (`nome`):** papel ou identificação opcional do respondente.
2. **O que funcionou bem? (`positivo`):** pontos positivos observados na experiência.
3. **Que dificuldade encontrou? (`negativo`):** gargalos de navegação, sessão ou configuração.
4. **Sugestões de melhoria (`sugestoes`):** propostas práticas para as próximas iterações.

---

## 2. Usuários Validados no Banco (`usuarios`)

Registros conferidos na instância **TiDB Cloud** (`pulsevida`):

mysql> SELECT id, nome, usuario, tipo, especialidade, crm, ativo, criado_em
    ->     FROM usuarios
    ->     ORDER BY id;
+---------+---------------------------+-----------+---------------+--------------------------+---------------+-------+---------------------+
| id      | nome                      | usuario   | tipo          | especialidade            | crm           | ativo | criado_em           |
+---------+---------------------------+-----------+---------------+--------------------------+---------------+-------+---------------------+
|       1 | Dado  Dollabela           | dado      | paciente      | NULL                     | NULL          |     1 | 2026-11-20 16:56:31 |
|       2 | Admin                     | admin     | administrador | NULL                     | NULL          |     1 | 2026-11-20 16:58:39 |
|   30001 | Dr. Pedro Paulo           | pedro     | medico        | Cl├¡nica Geral           | CRM-RJ 530112 |     1 | 2026-11-20 17:09:24 |
| 2030001 | Neto Caneco               | caneco    | paciente      | NULL                     | NULL          |     1 | 2026-11-21 01:41:18 |
| 2060001 | Neco Boneco Doido         | neco      | paciente      | NULL                     | NULL          |     1 | 2026-11-21 14:44:54 |
| 2090001 | admin                     | zeca      | paciente      | NULL                     | NULL          |     1 | 2026-11-21 15:21:52 |
| 2090004 | Carlos Magno              | carlos    | paciente      | NULL                     | NULL          |     1 | 2026-11-21 15:29:21 |
| 2120001 | Mara Maravilha            | mara      | paciente      | NULL                     | NULL          |     1 | 2026-11-21 19:08:12 |
| 2120002 | Man├® Garrincha           | garrincha | paciente      | NULL                     | NULL          |     1 | 2026-11-21 19:12:54 |
| 2150001 | Dra. Simone Almeida Couto | almeida   | medico        | Cl├¡nica Geral           | CRM-RJ 530420 |     1 | 2026-11-22 14:34:05 |
| 2180002 | Dr. Mario Sergio          | mario     | medico        | Cirurgi├úo Cardiologista | 4321          |     1 | 2026-11-22 15:48:12 |
| 2180004 | Jose Carlos               | jose      | paciente      | NULL                     | NULL          |     1 | 2026-11-22 15:51:00 |
+---------+---------------------------+-----------+---------------+--------------------------+---------------+-------+---------------------+
12 rows in set (0.02 sec)


---

## 3. Feedbacks Reais Registrados no Banco (`pesquisas`)



---

## 4. Ajustes Realizados (Semana 7) — Entregável: MVP v2

### Mudanças Aplicadas
- **Variáveis de ambiente:** `.env.example` sanitizado com placeholders (removendo senhas reais) e `.env` configurado localmente com `JWT_SECRET` para estabilidade das sessões.
- **Banco de dados:** conexão validada no **TiDB Cloud** via `mysqls://` na porta `4000` na base `pulsevida` com codificação `utf8mb4` ativa.
- **Versionamento:** diretório do projeto alinhado, alterações conferidas via `git diff` e enviadas para o branch `main` no GitHub.
- **Fluxo e validação:** reforço na orientação da tela `/feedback` (sem dados clínicos ou pessoais sensíveis) e validação ponta a ponta da gravação e consulta na tabela `pesquisas`.

### Quadro Comparativo (MVP v1 → MVP v2)

| Área / Módulo | Diagnóstico nos Feedbacks e Testes | Melhoria Entregue no MVP v2 |
| :--- | :--- | :--- |
| **Segurança e Ambiente** | Credenciais reais no `.env.example` e necessidade de chave fixa de autenticação | `.env.example` sanitizado e `JWT_SECRET` configurado no ambiente |
| **Banco na Nuvem** | Necessidade de conexão SSL estável e suporte correto a acentuação (`utf8mb4`) | Conexão validada no TiDB Cloud (`mysqls://`, porta `4000`, `utf8mb4`) |
| **Versionamento Git** | Pastas locais divergentes e risco de commit com segredos | Diretório único padronizado, `git diff` validado e `main` sincronizada |
| **Operação e Feedback** | Necessidade de validar ponta a ponta o envio e persistência das avaliações | Fluxo `/feedback` testado em produção com 5 avaliações gravadas em `pesquisas` |
