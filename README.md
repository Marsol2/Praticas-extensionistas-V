# PulseVida — Saúde conectada

Site acadêmico responsivo para organizar o acompanhamento de pacientes e os fluxos de profissionais de saúde e administração. Esta versão reaproveita a identidade visual, parte dos assets e o domínio funcional do PulseVida enviado pelo grupo, com backend Node/Express e persistência em banco MySQL-compatible.

## O que está implementado

| Perfil | Telas e ações |
|---|---|
| Paciente | Cadastro público exclusivo de paciente; login; resumo; registro e histórico pessoal de aferições; solicitação/cancelamento de consultas; status atualizado pela equipe; histórico de atendimentos; manutenção do próprio nome, e-mail, CPF e Cartão SUS. |
| Profissional de saúde | Fila de solicitações; confirmação/cancelamento de consulta; visualização de aferições registradas pelos pacientes; registro de atendimento com observações e pressão opcional; atendimento vinculado ao paciente/agendamento. |
| Administração | Visão geral sem expor dados clínicos; consulta a usuários e agendamentos; criação controlada de acessos profissionais/administrativos; ativação e desativação de contas; gerenciamento de solicitações; leitura administrativa da pesquisa de satisfação. |
| Público | Página institucional, acesso/cadastro, informações das unidades mencionadas no material original, orientações gerais de cuidado e pesquisa de satisfação persistida. |

A interface se reorganiza para telas pequenas, com navegação compacta, formulários de toque e tabelas com rolagem horizontal. Trata-se de um site web responsivo, não de um aplicativo nativo de loja.

## Iniciar no Preview do projeto
O projeto já está configurado para Node.js na porta 3000, com servidor e banco gerenciado. A conexão `DATABASE_URL` é injetada pelo ambiente Webdev; o código nunca a envia ao navegador. Ao iniciar, o servidor aplica somente as migrações pendentes registradas em `schema_migrations`, cria o primeiro administrador se ainda não houver um e inicia o listener. O healthcheck sem autenticação é `GET /healthz`.

O login inicial do administrador é `admin`. A senha é a que foi informada na caixa protegida de configuração do projeto; ela não está escrita neste README nem no código. Após o primeiro boot bem-sucedido, remova a variável protegida `PULSE_BOOTSTRAP_ADMIN_PASSWORD` no painel de configuração. As contas profissionais seguintes devem ser criadas dentro da área administrativa.

Para apresentar ao grupo, entre como `admin`, crie uma conta profissional e saia. Cadastre um paciente pela página pública; solicite uma consulta com essa conta, entre como profissional para confirmar e registrar o atendimento, depois volte ao paciente e atualize o painel/histórico. Utilize apenas dados fictícios. Nenhuma conta de paciente/profissional nem registro clínico de demonstração foi criado no banco desta entrega.

## Executar fora do Preview

Requisitos: Node.js 22 ou superior, pnpm 11.25.0 (ou Corepack com a versão indicada em `package.json`) e uma base MySQL compatível.

```bash
cp .env.example .env
# Edite DATABASE_URL com as credenciais da sua base MySQL.
# Para criar o primeiro administrador, preencha temporariamente PULSE_BOOTSTRAP_ADMIN_PASSWORD.
pnpm install --frozen-lockfile
pnpm start
```http://localhost:3000

Abra ``. A aplicação executa as migrações versionadas automaticamente no início. O comando `pnpm migrate` executa apenas as migrações e fecha a conexão. Nunca coloque a senha de administrador ou credenciais de banco em `README.md`, `package.json`, imagens, comandos salvos, frontend ou no GitHub; `.env` está ignorado pelo Git.
==========
## Execução e Banco de Dados — MySQL Local (Porta 3306)

### 1. Instalar dependências e iniciar o servidor (porta `3000`)

Na raiz do projeto:

```powershell
pnpm install --frozen-lockfile
```

Em seguida, acesse a pasta `backend` e inicie a aplicação:

```powershell
cd backend
pnpm start
```

Após iniciar, a aplicação fica disponível em:

```text
http://localhost:3000
```

### 2. Conectar ao MySQL local (porta `3306`) e conferir os dados

Abra o MySQL local na porta `3306` (ou pelo terminal passando `-P 3306`), informe a senha configurada e execute:

```powershell
& "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -h 127.0.0.1 -P 3306 -u root -D pulsevida --default-character-set=utf8mb4 -p
```

Dentro do MySQL:

```sql
USE pulsevida;
SHOW TABLES;
SELECT
  id,
  nome,
  usuario,
  email,
  tipo,
  especialidade,
  crm,
  ativo,
  DATE_FORMAT(criado_em, '%d/%m/%Y %H:%i') AS data_horario
FROM usuarios
ORDER BY id;
```

### 3. Padronizar perfis médicos, especialidade e CRM

Se os campos `tipo`, `especialidade` ou `crm` ainda não estiverem preenchidos nas contas médicas, execute:

