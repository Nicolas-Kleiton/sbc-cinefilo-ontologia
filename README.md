# Cinéfilo: base de conhecimento em RDF, RDFS e OWL

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Nicolas-Kleiton/sbc-cinefilo-ontologia/blob/main/Cinefilo_Ontologia.ipynb)

Grafo de conhecimento sobre cinema escrito em Turtle ([`cinema.ttl`](cinema.ttl)) e consultado
com `rdflib` e SPARQL no notebook [`Cinefilo_Ontologia.ipynb`](Cinefilo_Ontologia.ipynb).

Mini-Projeto 3 — Sistemas Baseados em Conhecimento (UFPB).

## Resumo da base

Nos Mini-Projetos 1 e 2, o Cinéfilo recomendava filmes com regras nítidas e com lógica fuzzy. Aqui
o mesmo domínio é modelado como **ontologia**: em vez de dizer o que fazer com os fatos, a base
declara o que eles significam. Ela descreve 17 filmes reais (de *2001: Uma Odisseia no Espaço*
a *Duna: Parte Dois*), as pessoas que os fizeram, os estúdios que os produziram e os prêmios que
ganharam.

O arquivo segue a separação vista em aula:

- **T-Box** (esquema): classes, hierarquia, propriedades com `rdfs:domain`/`rdfs:range` e axiomas OWL;
- **A-Box** (fatos): os indivíduos e suas relações.

| Item | Na base | Mínimo pedido |
|---|---|---|
| Classes | 18 | 8 |
| Níveis abaixo da raiz | 3 | 2 |
| Propriedades de objeto | 12 | 5 |
| Propriedades de dados | 7 | 5 |
| Instâncias | 68 | 25 |
| Triplas declaradas | 561 | 50 |
| Construções OWL | 7 | 5 |

Todas as 19 propriedades declaram `rdfs:domain` e `rdfs:range`. Depois do raciocínio OWL-RL, o
grafo passa de 561 para 1169 triplas. O notebook confere esses números direto no grafo.

## Principais entidades e classes

| Classe | Instâncias | Exemplos |
|---|---|---|
| `Filme` (e subclasses) | 17 | Interestelar, Parasita, Toy Story 3, Free Solo |
| `Diretor` | 10 | Christopher Nolan, Denis Villeneuve, Stanley Kubrick, Bong Joon-ho |
| `Ator` | 19 | Cillian Murphy, Zendaya, Tom Hanks, Song Kang-ho |
| `Roteirista` | 3 | Jonathan Nolan, Arthur C. Clarke, Paul Haggis |
| `Compositor` | 4 | Hans Zimmer, Ludwig Göransson, Randy Newman |
| `Estudio` | 8 | Warner Bros., Pixar, Legendary, CJ Entertainment |
| `Premio` | 7 | Oscar de Melhor Filme, Palma de Ouro |

Cada pessoa é declarada só com o papel principal. Os demais são inferidos: Nolan é declarado
`Diretor` e passa a ser também `Roteirista` porque aparece em `roteirizadoPor`; Clint Eastwood é
declarado `Diretor` e passa a ser também `Ator` porque está no elenco de *Menina de Ouro*.

### Propriedades

| Propriedade | Tipo | Domain → Range | Característica OWL |
|---|---|---|---|
| `dirigidoPor` / `dirigiu` | objeto | Filme → Diretor | inversas entre si |
| `temNoElenco` / `atuouEm` | objeto | Filme → Ator | inversas entre si |
| `roteirizadoPor` | objeto | Filme → Roteirista | |
| `trilhaSonoraDe` | objeto | Filme → Compositor | |
| `produzidoPor` | objeto | Filme → Estudio | |
| `ganhouPremio` | objeto | Filme → Premio | |
| `influenciou` | objeto | Diretor → Diretor | transitiva |
| `colaborouCom` | objeto | Profissional → Profissional | simétrica |
| `sequenciaDe` | objeto | Filme → Filme | assimétrica, subpropriedade de `posteriorNaFranquiaA` |
| `posteriorNaFranquiaA` | objeto | Filme → Filme | transitiva |
| `anoLancamento` | dados | Filme → `xsd:integer` | funcional |
| `duracaoMinutos` | dados | Filme → `xsd:integer` | funcional |
| `notaIMDb` | dados | Filme → `xsd:decimal` | funcional |
| `anoNascimento` | dados | Pessoa → `xsd:integer` | funcional |
| `nacionalidade` | dados | Pessoa → `xsd:string` | |
| `anoFundacao` | dados | Organizacao → `xsd:integer` | funcional |
| `paisSede` | dados | Organizacao → `xsd:string` | |

