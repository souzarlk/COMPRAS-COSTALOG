# Compartilhamento de cotações — configuração do banco

A interface de cotações já está preparada para consultar e gravar na tabela `public.costalog_cotacoes` via Supabase quando `assets/config.js` estiver preenchido. Sem isso, as cotações ficam apenas no navegador atual.

## Estrutura esperada da tabela

No SQL Editor do Supabase, crie a tabela:

```sql
create table if not exists public.costalog_cotacoes (
  id text primary key,
  payload jsonb not null,
  updated_at timestamptz not null default now()
);

alter table public.costalog_cotacoes enable row level security;
```

## Configuração no site

Edite `assets/config.js` e preencha:

- `url`: Project URL do Supabase.
- `anonKey`: chave anon/public do projeto (não use a service_role key no site).

A página carrega os registros da tabela e se inscreve para receber alterações em tempo real.

## Segurança importante

A tela atual está temporariamente aberta para configuração. O seletor “Setor de Compras” é apenas uma troca visual de área, não uma autenticação. Para garantir que somente membros do setor de Compras possam ler e editar cotações, o projeto precisa de login real no Supabase Auth e políticas RLS baseadas em um papel atribuído administrativamente (por exemplo, `app_metadata.role = 'compras'`). Não crie políticas abertas para `anon` em produção, pois isso permitiria que visitantes públicos lessem ou alterassem as cotações.