```sql
UPDATE usuarios
SET tipo = 'medico',
    especialidade = COALESCE(NULLIF(especialidade, ''), 'Cardiologia'),
    crm = COALESCE(NULLIF(crm, ''), 'CRM-RJ 528491')
WHERE usuario = 'pedrosa';

UPDATE usuarios
SET tipo = 'medico',
    especialidade = COALESCE(NULLIF(especialidade, ''), 'Clínica Médica'),
    crm = COALESCE(NULLIF(crm, ''), 'CRM-RJ 529104')
WHERE usuario = 'severo';


UPDATE usuarios
SET tipo = 'medico',
    especialidade = COALESCE(NULLIF(especialidade, ''), 'Clínica Geral'),
    crm = COALESCE(NULLIF(crm, ''), 'CRM-RJ 530112')
WHERE usuario = 'pedro' OR id = 30001;
```

==========

## Banco e segurança

`migrations/001_initial.sql` cria `usuarios`, `sessoes`, `metricas_usuario`, `agendamentos`, `atendimentos` e `pesquisas`. Senhas são derivadas com scrypt e sal aleatório; a tabela de sessões guarda apenas o hash SHA-256 de tokens aleatórios com validade de 12 horas. A autorização acontece no backend, não apenas na interface. O perfil público só pode criar paciente; profissionais e administradores são provisionados por uma conta administrativa. As rotas do paciente filtram os dados pelo ID da sessão autenticada.

As tabelas, eventos e usuários são do banco gerenciado deste projeto. As escritas de desenvolvimento no Preview persistem no mesmo banco de publicação deste projeto. O bootstrap desta entrega criou apenas a primeira conta administrativa; este pacote não importa automaticamente dados que estejam em um banco PostgreSQL antigo do projeto original. Para trazer dados preexistentes, faça uma exportação revisada, mapeamento de campos e conversão de engine fora deste pacote antes de qualquer carga; não conecte diretamente um banco antigo sem cópia de segurança.

Por minimização de dados, CPF e Cartão SUS só são devolvidos ao próprio paciente; não entram nas listagens administrativas ou de consultas. O e-mail do paciente só aparece na fila profissional. A data mínima de consulta segue o fuso `America/Sao_Paulo`. O formulário público de feedback limita a cinco tentativas por endereço IP em 15 minutos (controle em memória do processo; protótipo acadêmico, não proteção antiabuso distribuída).

## Hospedagem e GitHub

O projeto contém `Dockerfile`, lockfile, `.env.example` sem credenciais e a declaração de contêiner com healthcheck `/healthz`. GitHub armazena o código, mas **GitHub Pages não executa Express nem hospeda o banco**. Para deixá-lo acessível online, conecte o repositório a uma hospedagem que execute Node em contêiner e configure a base MySQL e as variáveis protegidas. A publicação do site e o envio a qualquer repositório GitHub não foram feitos nesta etapa.

Para enviar uma cópia a um repositório próprio, crie o repositório no GitHub e, na cópia local do projeto, adicione um remote separado em vez de substituir o remote existente:

```bash
git remote -v
git remote add meu-github https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git
git push -u meu-github main
```

Ajuste o nome do remote se já existir. Antes do push, confira `git status`, confirme que `.env` não está incluído e nunca envie credenciais. Instruções de deploy dependem do provedor que o grupo escolher.

## Limites importantes

Este é um protótipo acadêmico. Não foi submetido a validação clínica, certificação, avaliação de segurança independente ou processo de adequação regulatória/LGPD para operação real. Não armazene dados de saúde reais ou credenciais reais de pacientes. Os valores registrados não recebem diagnóstico nem recomendação médica automática. Para sintomas graves ou emergência, use os serviços oficiais de urgência, não aguarde retorno do site.

<<<<<<< HEAD
# PulseVida — Saúde Conectada
Plataforma web acadêmica responsiva para monitoramento de pacientes e gestão de unidades de saúde em Saquarema.
<<<<<<< HEAD

* **Backend API (Vercel):** Hospedagem serverless dedicada para os serviços Express e rotas do (praticas-extensionistas-v.vercel.app)
* **Banco de Dados (MySQL Cloud):** Persistência relacional em nuvem gerenciando usuários, agendamentos e métricas.
=======
* **Frontend Estático (Netlify):** 
# 🔗 **Aplicação online (Vercel):** [Acessar PulseVida na Vercel — praticas-extensionistas-v.vercel.app](https://praticas-extensionistas-v.vercel.app)
* **Backend API (Vercel):** Hospedagem serverless dedicada para os serviços Express e rotas do sistema.
* **Banco de Dados (MySQL TiDBSCloud):**
=========
## 1__Comando no PowerShell (entrar no TiDB Cloud).
# & "C:\Program Files\MySQL\MySQL Server 8.0\bin\mysql.exe" -h gateway01.sa-east-1.prod.aws.tidbcloud.com -P 4000 -u 2J1akc6etHw1hsG.root -pNqbYLzxvKszl8o9W -D pulsevida --ssl-mode=REQUIRED