Nomes e títulos usam o padrão `rdfs:label`.

## Taxonomia

```
Obra
 └─ Filme                         ≡ schema:Movie
     ├─ Ficcao
     │   ├─ FiccaoCientifica
     │   ├─ Drama
     │   ├─ Acao
     │   ├─ Terror
     │   └─ Animacao
     └─ Documentario              disjunta de Ficcao
Pessoa                            ≡ schema:Person
 └─ Profissional
     ├─ Diretor
     ├─ Ator
     ├─ Roteirista
     └─ Compositor
Organizacao
 └─ Estudio
Premio
```

São quatro hierarquias independentes, declaradas disjuntas entre si: uma obra nunca é uma pessoa,
uma organização ou um prêmio.

- **Obra → Filme → Ficcao/Documentario → gênero.** O primeiro corte separa ficção de documentário,
  que são excludentes. Dentro da ficção, os gêneros **não** são disjuntos, porque um filme pode ter
  mais de um: *Interestelar* é ficção científica e drama. Cada filme declara só os gêneros mais
  específicos; que ele é `Ficcao`, `Filme` e `Obra` vem da transitividade de `rdfs:subClassOf`.
- **Pessoa → Profissional → papel.** Os papéis também não são disjuntos, de propósito: o mesmo
  profissional pode dirigir, escrever e atuar.
- **Organizacao → Estudio.** `Organizacao` deixa espaço para outras instituições (festivais,
  academias) sem mudar o esquema.
- **Premio.** Representa a categoria (ex.: Oscar de Melhor Filme), não a edição.

## Construções OWL e justificativas

| # | Construção | Onde | Por que o domínio pede |
|---|---|---|---|
| 1 | `owl:disjointWith` | `Ficcao` × `Documentario`; `Obra` × `Pessoa` × `Organizacao` × `Premio` | Um filme não pode ser ficção e documentário ao mesmo tempo. O reasoner acusa a contradição se alguém marcar *Free Solo* como drama (seção 4.1 do notebook). |
| 2 | `owl:equivalentClass` | `Filme ≡ schema:Movie`, `Pessoa ≡ schema:Person` | Alinha a base ao vocabulário schema.org, usado por buscadores: quem consulta por `schema:Movie` encontra os filmes. |
| 3 | `owl:inverseOf` | `dirigidoPor`/`dirigiu`, `temNoElenco`/`atuouEm` | Os créditos são registrados a partir do filme, mas a pergunta costuma partir da pessoa ("o que Nolan dirigiu?"). A inversa responde as duas sem duplicar fatos. |
| 4 | `owl:TransitiveProperty` | `influenciou`, `posteriorNaFranquiaA` | Influência se propaga: Kubrick → Nolan → Villeneuve implica Kubrick → Villeneuve. |
| 5 | `owl:SymmetricProperty` | `colaborouCom` | Colaboração é mútua: se Nolan colaborou com Zimmer, Zimmer colaborou com Nolan. |
| 6 | `owl:AsymmetricProperty` | `sequenciaDe` | Se B é sequência de A, A não pode ser sequência de B. A sequência direta não é transitiva, mas é subpropriedade de `posteriorNaFranquiaA`, que é: *Toy Story 3* vem depois de *Toy Story* por inferência. |
| 7 | `owl:FunctionalProperty` | `anoLancamento`, `duracaoMinutos`, `notaIMDb`, `anoNascimento`, `anoFundacao` | Um filme tem um único ano, uma única duração e uma única nota atual. Por isso a atualização da nota usa DELETE/INSERT, e não só INSERT. |

`dirigidoPor` não é funcional de propósito: *Toy Story 2* e *Free Solo* têm dois diretores.

## Consultas

O notebook roda todas as consultas e imprime cada resultado com o número e o propósito.

**Parte 1, `g.triples()`.** Rodam sobre a base sem inferência, para mostrar o que está escrito.

