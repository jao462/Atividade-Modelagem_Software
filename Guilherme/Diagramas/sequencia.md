```mermaid
sequenceDiagram
    autonumber

    participant Sensor as Sensor IoT
    participant API as API de Recebimento
    participant Ingestao as Servico de Ingestao
    participant Banco as Repositorio
    participant Analise as Servico de Analise
    participant Limiares as Motor de Limiares
    participant Manutencao as Servico de Manutencao
    participant Notificacao as Servico de Notificacao

    Note over Sensor,API: DTO: LeituraSensorDTO (idMaquina, temperatura, vibracao, timestamp)

    Sensor->>API: enviarLeitura(dto)
    API->>Ingestao: receberLeitura(dto)
    Ingestao->>Ingestao: validarLeitura(dto)

    alt Leitura invalida
        Ingestao-->>API: ResultadoValidacaoDTO(erro)
        API-->>Sensor: RespostaDTO(400, erro)
    else Leitura valida
        Ingestao->>Banco: salvarLeitura(dto)
        Banco-->>Ingestao: LeituraSalvaDTO
        Ingestao->>Analise: analisarLeitura(dto)

        par Analise de temperatura
            Analise->>Limiares: avaliarTemperatura(valor)
            Limiares-->>Analise: ResultadoTemperaturaDTO
        and Analise de vibracao
            Analise->>Limiares: avaliarVibracao(valor)
            Limiares-->>Analise: ResultadoVibracaoDTO
        end

        Analise->>Analise: consolidarResultados()

        alt Existe limiar critico
            Analise->>Manutencao: abrirOcorrencia(dto)
            Manutencao-->>Analise: OcorrenciaDTO
            Analise-)Notificacao: enviarAlerta(ocorrenciaDTO)
            Notificacao-->>Analise: ConfirmacaoEnvioDTO
        else Valores normais ou em atencao
            Analise->>Banco: registrarResultado(dto)
            Banco-->>Analise: ResultadoAnaliseDTO
        end

        Analise-->>Ingestao: AnaliseConcluidaDTO
        Ingestao-->>API: RespostaDTO(200, resultado)
```
        API-->>Sensor: ConfirmacaoRecebimentoDTO
    end
