## 📋 Índice

- [Sobre o projeto](#-sobre-o-projeto)
- [Funcionalidades](#-funcionalidades)
- [Tipos de usuário](#-tipos-de-usuário)
- [Fluxo de registro](#-fluxo-de-registro)
- [Tecnologias](#-tecnologias)
- [Estrutura do projeto](#-estrutura-do-projeto)
- [Como configurar](#-como-configurar)
- [Gestão de usuários](#-gestão-de-usuários)
- [Segurança](#-segurança)
- [Contribuição](#-contribuição)

---

## 💡 Sobre o projeto

O **Ponto Eletrônico** é uma aplicação web desenvolvida para pequenas e médias equipes que precisam de um sistema simples, bonito e funcional para registrar a jornada de trabalho diária.

Sem instalação. Sem mensalidade. Abre direto no navegador, no computador do escritório, no celular ou no tablet.

---

## ✨ Funcionalidades

### Para todos os colaboradores
- 🔐 **Cadastro e login** com e-mail e senha
- 📝 **Registro de ponto sequencial** — as etapas são liberadas uma a uma, na ordem correta
- ⏱ **Cronômetro ao vivo** — conta o tempo de jornada em tempo real, pausando automaticamente durante o almoço
- 📋 **Histórico pessoal** — visualize todos os seus expedientes com filtro por data
- 📄 **Exportação de PDF individual** — relatório com horas trabalhadas por período, com atalhos para semana atual, mês atual ou todo o período
- 📱 **Interface responsiva** — funciona em desktop, tablet e celular

### Exclusivas para administradores
- 🛡️ **Painel Admin** — visualiza os registros de **todos** os colaboradores em uma única tabela
- 📊 **Estatísticas em tempo real** — total de colaboradores, expedientes, registros do dia e soma de horas
- 🔍 **Filtros avançados** — busca por nome, por data exata ou por intervalo de datas
- 📄 **Exportação de PDF administrativo** — relatório consolidado com resumo por colaborador e total geral de horas
- 🗑️ **Gestão de registros** — exclusão de expedientes individuais

---

## 👥 Tipos de usuário

| | Colaborador | Administrador |
|---|:---:|:---:|
| Registrar ponto | ✅ | ✅ |
| Ver próprio histórico | ✅ | ✅ |
| Exportar PDF próprio | ✅ | ✅ |
| Ver histórico de todos | ❌ | ✅ |
| Exportar PDF consolidado | ❌ | ✅ |
| Excluir registros | ❌ | ✅ |
| Ver estatísticas gerais | ❌ | ✅ |

---

## 🔄 Fluxo de registro

O sistema utiliza um fluxo **sequencial e travado**, cada etapa só é liberada após a conclusão da anterior, evitando registros fora de ordem.

```
┌─────────────────────┐
│  1. Início de       │  ← Libera o cronômetro
│     Expediente      │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  2. Início de       │  ← Cronômetro pausa
│     Almoço          │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  3. Volta do        │  ← Cronômetro retoma
│     Almoço          │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  4. Fim de          │  ← Calcula e salva horas
│     Expediente      │     trabalhadas
└─────────────────────┘
```

O cálculo de horas trabalhadas **desconta automaticamente** o tempo de almoço.

---

## 🛠 Tecnologias

| Camada | Tecnologia | Descrição |
|---|---|---|
| **Frontend** | HTML5 + CSS3 + JavaScript | Interface pura, sem frameworks |
| **Tipografia** | Plus Jakarta Sans (Google Fonts) | Fonte moderna e legível |
| **PDF** | jsPDF + jsPDF-AutoTable | Geração de relatórios no navegador |
| **Autenticação** | Supabase Auth | Login seguro com JWT |
| **Banco de dados** | Supabase (PostgreSQL) | Dados em nuvem com Row Level Security |
| **Hospedagem** | GitHub Pages | Deploy automático gratuito |

---

## 📁 Estrutura do projeto

```
ponto-eletronico/
├── index.html          ← Redireciona para auth.html
├── auth.html           ← Tela de login e cadastro
├── app.html            ← Aplicativo principal
└── README.md           ← Este arquivo
```

---

## ⚙️ Como configurar

### Pré-requisitos
- Conta no [GitHub](https://github.com) (gratuito)
- Conta no [Supabase](https://supabase.com) (gratuito)

### 1. Configurar o Supabase

1. Crie um novo projeto em **supabase.com**
2. Acesse **SQL Editor** e execute o script abaixo para criar as tabelas:

```sql
-- Tabela de perfis de usuário
create table public.profiles (
  id        uuid primary key references auth.users(id) on delete cascade,
  nome      text not null,
  email     text not null,
  role      text not null default 'user',
  created_at timestamptz default now()
);

-- Tabela de registros de ponto
create table public.registros (
  id          uuid primary key default gen_random_uuid(),
  user_id     uuid not null references auth.users(id) on delete cascade,
  nome        text,
  date        date not null,
  logs        jsonb default '[]',
  fim         boolean default false,
  horas       text,
  horas_ms    bigint default 0,
  paused_ms   bigint default 0,
  pause_start timestamptz,
  updated_at  timestamptz default now(),
  created_at  timestamptz default now()
);

-- Habilitar RLS
alter table public.profiles  enable row level security;
alter table public.registros enable row level security;

-- Política de acesso
create policy "acesso_autenticado"
  on public.profiles for all
  using (auth.uid() is not null)
  with check (auth.uid() is not null);

create policy "registros_proprio"
  on public.registros for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

create policy "registros_admin"
  on public.registros for select
  using (
    exists (
      select 1 from public.profiles
      where id = auth.uid() and role = 'admin'
    )
  );
```

3. Acesse **Project Settings → API** e copie:
   - **Reference ID** → sua URL será `https://SEU_ID.supabase.co`
   - **anon / public key** → chave começando com `eyJ...`

### 2. Configurar os arquivos HTML

Em `auth.html` e `app.html`, substitua as duas constantes no início do script:

```javascript
const SUPABASE_URL  = 'https://SEU_REFERENCE_ID.supabase.co';
const SUPABASE_ANON = 'SUA_ANON_KEY_AQUI';
```

### 3. Publicar no GitHub Pages

1. Crie um repositório público no GitHub
2. Suba os arquivos: `index.html`, `auth.html`, `app.html`
3. Vá em **Settings → Pages → Deploy from branch → main**
4. Acesse: `https://SEU_USUARIO.github.io/ponto-eletronico/`

---

## 👤 Gestão de usuários

Todos os comandos são executados no **SQL Editor** do Supabase.

### Ver todos os usuários e seus cargos
```sql
select nome, email, role, created_at 
from public.profiles 
order by created_at;
```

### Promover colaborador para admin
```sql
update public.profiles 
set role = 'admin' 
where email = 'email@dapessoa.com';
```

### Remover cargo de admin
```sql
update public.profiles 
set role = 'user' 
where email = 'email@dapessoa.com';
```

### Inserir perfil manualmente (caso não tenha sido criado no cadastro)
```sql
insert into public.profiles (id, nome, email, role)
values (
  (select id from auth.users where email = 'email@dapessoa.com'),
  'Nome Completo',
  'email@dapessoa.com',
  'user'
);
```

### Excluir usuário completamente
```sql
delete from public.profiles where email = 'email@dapessoa.com';
```

> ⚠️ Após qualquer alteração de cargo, o usuário precisa fazer **logout e login novamente** para que a mudança seja aplicada.

---

## 🔒 Segurança

- **Senhas criptografadas** pelo Supabase Auth (bcrypt)
- **JWT tokens** com expiração automática
- **Row Level Security (RLS)** no banco, cada usuário só acessa seus próprios dados
- **Admin** tem acesso ampliado configurado por política no banco de dados, não no frontend
- A chave `anon/public` usada no frontend é segura para uso público quando o RLS está ativado
- **Nunca** utilize a chave `service_role` no frontend

---

## 📱 Compatibilidade

| Dispositivo | Suporte |
|---|:---:|
| Desktop (Chrome, Edge, Firefox, Safari) | ✅ |
| Tablet | ✅ |
| Celular (Android e iOS) | ✅ |

Em dispositivos móveis, a navegação é feita por uma **barra inferior** (bottom navigation), substituindo a sidebar lateral.

---

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se livre para:

1. Fazer um **fork** do projeto
2. Criar uma branch: `git checkout -b minha-feature`
3. Commitar suas mudanças: `git commit -m 'Adiciona nova feature'`
4. Fazer push: `git push origin minha-feature`
5. Abrir um **Pull Request**

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

<div align="center">


**[Acessar o sistema →](https://jramoss02.github.io/ponto-eletronico/)**

</div>