| # | Padrão | Pergunta |
|---|---|---|
| T1 | `(s, p, None)` | Elenco de Interestelar |
| T2 | `(None, p, o)` | Filmes de Villeneuve |
| T3 | `(None, p, None)` | Cinco maiores notas do IMDb |
| T4 | `(s, None, None)` | Tudo o que a base diz sobre Clint Eastwood |
| T5 | `(None, None, o)` | Onde Eastwood aparece como objeto |
| T6 | `(s, None, o)` | Que relação liga Nolan a Villeneuve |
| T7 | `(None, rdf:type, o)` | Filmes do tipo `Ficcao` (vazio sem inferência) |

Entre as duas partes, o notebook roda o reasoner OWL-RL (`owlrl`) e repete padrões de
`g.triples()` antes e depois, mostrando o que cada construção deduziu.

**Parte 2, SPARQL.** Rodam sobre o grafo inferido.

| # | Forma | Propósito |
|---|---|---|
| S1 | SELECT + `FILTER` | Ficção a partir de 2010 com nota ≥ 8.3 |
| S2 | SELECT + `ORDER BY`/`LIMIT` | Cinco filmes mais longos |
| S3 | SELECT + `GROUP BY`/`HAVING` (COUNT, AVG, MIN, MAX) | Estatísticas por diretor, usando a inversa `dirigiu` |
| S4 | SELECT + agregação | Filmes e nota média por gênero (tipos inferidos) |
| S5 | SELECT + `VALUES`/`GROUP_CONCAT` | Profissionais com mais de um papel (inferidos por `range`) |
| S6 | SELECT + `OPTIONAL` | Ficção científica e prêmios, quando houver |
| S7 | SELECT + caminho `influenciou+` | Cadeia de influência sem reasoner |
| S8 | ASK | "Kubrick influenciou Villeneuve?" antes (False) e depois (True) da inferência |
| S9 | ASK | Existe quem atuou num filme que dirigiu? |
| S10 | CONSTRUCT | Exporta o catálogo para o vocabulário schema.org |
| S11 | INSERT DATA | Cadastra *Tenet* |
| S12 | DELETE/INSERT | Atualiza a nota de *Tenet* (propriedade funcional) |
| S13 | DELETE | Modo família: remove filmes de terror e todas as referências a eles |

Os updates trabalham sobre uma cópia do grafo e mostram o estado antes e depois de cada operação.
O arquivo `.ttl` nunca é alterado.

## Como executar

### Opção 1: Google Colab

1. Clique no botão **Abrir no Colab** acima.
2. Execute **Ambiente de execução → Executar tudo**.

A primeira célula instala `rdflib` e `owlrl`, e a segunda baixa o `cinema.ttl` deste repositório.

### Opção 2: Jupyter local

Requer Python 3.10 ou mais recente.

```bash
git clone https://github.com/Nicolas-Kleiton/sbc-cinefilo-ontologia.git
cd sbc-cinefilo-ontologia

python -m venv .venv
# Windows:      .venv\Scripts\activate
# Linux/macOS:  source .venv/bin/activate

pip install -r requirements.txt
jupyter notebook Cinefilo_Ontologia.ipynb
```

No Jupyter, use **Run → Run All Cells**. No VS Code, abra o notebook, escolha o kernel do `.venv`
em **Select Kernel** e clique em **Run All**.

Também é possível executar sem abrir o navegador, gravando as saídas no próprio notebook:

```bash
jupyter nbconvert --to notebook --execute --inplace Cinefilo_Ontologia.ipynb
```

Testado com Python 3.14, rdflib 7.6.0 e owlrl 7.6.2.

## Estrutura do repositório

| Arquivo | Conteúdo |
|---|---|
| `cinema.ttl` | A base de conhecimento (T-Box e A-Box) em Turtle |
| `Cinefilo_Ontologia.ipynb` | Carga da base, inferência e todas as consultas, com as saídas salvas |
| `requirements.txt` | Dependências |

## Fontes dos dados

Anos, elencos, diretores e prêmios são de conhecimento público. Notas do IMDb e durações foram
consultadas em 2026 e são aproximadas: as notas mudam com o tempo.

## Licença

[MIT](LICENSE)
