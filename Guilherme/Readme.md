### Projeto Modelagem de software branch do Guilherme
# Dossiê de Arquitetura Viva — Monitoramento Industrial IoT

**Unidade Curricular:** Modelagem de Software  
**Curso:** Engenharia de Software — 4º Período  
**Instituição:** UniSenai  
**Docente:** Sidney L. da Silva  
**Atividade:** Avaliação Prática Supervisionada (APS 2)

---

## 1. Apresentação do Sistema

O projeto consiste no desenvolvimento de um sistema de monitoramento de máquinas industriais utilizando sensores de Internet das Coisas (IoT). A proposta tem como objetivo acompanhar continuamente as condições de funcionamento dos equipamentos por meio da coleta de informações de temperatura e vibração, auxiliando na identificação antecipada de possíveis falhas.

O sistema utiliza sensores instalados nas máquinas para coletar informações operacionais e transmiti-las para uma plataforma centralizada. Após o recebimento, os dados são validados, armazenados e submetidos a análises que permitem identificar alterações nos padrões de funcionamento dos equipamentos.

A partir dessas análises, a plataforma identifica situações que exigem atenção ou que apresentam condições críticas, permitindo registrar ocorrências de manutenção e notificar os responsáveis.

Dessa maneira, o sistema busca contribuir para a manutenção preventiva e preditiva dos equipamentos industriais, reduzindo a ocorrência de falhas inesperadas, diminuindo o tempo de máquinas paradas e auxiliando na tomada de decisões relacionadas à manutenção.

## 2. Funcionalidade Crítica Selecionada

**Análise de Limiares Críticos de Temperatura e Vibração.**

A funcionalidade selecionada é responsável por analisar os dados coletados pelos sensores IoT e identificar possíveis condições anormais no funcionamento das máquinas industriais.

Após o recebimento de uma leitura, o sistema realiza sua validação e armazenamento. Em seguida, os valores de temperatura e vibração são analisados individualmente, permitindo identificar se os indicadores estão dentro dos parâmetros normais, em situação de atenção ou em condição crítica.

Quando pelo menos um dos indicadores ultrapassa seu limite crítico, o sistema registra uma ocorrência de manutenção e solicita o envio de uma notificação aos responsáveis pelo equipamento.

Essa funcionalidade foi escolhida por representar uma etapa essencial do sistema, pois conecta a coleta de dados dos sensores à identificação de possíveis falhas e ao processo de manutenção industrial.

## 3. Regras de Negócio

Para a modelagem da funcionalidade, foram definidos limites de referência ilustrativos para temperatura e vibração.

### 3.1. Limiares de Temperatura

| Temperatura | Classificação |
|---|---|
| Até 70 °C | Normal |
| Acima de 70 °C até 85 °C | Atenção |
| Acima de 85 °C | Crítico |

### 3.2. Limiares de Vibração

| Vibração | Classificação |
|---|---|
| Até 4 mm/s | Normal |
| Acima de 4 até 8 mm/s | Atenção |
| Acima de 8 mm/s | Crítico |

**Observação:** Os valores apresentados são parâmetros hipotéticos utilizados para fins acadêmicos e de modelagem. Em uma aplicação industrial real, os limites deverão ser configurados conforme as características de cada máquina, sensores, condições operacionais e recomendações técnicas aplicáveis.

### 3.3. Regras de Processamento

1. Toda leitura recebida deve possuir identificação da máquina, temperatura, vibração e data e horário da coleta.
2. Leituras inválidas não devem ser processadas pela análise de limiares.
3. Leituras válidas devem ser armazenadas no banco de dados.
4. A análise de temperatura e vibração deve ocorrer paralelamente.
5. Após as duas análises, os resultados devem ser consolidados.
6. Caso pelo menos um indicador esteja em condição crítica, uma ocorrência de manutenção deve ser registrada.
7. O sistema deve solicitar uma notificação aos responsáveis quando uma ocorrência crítica for identificada.
8. Leituras normais ou em condição de atenção devem ser registradas sem abertura automática de ocorrência crítica.

## 4. Tecnologias e Recursos Propostos

A arquitetura proposta considera as seguintes tecnologias:

| Tecnologia | Aplicação |
|---|---|
| ESP32 | Comunicação e processamento nos dispositivos IoT |
| Sensores de temperatura e vibração | Coleta dos dados industriais |
| MQTT | Comunicação entre sensores e plataforma |
| Eclipse Mosquitto | Broker de mensagens MQTT |
| Node.js | Desenvolvimento dos serviços Back-End |
| PostgreSQL | Persistência dos dados |
| React | Desenvolvimento da interface Web |
| Mermaid.js | Modelagem dos diagramas como código |

As tecnologias representam a proposta arquitetural do sistema, não necessariamente uma implementação já concluída.

## 5. Dossiê de Arquitetura

Os artefatos apresentados a seguir representam diferentes perspectivas da mesma funcionalidade crítica. A modelagem foi desenvolvida utilizando Mermaid, permitindo que os diagramas sejam armazenados como código-fonte e versionados juntamente com o restante do projeto.

### 5.1. Diagrama de Sequência

**Arquivo:** `docs/diagrams/sequencia.mmd`

O diagrama de sequência representa a comunicação cronológica entre os componentes envolvidos no recebimento e na análise das leituras dos sensores.

São demonstradas as chamadas entre os serviços, a validação dos dados, o armazenamento das informações, a análise paralela de temperatura e vibração e o envio assíncrono de notificações quando uma condição crítica é identificada.

