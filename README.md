# API pública de anúncios: guia de início rápido

Documentação de exemplo (amostra de escrita técnica de API) para a view pública `public_listings` do **JR Anúncios**: o contrato que liga o painel administrativo (que escreve) à vitrine estática (que só lê).

> **Transparência:** este contrato foi derivado do código do front-end que consome a view. O servidor estava fora do ar quando o escrevi, então ele reflete o que a vitrine usa, e não uma inspeção do banco. Os exemplos são fictícios e não há chave nem endereço reais aqui.

Especificação completa: [`openapi.yaml`](openapi.yaml).

## Em 1 minuto

Você precisa do endereço do projeto e da chave pública (anon). A chave pública é feita para ficar no navegador; o que protege os dados são as regras de acesso no banco.

```bash
# Lista os anúncios, destaques primeiro
curl "https://<SEU-PROJETO>.supabase.co/rest/v1/public_listings?select=*&order=is_featured.desc,featured_at.desc.nullslast,published_at.desc" \
  -H "apikey: <CHAVE-PUBLICA>" \
  -H "Authorization: Bearer <CHAVE-PUBLICA>"
```

```bash
# Busca um único anúncio pelo slug (página de detalhe)
curl "https://<SEU-PROJETO>.supabase.co/rest/v1/public_listings?select=*&slug=eq.jeep-renegade-2020" \
  -H "apikey: <CHAVE-PUBLICA>" \
  -H "Authorization: Bearer <CHAVE-PUBLICA>"
```

A resposta é sempre uma **lista**. Para um slug que não existe, a lista vem vazia (a vitrine mostra "Anúncio não encontrado").

## Exemplo de resposta (fictício)

```json
[
  {
    "slug": "jeep-renegade-2020",
    "title": "Jeep Renegade 2020",
    "description": "Único dono, revisões em dia.",
    "price": 98900,
    "images": ["https://exemplo.com/fotos/renegade-1.jpg"],
    "is_featured": true,
    "featured_at": "2026-09-10T14:00:00Z",
    "published_at": "2026-09-01T12:00:00Z",
    "category_slug": "suvs",
    "category_name": "SUVs",
    "parent_category_name": "Automóveis",
    "whatsapp": "+5551999999999",
    "contact_email": "contato@exemplo.com",
    "attributes": [
      { "slug": "ano", "name": "Ano", "type": "NUMBER", "value": "2020" },
      { "slug": "cambio", "name": "Câmbio", "type": "SELECT", "value": "Automático" }
    ]
  }
]
```

## Campos

| Campo | Tipo | Observação |
|---|---|---|
| `slug` | texto | Identificador único, usado na URL. |
| `title` | texto | |
| `description` | texto ou nulo | |
| `price` | número | Em reais; a vitrine formata em BRL, sem centavos. |
| `images` | lista de URLs | A primeira é a capa. |
| `is_featured` | verdadeiro/falso | Destaques aparecem primeiro. |
| `featured_at` | data e hora ou nulo | Quando virou destaque. |
| `published_at` | data e hora | Usada para ordenar do mais novo ao mais antigo. |
| `category_slug`, `category_name`, `parent_category_name` | texto | A categoria-mãe vem à parte. |
| `whatsapp`, `contact_email` | texto ou nulo | Alimentam os botões de contato. |
| `attributes` | lista | Atributos dinâmicos da categoria (`slug`, `name`, `type`, `value`). |

### Atributos dinâmicos

Cada categoria tem seus próprios atributos (marca, modelo, ano, câmbio, quilometragem...), herdados da categoria-mãe. Por isso `attributes` é uma lista de pares, e não colunas fixas. Os tipos `NUMBER` e `PRICE` chegam como texto e devem ser convertidos para número antes de comparar ou ordenar.

## Erros

| Situação | O que acontece | O que fazer |
|---|---|---|
| Chave ausente ou inválida | `401` | Confira o cabeçalho `apikey` e `Authorization`. |
| View não encontrada | `404` | Confirme o nome `public_listings` e se ela está exposta. |
| Qualquer resposta que não seja 2xx | A vitrine mostra "Não foi possível carregar os anúncios agora." | Tente de novo; se persistir, o serviço está indisponível. |

## Boas práticas de uso

- **Só leitura.** Esta view nunca aceita escrita; a escrita é do painel administrativo.
- **Peça só o que precisa**: use `select=slug,title,price,images` em listas grandes.
- **Cache no cliente** quando possível: a lista muda pouco entre acessos.
- **Trate a lista vazia** como um caso normal, não como erro.

## O que eu faria a seguir

1. Versionar a view como uma API (`public_listings_v1`) para poder evoluí-la sem quebrar a vitrine.
2. Testar o contrato automaticamente a cada deploy do painel.
3. Confirmar os tipos reais dos atributos (`SELECT`, `TEXT`...) com o banco no ar e atualizar este guia.
