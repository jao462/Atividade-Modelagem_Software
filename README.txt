![Diagrama de Arquitetura](//www.plantuml.com/plantuml/png/NP11Qkj044JtSuh6-t-1lmWs9fWi78l85-X8hRXXD6EcMmh9RK8MFa5Ee8kbi70S1mmMrAlfgbgQS_6XZo5muxbGBSbo2KSkheqUEJajDkpoocyBOgFD_bQh8ZjtkVHC3taFnF3ljymgSLJZ5izNHLfr7VEPN5L_3eave5PeosbmITDf-fpHpVOMhTLSS1JihhG8ZI-4BwDEfzbHx-eGvZhqxmx1wJz96XKUH06VoUQoa00WDeXgU8xQCBb4wSMBSe007M4XtwU6Uqs7ooF8nCHf-a112K0ku6f4PuRUbkbBZ8HG1paRvNFKZkE1PmEwmV2ONJjSfYIDc6dBeF3hBqjmfcEHKHBcOmemmOgIyAuHap9cwE50gtc50F7d-NLEdHOCre0zpue6X8a0NKFxWDJlJERiWu6udeS3SQz3FScBcfVVIr7Ak1334as7Fdu3)

```plantuml
@startuml

actor Sensor
participant "Gateway" as GT
participant "Plataforma IoT" as Iot
participant "Analisador Preditivo" as AP
participant "Gestor de Manutenção" as GM
participant "Sistema de Ordens de Serviço" as SOS

Sensor -> GT : envia dados do sensor
GT -> Iot : Publica telemetria
Iot -> AP : Encaminha dados para análise
AP -> AP : Avalia padrões e tendencias de falha
AP -> GM : Notifica necessidade de manutençao preventiva 
GM -> SOS : Cria ordem de serviço
SOS -> GM : Confirma criação da OS
SOS -> AP : Atualiza status da manutenção
AP -> Iot : Registra telemetria normal

@enduml
```