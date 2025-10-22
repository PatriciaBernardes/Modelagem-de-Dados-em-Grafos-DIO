# 🚀 Projeto de Modelagem de Dados em Grafos para um Serviço de Streaming (DIO)

Este repositório documenta a modelagem e implementação de um **Grafo de Propriedades (Property Graph)** no banco de dados **Neo4j**, com foco na estrutura de um serviço de streaming (como Netflix, HBO Max, etc.).

O projeto foi desenvolvido como parte do curso **Modelagem de Dados em Grafos** da **DIO (Digital Innovation One)**, utilizando a linguagem **Cypher** para consultas e a ferramenta `arrow.app` para visualização inicial.

## 1. Modelo de Dados (Arrow.app)

O modelo de grafo define as entidades principais (Nós) e como elas se relacionam (Arestas/Relacionamentos), permitindo consultas rápidas e intuitivas sobre o conteúdo e as interações do usuário.

### Estrutura do Modelo

| Entidade | Rótulo (Label) | Propriedades Chave |
| :--- | :--- | :--- |
| Usuário | `USER` | `id`, `name`, `age`, `state` |
| Conteúdo | `Movies`, `Series` | `id`, `title` |
| Profissional | `Actor`, `Director` | `id`, `name` |
| Categoria | `Genre` | `id`, `type` |

**Propriedades em Relacionamentos:**

| Relacionamento | Tipos de Nós Conectados | Propriedade | Exemplo |
| :--- | :--- | :--- | :--- |
| `:WATCHED` | `USER` -> `Movies/Series` | `rating` (Nota) | `rating: 5` |

### Visualização do Modelo

<img width="967" height="635" alt="Image" src="https://github.com/user-attachments/assets/9cce7c6a-51f0-4a72-bfff-79ad97f0b9d0" />

## 2. Implementação no Neo4j (Cypher)

A implementação foi realizada usando a linguagem de consulta **Cypher** para criar a estrutura do esquema e popular o banco de dados.

### 2.1. Criação de Restrições (Schema)

As restrições de unicidade garantem a integridade dos dados e otimizam a performance das consultas, agindo como chaves primárias.

```cypher
// Restrições de Unicidade
CREATE CONSTRAINT IF NOT EXISTS FOR (u:USER) REQUIRE u.id IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS FOR (m:Movies) REQUIRE m.id IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS FOR (s:Series) REQUIRE s.id IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS FOR (a:Actor) REQUIRE a.id IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS FOR (d:Director) REQUIRE d.id IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS FOR (g:Genre) REQUIRE g.type IS UNIQUE;
```


### 2.2. Inserção de Dados e Conexões

Exemplo de inserção de dados e criação de um relacionamento complexo que envolve a propriedade `rating` no relacionamento `:WATCHED`:

```cypher
// 1. Criar e Conectar Entidades Chave
MERGE (u:USER {id: 'u-001', name: 'Alice Silva', age: 28, state: 'SP'});
MERGE (m:Movies {id: 'm-001', title: 'Inception'});
MERGE (a:Actor {id: 'a-001', name: 'Leonardo DiCaprio'});
MERGE (d:Director {id: 'd-001', name: 'Christopher Nolan'});
MERGE (g:Genre {type: 'Sci-Fi'});

// 2. Conexão de Profissionais
MATCH (m:Movies {title: 'Inception'}), (a:Actor {name: 'Leonardo DiCaprio'})
MERGE (a)-[:ACTED_IN]->(m);

MATCH (m:Movies {title: 'Inception'}), (d:Director {name: 'Christopher Nolan'})
MERGE (d)-[:DIRECTED]->(m);

// 3. Conexão de Gênero
MATCH (m:Movies {title: 'Inception'}), (g:Genre {type: 'Sci-Fi'})
MERGE (m)-[:IN_GENRE]->(g);

// 4. Conexão de Visualização (com Propriedade)
// Alice Silva assiste Inception e atribui a nota 5.
MATCH (u:USER {name: 'Alice Silva'}), (m:Movies {title: 'Inception'})
MERGE (u)-[w:WATCHED]->(m)
SET w.rating = 5, w.date = date('2025-10-22') // Adiciona a nota e a data
RETURN u.name, m.title, w.rating;
```
### 3. Resultado no Neo4j Browser

A imagem a seguir mostra o resultado da execução dos comandos, demonstrando um subgrafo centrado no filme "Inception". É possível verificar todos os tipos de relacionamentos implementados e as propriedades registradas (como o `rating: 5`) no painel lateral.

<img width="1847" height="933" alt="Image" src="https://github.com/user-attachments/assets/d96d37a0-ffee-4172-9335-f63d61434025" />


#### 4. Próximos Passos (Consultas)

Com esta estrutura, é possível realizar consultas complexas de forma eficiente, como:

Recomendar filmes: "Quais filmes o usuário X ainda não viu, mas que são do mesmo Gênero e dirigidos pelos mesmos Diretores dos filmes que ele deu nota 5?"

Análise de Influência: "Quais Atores estão conectados a mais Séries de Gêneros de Ação e são populares no estado de SP?"
