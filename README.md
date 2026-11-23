# Recomendação Musical com Neo4j

Projeto de banco de dados em grafos desenvolvido com Neo4j e Cypher para gerar recomendações musicais personalizadas por meio das conexões entre usuários, músicas, artistas e gêneros.

## 1. Problema

Sistemas de streaming possuem muitas músicas e precisam ajudar cada pessoa a descobrir conteúdos relevantes. Este projeto representa usuários e catálogo musical em um grafo e utiliza as interações entre usuários para identificar gostos semelhantes e recomendar músicas ainda não consumidas pelo usuário-alvo.

## 2. Por que Neo4j?

O problema envolve conexões: usuários escutam e curtem músicas, seguem artistas, artistas interpretam músicas e músicas pertencem a gêneros. O modelo de grafos representa essas relações diretamente e permite percorrer caminhos para encontrar usuários semelhantes e candidatos a recomendação.

## 3. Modelo do grafo

### Nós

| Label | Propriedades principais |
|---|---|
| `Usuario` | `user_id`, `nome` |
| `Musica` | `track_id`, `track_name`, `album_name`, `popularity`, características de áudio |
| `Artista` | `artists` |
| `Genero` | `track_genre` |

### Relacionamentos

| Origem | Relacionamento | Destino |
|---|---|---|
| `Usuario` | `ESCUTOU` | `Musica` |
| `Usuario` | `CURTIU` | `Musica` |
| `Usuario` | `SEGUE` | `Artista` |
| `Artista` | `INTERPRETA` | `Musica` |
| `Musica` | `PERTENCE_A` | `Genero` |

Veja o diagrama em [`docs/modelo-grafo.mmd`](docs/modelo-grafo.mmd).

No Neo4j Browser, o esquema também pode ser visualizado com:

```cypher
CALL db.schema.visualization();
```

## 4. Dataset

O catálogo musical foi carregado a partir do Spotify Tracks Dataset, contendo metadados das faixas, artistas, gêneros e características de áudio.

O CSV original não está incluído neste repositório. Adicione no README o link exato da fonte que você utilizou e respeite os termos de distribuição do dataset. Para reproduzir a carga, use o Neo4j Aura Importer/Data Importer com os campos mapeados para os nós e relacionamentos descritos acima.

Propriedades de áudio usadas anteriormente no projeto incluem `energy`, `danceability`, `valence`, `acousticness`, `instrumentalness` e `tempo`.

## 5. Como executar

1. Abra seu banco Neo4j AuraDB.
2. Importe o dataset musical, caso ainda não tenha feito isso.
3. Execute `cypher/01-constraints.cypher`.
4. Execute `cypher/02-demo-interactions.cypher` para criar três usuários demonstrativos e suas interações.
5. Execute `cypher/03-recommendations.cypher` para consultar recomendações.
6. Altere o parâmetro `userId` para testar outro usuário.
7. Execute as consultas de negócio em `cypher/04-business-queries.cypher`.

> Os três usuários e suas interações são dados de demonstração, não registros reais de ouvintes. Para usar interações reais, substitua os exemplos por dados coletados legitimamente.

## 6. Algoritmo de recomendação

A consulta principal combina três sinais:

- **Similaridade colaborativa:** quantidade de músicas que dois usuários curtiram em comum (peso 2) e escutaram em comum (peso 1).
- **Preferência do usuário semelhante:** uma música curtida por um usuário semelhante recebe peso 3; uma música apenas escutada recebe peso 1.
- **Afinidade com artistas:** uma recomendação interpretada por um artista seguido pelo usuário-alvo recebe um bônus de 1,5.

O algoritmo exclui músicas que o usuário-alvo já escutou ou curtiu. As recomendações são ordenadas pela pontuação calculada.

Os pesos são heurísticos e podem ser ajustados conforme os resultados observados. Com poucos usuários e poucas interações, o resultado serve como demonstração do método, não como avaliação estatística de qualidade em produção.

## 7. Perguntas de negócio respondidas

- Quais músicas podem ser recomendadas para determinado usuário?
- Quais usuários possuem gostos musicais semelhantes?
- Quais músicas são mais curtidas?
- Quais artistas são mais seguidos?
- Quais gêneros aparecem no catálogo?
- Quais músicas um usuário já escutou ou curtiu?

## 8. Troubleshooting

### Propriedade de artista aparece como `null`
No modelo importado, o nome do artista está na propriedade `artists` do nó `Artista`, não necessariamente no nó `Musica`. Para exibir o artista, percorra `(Artista)-[:INTERPRETA]->(Musica)`.

### Propriedades de áudio não aparecem
Confira o mapeamento no Data Importer. `acousticness`, `instrumentalness` e `liveness` devem estar mapeadas como propriedades numéricas de `Musica`.

### A consulta não retorna recomendações
O usuário-alvo precisa ter pelo menos uma interação em comum com outro usuário, e esse outro usuário precisa ter escutado ou curtido uma música que o alvo ainda não escutou nem curtiu.

### Mais relacionamentos do que o esperado
Evite `CONTAINS` para selecionar artistas quando deseja um artista específico. Prefira localizar o artista pelo relacionamento com a música ou por uma propriedade exata. `MERGE` evita duplicar a mesma relação quando o padrão completo já existe.

## 9. Evidências visuais

Adicione capturas de tela do seu próprio Neo4j Browser/Explore em `evidencias/`:

- `01-modelo.png`: resultado de `CALL db.schema.visualization();`
- `02-grafo.png`: usuários, músicas, artistas e gêneros conectados.
- `03-recomendacoes.png`: resultado da consulta principal.
- `04-consultas.png`: resultados de consultas de negócio.

As imagens devem ser capturadas no seu banco, para representar fielmente a execução do projeto.

## 10. Limitações e melhorias futuras

- O resultado depende da quantidade e qualidade das interações disponíveis.
- Os usuários atuais são demonstrativos.
- Os relacionamentos `ESCUTOU` e `CURTIU` representam a existência de uma interação; o modelo pode ser expandido com data, quantidade de reproduções ou duração ouvida.
- Melhorias possíveis: normalização da similaridade (Jaccard), janela temporal, avaliação de precisão/recall e combinação com características de áudio e gênero.

## 11. Tecnologias

- Neo4j AuraDB
- Cypher
- Spotify Tracks Dataset

---
Projeto prático de recomendação musical com banco de dados em grafos.