\`\`\`mermaid
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

    Note over Sensor,API: LeituraSensorDTO

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
        API-->>Sensor: ConfirmacaoRecebimentoDTO
    end
\`\`\`

### 5.2. Diagrama de Atividades com Raias

**Arquivo:** `docs/diagrams/atividades.mmd`

O diagrama de atividades representa o fluxo de trabalho da funcionalidade, organizando as responsabilidades em três raias: Sensores IoT, Plataforma Back-End e Serviço de Notificação.

O fluxo demonstra a coleta e transmissão das leituras, a validação e o armazenamento dos dados, a execução paralela das análises e a tomada de decisão sobre a abertura de uma ocorrência de manutenção.

A utilização de paralelismo permite que os indicadores de temperatura e vibração sejam analisados simultaneamente. Após a conclusão das duas atividades, os resultados são sincronizados para a consolidação.

### 5.3. Diagrama de Transição de Estados

**Arquivo:** `docs/diagrams/estados.mmd`

O diagrama de estados representa o ciclo de vida de uma ocorrência de manutenção gerada após a identificação de uma condição crítica.

A ocorrência passa pelos estados de Detecção, Registro, Notificação, Aguardando Manutenção, Em Manutenção e Encerramento.

Também são considerados cenários alternativos, como falha no envio da notificação, suspensão de uma manutenção e identificação de um falso positivo.

A modelagem utiliza gatilhos, condições de guarda e ações de efeito para controlar as transições, evitando que uma ocorrência avance para um estado incompatível com sua situação atual.

### 5.4. Visão Arquitetural Macro — C4 Nível 2

**Arquivo:** `workspace.mmd`

A visão arquitetural representa os principais contêineres e suas responsabilidades dentro da plataforma de monitoramento industrial.

A arquitetura contempla a camada IoT, o broker MQTT, os serviços Back-End, a camada de persistência e a interface Web.

O protocolo MQTT é utilizado na comunicação entre os sensores e a plataforma, enquanto a comunicação entre a interface e os serviços de aplicação ocorre por meio de HTTPS e REST.

O PostgreSQL é responsável pelo armazenamento das leituras, informações das máquinas, limites configurados e ocorrências de manutenção.

### 5.5. Registro de Decisão Arquitetural — ADR 0001

**Arquivo:** `docs/architecture/adr-0001.md`

**Título:** ADR 0001 — Adoção do protocolo MQTT para comunicação IoT.

**Status:** Aprovado.

A decisão arquitetural estabelece a utilização do protocolo MQTT para comunicação entre os sensores industriais e a plataforma de monitoramento.

A escolha considera a necessidade de receber continuamente informações de diversos dispositivos IoT, utilizando uma comunicação leve e baseada no modelo Publish/Subscribe.

Entre as principais vantagens estão a redução da sobrecarga de comunicação, a facilidade de expansão da infraestrutura e a possibilidade de comunicação assíncrona entre os dispositivos e os serviços.

Como consequências, são considerados os riscos relacionados à disponibilidade do broker, à possibilidade de mensagens duplicadas e à necessidade de implementar autenticação, controle de acesso e mecanismos de reconexão.

A decisão busca garantir uma comunicação adequada às características do ambiente industrial e às necessidades de monitoramento contínuo.

## 6. Consistência entre os Artefatos

Os cinco artefatos foram elaborados considerando a mesma funcionalidade crítica e as mesmas regras de negócio.

O diagrama de sequência demonstra a comunicação entre os componentes responsáveis pelo processamento das leituras. O diagrama de atividades apresenta o fluxo operacional desse processamento, incluindo a execução paralela das análises. A máquina de estados representa a evolução das ocorrências de manutenção geradas a partir dos resultados críticos.

A visão C4 apresenta a organização arquitetural dos componentes necessários para executar essa funcionalidade, enquanto a ADR registra e justifica uma decisão tecnológica importante para sua implementação.

Essa relação permite manter a coerência entre os diferentes níveis de modelagem e facilita a manutenção e evolução da documentação ao longo do desenvolvimento do sistema.

## 7. Estrutura do Repositório

A organização dos arquivos segue a estrutura proposta para a atividade:

\`\`\`text
aps2-nome-sobrenome/
│
├── docs/
│   ├── architecture/
│   │   └── adr-0001.md
│   │
│   └── diagrams/
│       ├── sequencia.mmd
│       ├── atividades.mmd
│       └── estados.mmd
│
├── workspace.mmd
│
└── README.md
\`\`\`

## 8. Considerações Finais

O desenvolvimento deste dossiê permitiu representar a funcionalidade de análise de limiares críticos sob diferentes perspectivas da Engenharia de Software.

A utilização de diagramas como código possibilita manter os modelos próximos da documentação técnica do projeto, facilitando o versionamento, a revisão e a identificação de possíveis inconsistências arquiteturais.

A modelagem também evidencia a importância da comunicação entre dispositivos IoT, do processamento paralelo de informações e do controle dos estados das ocorrências de manutenção.

Por fim, a proposta demonstra como a integração entre sensores, serviços de processamento e mecanismos de notificação pode contribuir para um monitoramento industrial mais eficiente, auxiliando na identificação antecipada de problemas e na redução de paradas inesperadas dos equipamentos.

---

**Desenvolvido como parte da Avaliação Prática Supervisionada (APS 2) — Modelagem de Software.**
