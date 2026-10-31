```mermaid
sequenceDiagram
    actor Tecnico
    actor Maquina
    actor Sensor
    Maquina->>Sensor: Apresenta defeito
    Sensor->>Tecnico: Notifica o Tecnico
    Tecnico->>Maquina: Realiza o Reparo Necessario
    Maquina->>Sensor: Apresenta-se em Estado Padrao
```