--PASSWORD: NqbYLzxvKszl8o9W
============
## 2__Comando dentro do cliente MySQL (selecionar/garantir o banco pulsevida).
# CREATE DATABASE IF NOT EXISTS pulsevida CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# USE pulsevida;
=================
## Listar todas as tabelas do banco pulsevid.
# SHOW TABLES;

## Conferir os dados salvos nas tabelas.
# SELECT * FROM usuarios;
# SELECT * FROM agendamentos;
# SELECT * FROM metricas_usuario;
# SELECT * FROM atendimentos;

##| SELECT id, nome, usuario, tipo, especialidade, crm, ativo, criado_em
    FROM usuarios
    ORDER BY id; |
# SELECT
    id,
    nome,
    usuario,
    tipo,
    especialidade,
    crm,
    ativo,
    DATE_FORMAT(criado_em, '%d/%m/%Y %H:%i') AS data_horario
  FROM usuarios
  ORDER BY id;

 SPersistência relacional em nuvem gerenciando usuários, agendamentos e métricas.

=======
## #### PulseVida — Saúde Conectada
Plataforma web acadêmica responsiva para monitoramento de pacientes e gestão de unidades de saúde em Saquarema.
* **Frontend Estático (Netlify):** [praticas-extensionista-v.netlify.app](https://praticas-extensionista-v.netlify.app)
* **Backend API (Vercel):** Hospedagem serverless dedicada para os serviços Express e rotas do sistema.
* **Banco de Dados (MySQL Cloud):** Persistência relacional em nuvem gerenciando usuários, agendamentos e métricas.
>>>>>>> marsol/main
* **Autenticação:** Sistema seguro com sessões baseadas em hash e senhas derivadas com scrypt.
* **Perfis de Acesso:** Fluxos segregados para Pacientes, Profissionais de Saúde e Administradores.
* **Repositório:** Código-fonte versionado e organizado sob padrões de desenvolvimento full-stack.
* **Repositório Oficial:** Disponível no GitHub para consulta e contribuições acadêmicas.

<<<<<<< HEAD
<<<<<<< HEAD
* praticas-extensionistas-v.vercel.app
=======
====================================
## Atualizações recentes: Portal Médico, Área Administrativa e Banco (TiDB Cloud)

### 1. Contas médicas e perfil administrativo
- **Profissionais de saúde (`tipo = 'medico'`):** contas configuradas com `especialidade` e `crm` (ex.: `pedro` / Dr. Pedro Paulo e `almeida` / Dra. Simone Almeida Couto, `CRM-RJ 530420`).
- **Conta administrativa (`tipo = 'administrador'`):** conta `admin` (`admin@admin.com`) promovida ao perfil `administrador` para gerenciamento de acessos e visualização de dados administrativos, separada do fluxo de `paciente`.

### 2. Fluxo entre Agendamentos, Área do Médico e Atendimentos do Paciente
- **Fila e confirmação de consultas:** agendamentos confirmados (`status = 'Confirmado'`) e solicitações pendentes são exibidos na Área Profissional (`/portal/profissional` e `/portal/medico`) via `GET /api/medico/fila`, trazendo os dados do paciente e a última aferição de `metricas_usuario`.
- **Registro de atendimentos (`atendimentos`):** ao registrar um atendimento clínico (`POST /api/atendimentos`), o backend grava o registro na tabela `atendimentos` (`agendamento_id`, `paciente_id`, `profissional_id`, `pressao`, `observacoes`) e atualiza o agendamento para `'Atendido'`, permitindo que os atendimentos apareçam na tela do médico e na área do paciente (`GET /api/atendimentos`).

### 3. Novas rotas administrativas no backend (`backend/server.js`)
- `GET /portal/administracao`: serve a interface administrativa (utilizando `portal_admin.html` quando presente, com fallback para `portal.html`).
- `GET /api/admin/usuarios`: lista usuários cadastrados para gestão administrativa (protegida por `authenticate` e `roles('administrador')`).
- `PATCH /api/admin/usuarios/:id`: atualiza `tipo`, `especialidade`, `crm` e `ativo` de um usuário (limpando `especialidade` e `crm` quando o perfil não for `medico`).
- `GET /api/admin/pesquisas`: consulta as respostas da tabela `pesquisas` para o perfil `administrador`.

### 4. Boas práticas de terminal e banco de dados (PowerShell + TiDB Cloud)
- **Ambientes distintos:** o **TiDB Cloud** roda na porta `4000` (`gateway01.sa-east-1.prod.aws.tidbcloud.com`) com `--ssl-mode=REQUIRED`, enquanto o **MySQL local** roda em `127.0.0.1:3306` com sua senha local do `root`.
- **Codificação no PowerShell:** execute `chcp 65001` e `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8` no PowerShell **antes** de abrir o `mysql.exe` (nunca dentro do prompt `mysql>`), e passe `--default-character-set=utf8mb4` na conexão.
- **Consultas legíveis e ASCII:** evite `SELECT *` em tabelas com hashes longos (`senha_hash`); para saída apenas em ASCII sem estourar o terminal, use `CAST(coluna AS CHAR CHARACTER SET ascii)` ou saída vertical `\G`.

=======
>>>>>>> marsol/main
