# Diagramas — techbazar

Diagramas conceituais simplificados produzidos no Hands-on 1 "A Gênese". O escopo é o do slide da aula: um diagrama Entidade-Relacionamento com as três entidades essenciais e um diagrama de fluxo cobrindo a jornada de busca e adição ao carrinho. Ambos são vistas conceituais simplificadas — não representam o modelo físico final do banco nem todas as classes do domínio.

---

## 1. Diagrama Entidade-Relacionamento (ER)

Modelo conceitual com as três entidades exigidas pelo Hands-on: **Usuário, Produto e Pedido**. Entidades auxiliares (Categoria, Carrinho, ItemCarrinho, ItemPedido, Pagamento, FotoProduto) ficam fora desta vista por escolha de escopo.

```mermaid
erDiagram
    USUARIO {
        uuid id PK
        string nome
        string email
        string telefone
        datetime criado_em
    }

    PRODUTO {
        uuid id PK
        string titulo
        string descricao
        decimal preco
        int estoque
        string status
        uuid vendedor_id FK
        datetime publicado_em
    }

    PEDIDO {
        uuid id PK
        uuid comprador_id FK
        uuid produto_id FK
        int quantidade
        decimal total
        string status
        datetime criado_em
    }

    USUARIO ||--o{ PRODUTO : "anuncia"
    USUARIO ||--o{ PEDIDO : "realiza"
    PRODUTO ||--o{ PEDIDO : "referenciado_em"
```

**Notas:**

- Um `USUARIO` pode anunciar zero ou mais `PRODUTO` (papel de vendedor) e realizar zero ou mais `PEDIDO` (papel de comprador). A mesma pessoa pode acumular os dois papéis.
- `PEDIDO` referencia diretamente `PRODUTO` nesta vista simplificada — no modelo detalhado da aplicação, essa relação passará por uma tabela `ItemPedido` para suportar múltiplos itens por pedido.

---

## 2. Diagrama de Fluxo — Busca e adição ao carrinho

Jornada do comprador desde a Home até o produto efetivamente adicionado ao carrinho. Cobre o caminho feliz e três desvios: busca sem resultados, usuário não autenticado e produto sem estoque.

```mermaid
flowchart TD
    A([Usuário acessa a Home]) --> B[Digita termo na barra de busca]
    B --> C{Busca retorna resultados?}
    C -->|Não| D[Exibir mensagem 'Nenhum produto encontrado']
    D --> B
    C -->|Sim| E[Exibir lista de produtos]
    E --> F[Usuário seleciona um produto]
    F --> G[Visualiza detalhes do produto]
    G --> H[Clica em 'Adicionar ao carrinho']
    H --> I{Usuário autenticado?}
    I -->|Não| J[Redirecionar para tela de login]
    J --> K[Usuário autentica com sucesso]
    K --> H
    I -->|Sim| L{Produto com estoque?}
    L -->|Não| M[Exibir 'Produto indisponível']
    L -->|Sim| N([Produto adicionado ao carrinho])
```

**Pontos do fluxo:**

- O laço `D → B` permite ao usuário refinar o termo após uma busca vazia.
- Após o login bem-sucedido (`K`), o fluxo retorna ao ponto exato da ação que o exigiu (clicar em "Adicionar ao carrinho").
- "Produto indisponível" é um estado terminal nesta jornada — o usuário pode voltar à listagem por navegação, fora do escopo deste diagrama.
