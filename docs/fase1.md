# Milestone 1 - Compreensão e planeamento

## 1. Compreensão do problema

O objetivo deste projeto é criar uma aplicação gráfica para construir,
visualizar e analisar uma rede elétrica.

### 1.1. Principais requisitos

#### Requisitos operacionais

-   Suportar três tipos de componentes:
    -   Gerador;
    -   Subestação;
    -   Consumidor.
-   Suportar linhas elétricas, respeitando as restrições de ligação:
    -   Gerador--Subestação;
    -   Consumidor--Subestação;
    -   Subestação--Subestação.
-   Permitir os níveis de tensão de 0,4 kV, 15 kV e 60 kV.
-   Cada gerador e consumidor tem um nível de ligação, enquanto cada
    subestação suporta um ou mais níveis.
-   Visualizar o grafo, distinguindo visualmente os componentes e os
    estados das linhas, e permitir criar, editar, remover elementos e
    consultar os seus atributos.
-   Permitir anular sucessivamente operações de edição manual, pela
    ordem inversa, restaurando o estado completo anterior.
-   Importar redes nos formatos JSON e YAML, validando a sintaxe e as
    regras do domínio. Em caso de erro, o estado atual da rede deve ser
    preservado.
-   Exportar a rede.
-   Encontrar caminhos entre componentes, considerando apenas linhas
    ativas.
-   Determinar os componentes conexos e os vértices isolados.
-   Simular falhas temporárias numa linha.
-   Identificar ligações críticas (pontes), utilizando o algoritmo de
    Tarjan.
-   Identificar os geradores alcançáveis e os respetivos caminhos para
    um dado consumidor.

#### Requisitos de implementação

-   Utilizar obrigatoriamente Java, JavaFX (sem ficheiros FXML) e a
    biblioteca JavaFXSmartGraph para a representação visual do grafo.
-   Organizar a aplicação em Modelo de Domínio, Grafo, Lógica de
    Negócio, Interface Gráfica e Persistência, mantendo a lógica de
    negócio independente da interface.
-   Aplicar Design Patterns de forma justificada, para resolver
    problemas concretos da arquitetura.
-   Implementar testes unitários que cubram as regras de construção, os
    casos de utilização, situações inválidas e casos-limite.

## 2. Modelo inicial

### 2.1. RedeEletrica (ou Rede)

**Responsabilidade:** Representa o modelo global da rede elétrica. Gere
a coleção de componentes e linhas, assegura a validação global das
regras de construção e gere o histórico de operações para o *undo*
multinível.

### 2.2. Componente (classe abstrata ou interface)

**Responsabilidade:** Define a base comum para todos os elementos da
rede que formam os vértices do grafo. Contém os atributos partilhados:
identificador único e nome.

**Subclasses:**

-   **Gerador:** Modela instalações de produção de energia. Atributos
    específicos: tensão de ligação (kV) e potência máxima de produção
    (MW).
-   **Subestacao:** Modela pontos de interligação e transformação.
    Atributos específicos: níveis de tensão suportados (kV) e capacidade
    máxima processada (MW).
-   **Consumidor:** Modela instalações que consomem energia. Atributos
    específicos: tensão de ligação (kV) e potência necessária (MW).

### 2.3. LinhaEletrica

**Responsabilidade:** Representa as arestas do grafo. Contém atributos
como identificador único, tensão nominal (kV), capacidade máxima de
transporte (MW), comprimento (km) e estado operacional (ativa ou fora de
serviço), além de referenciar os dois componentes extremos.

### 2.4. GestorGrafo (ou adaptador para JavaFXSmartGraph)

**Responsabilidade:** Faz a ligação entre as classes de domínio e a
biblioteca gráfica JavaFXSmartGraph, gerindo a visualização dos vértices
e das arestas na interface.

## 3. Diagrama de classes preliminar

``` mermaid
classDiagram
    class RedeEletrica {
        - String nome
        - Map~String, Componente~ componentes
        - Map~String, LinhaEletrica~ linhas
        + adicionarComponente(Componente c)
        + adicionarLinha(LinhaEletrica l)
        + validarRegras() boolean
    }

    class Componente {
        <<abstract>>
        # String id
        # String nome
        # double tensaoLigacao
    }

    class Gerador {
        - double potenciaMaxima
    }

    class Subestacao {
        - List~Double~ niveisTensaoSuportados
        - double capacidadeProcessada
    }

    class Consumidor {
        - double potenciaNecessaria
    }

    class LinhaEletrica {
        - String id
        - double tensaoNominal
        - double capacidadeMaxima
        - double comprimento
        - boolean ativa
        - Componente extremo1
        - Componente extremo2
    }

    RedeEletrica "1" *-- "*" Componente : contém
    RedeEletrica "1" *-- "*" LinhaEletrica : contém
    Componente <|-- Gerador
    Componente <|-- Subestacao
    Componente <|-- Consumidor
    LinhaEletrica "*" --> "2" Componente : liga
```
