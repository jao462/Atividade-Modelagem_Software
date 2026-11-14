```mermaid
flowchart TD
subgraph IoT Preventiva
   
        subgraph Telemetria
            S[Sensor]
            A[Analise]
            Sis[Sistema]
            B[Banco de dados]
            F@{ shape: "fork", label: "Fork or Join" }
        end

        subgraph Disparo OS
            T[Tecnico]
            O[OS]
            M[Maquina]
            F2@{ shape: "fork", label: "Fork or Join" }
            F3@{ shape: "fork", label: "Fork or Join" }
        end
   
end
    S ---> F
    F -- Armazena dados --> B
    F -- Mostra dados --> Sis
    Sis -- Compara com dados antigos --> A
    B -- Compara com dados novos --> A
    A -- Sem anormalidades --> S

    A -- Anormalidade detectada --> O
    O ---> F2
    F2 -- Tecnico que efetuara reparo/manutencao --> T
    F2 -- Maquina que necessita de reparo/manutencao --> M
    T ---> F3
    M ---> F3
    F3 -- Reparo/Manutencao finalizado --> S
```
