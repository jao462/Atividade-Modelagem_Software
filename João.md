
# Conciliação de Competências - UniSenai

```mermaid
sequenceDiagram
    actor Estudante
    participant Sistema
    participant Algoritmo
    participant Vagas

    Estudante->>Sistema: Envia seu perfil e competências
    Sistema->>Algoritmo: Envia as competências do estudante
    Sistema->>Vagas: Busca vagas disponíveis
    Vagas-->>Sistema: Retorna os requisitos das vagas

    Sistema->>Algoritmo: Envia os requisitos das vagas
    Algoritmo->>Algoritmo: Compara as competências com as vagas
    Algoritmo-->>Sistema: Retorna o resultado da comparação

    Sistema-->>Estudante: Mostra as vagas compatíveis
```
