# ADR-001: Adoção da stack EMQX, Apache Kafka, TimescaleDB e Grafana para o Monitoramento IoT de Manutenção Preventiva

| Campo | Valor |
|---|---|
| **Status** | Aceito |
| **Data** | 13/11/2026 |
| **Projeto** | Monitoramento IoT de Manutenção Preventiva |
| **Categoria** | Arquitetura de dados e integração |
| **Decisores** | Equipe de Arquitetura / Governança de Software |

---

## 1. Título

**Adotar EMQX (ingestão MQTT), Apache Kafka (mensageria), TimescaleDB (armazenamento de séries temporais) e Grafana (visualização) como stack principal da plataforma de Monitoramento IoT de Manutenção Preventiva.**

---

## 2. Status

**Aceito**

---

## 3. Contexto

A aplicação industrial **Monitoramento IoT de Manutenção Preventiva** tem como objetivo coletar, de forma contínua, dados de sensores instalados em equipamentos (por exemplo: temperatura, vibração, corrente elétrica, pressão e horas de operação) para permitir a detecção precoce de anomalias, a análise de tendências de degradação e o planejamento de intervenções antes que ocorram falhas.

O cenário apresenta as seguintes características e restrições:

- **Alto volume e frequência de dados:** muitos sensores publicando leituras em intervalos curtos, gerando fluxo contínuo de dados com padrão de série temporal.
- **Dispositivos heterogêneos e com recursos limitados:** os sensores e gateways industriais necessitam de um protocolo leve, com baixo consumo de banda e tolerância a conexões instáveis (MQTT é o padrão de fato em IoT).
- **Desacoplamento entre origem e consumo:** os dados dos equipamentos precisam ser entregues a múltiplos consumidores (persistência, regras de alerta, analytics e, futuramente, modelos de manutenção preditiva) sem que o equipamento ou o ponto de entrada conheça cada um deles.
- **Confiabilidade e rastreabilidade:** perda de mensagens pode comprometer o diagnóstico de falhas. É necessário permitir reprocessamento histórico de eventos.
- **Consultas analíticas sobre o histórico:** é preciso agregar, comparar e consultar dados por janelas de tempo (médias, picos, tendências), além de manter retenção configurável e compressão de dados antigos.
- **Visibilidade operacional:** equipes de manutenção e operação precisam de dashboards e alertas em tempo quase real, sem depender de desenvolvimento de telas sob medida.
- **Preferência por tecnologias consolidadas:** adoção de componentes com ecossistema maduro, ampla comunidade e possibilidade de execução em infraestrutura própria (on-premises ou nuvem), preservando soberania dos dados industriais.

Foi necessário, portanto, definir uma arquitetura de referência que cubra todo o fluxo: **ingestão → transporte → armazenamento → visualização**.

---

## 4. Decisão

Decidimos adotar as quatro tecnologias abaixo, cada uma com responsabilidade bem definida na arquitetura:

| Camada | Tecnologia | Responsabilidade |
|---|---|---|
| Ponto de entrada (ingestão) | **EMQX** | Broker MQTT que recebe as publicações dos sensores/gateways, gerencia conexões, autenticação, autorização (ACL) e tópicos. |
| Mensageria | **Apache Kafka** | Barramento de eventos que transporta as mensagens do ponto de entrada até os serviços principais, garantindo desacoplamento, durabilidade, escalabilidade e possibilidade de reprocessamento. |
| Armazenamento | **TimescaleDB** | Banco de dados de séries temporais (extensão do PostgreSQL) para persistir as leituras dos sensores, com hypertables, agregações contínuas, compressão e políticas de retenção. |
| Visualização | **Grafana** | Interface gráfica para dashboards, exploração de dados e alertas, consumindo diretamente o TimescaleDB. |

### Fluxo de dados

```text
 ┌──────────────┐   MQTT    ┌────────┐  Bridge/Conector  ┌──────────────┐
 │ Sensores /   │ ────────► │  EMQX  │ ────────────────► │ Apache Kafka │
 │ Gateways     │           └────────┘                   └──────┬───────┘
 └──────────────┘                                               │ consumo
                                                                ▼
                                                  ┌─────────────────────────┐
                                                  │ Serviços principais     │
                                                  │ (processamento, regras, │
                                                  │  persistência)          │
                                                  └───────────┬─────────────┘
                                                              │ escrita
                                                              ▼
                                  ┌─────────┐  consulta  ┌─────────────┐
                                  │ Grafana │ ◄───────── │ TimescaleDB │
                                  └─────────┘            └─────────────┘
```

### Diretrizes de implementação

1. **Ingestão:** sensores e gateways publicam via MQTT no EMQX, usando uma convenção padronizada de tópicos (por exemplo, `planta/{unidade}/{equipamento}/{sensor}/telemetria`), com TLS, autenticação e ACLs por dispositivo ou grupo de dispositivos.
2. **Integração EMQX → Kafka:** o encaminhamento das mensagens para o Kafka será feito pelos mecanismos de integração nativos do EMQX (regras e conectores/bridge para Kafka), evitando a criação de código intermediário desnecessário.
3. **Transporte:** os tópicos Kafka serão particionados considerando a chave do equipamento (para preservar a ordem das leituras por equipamento) e configurados com fator de replicação e retenção compatíveis com a criticidade dos dados.
4. **Persistência:** os serviços consumidores do Kafka gravarão as leituras em *hypertables* do TimescaleDB, com particionamento por tempo, compressão para dados antigos, agregações contínuas para consultas frequentes e políticas de retenção.
5. **Visualização e alertas:** o Grafana usará o TimescaleDB (via data source PostgreSQL/Timescale) como fonte principal, com dashboards provisionados como código e alertas configurados para limiares e tendências relevantes à manutenção preventiva.
6. **Formato das mensagens:** payloads com esquema versionado e identificação do equipamento, sensor, timestamp de origem e unidade de medida, a fim de facilitar evolução e validação.

---

## 5. Consequências Positivas

- **Escalabilidade ponta a ponta:** EMQX suporta grande número de conexões simultâneas de dispositivos e o Kafka escala horizontalmente por partições, atendendo ao crescimento do número de sensores e da taxa de mensagens.
- **Protocolo adequado ao ambiente industrial:** o MQTT é leve, funciona bem em redes instáveis e conta com amplo suporte em sensores, CLPs e gateways industriais.
- **Desacoplamento entre produtores e consumidores:** o Kafka permite adicionar novos consumidores (alertas, analytics, modelos preditivos, integração com ERP/CMMS) sem alterar a camada de ingestão nem os equipamentos.
- **Durabilidade e reprocessamento:** a retenção de eventos no Kafka permite reprocessar dados históricos após falhas, correções de lógica ou criação de novas análises, aumentando a resiliência da solução.
- **Absorção de picos (buffering):** o Kafka protege o banco de dados contra picos de ingestão e indisponibilidades temporárias dos consumidores.
- **Armazenamento otimizado para séries temporais:** o TimescaleDB oferece alto desempenho de ingestão e consulta por janelas de tempo, compressão nativa, agregações contínuas e políticas de retenção, reduzindo custo de armazenamento e tempo de consulta.
- **Familiaridade com SQL/PostgreSQL:** por ser uma extensão do PostgreSQL, o TimescaleDB reduz a curva de aprendizado, permite uso de SQL padrão, ferramentas, drivers e ecossistema já conhecidos, além de possibilitar a combinação de dados de séries temporais com dados relacionais (cadastro de equipamentos, ordens de manutenção).
- **Observabilidade sem desenvolvimento customizado:** o Grafana entrega dashboards, exploração de dados e alertas de forma rápida, com grande variedade de visualizações e integração nativa com o TimescaleDB.
- **Tecnologias maduras e abertas:** as quatro ferramentas possuem comunidades ativas, documentação ampla e opções de implantação em infraestrutura própria ou em nuvem, reduzindo o risco de dependência de um único fornecedor.
- **Segurança na borda:** o EMQX centraliza autenticação, autorização e criptografia (TLS) na porta de entrada, simplificando a aplicação de políticas de segurança para dispositivos.

---

## 6. Consequências Negativas (Trade-offs)

- **Maior complexidade operacional:** a stack é composta por quatro componentes distribuídos, que exigem provisionamento, monitoramento, atualização e conhecimento especializado da equipe.
- **Sobrecarga de infraestrutura para volumes pequenos:** o Kafka, em especial, pode ser considerado sofisticado para cargas reduzidas; há custo fixo de recursos (CPU, memória, disco, rede) e de operação mesmo em cenários de baixo volume.
- **Curva de aprendizado:** a equipe precisa dominar conceitos de MQTT (QoS, sessões, tópicos), Kafka (partições, consumer groups, offsets, retenção) e TimescaleDB (hypertables, chunks, compressão, agregações contínuas).
- **Dois sistemas de mensageria em sequência:** a presença de EMQX e Kafka adiciona um salto extra no caminho dos dados, o que aumenta a latência fim a fim e o número de pontos de falha, além de exigir mapeamento correto entre tópicos MQTT e tópicos Kafka e entre os níveis de QoS e as garantias de entrega.
- **Garantias de entrega a serem tratadas:** para evitar perdas ou duplicações entre MQTT, Kafka e banco, será necessário projetar consumidores idempotentes e definir políticas de reentrega, o que aumenta a complexidade do desenvolvimento.
- **Escalabilidade de escrita do TimescaleDB:** por ser baseado em PostgreSQL, o dimensionamento horizontal da escrita é mais restrito do que em soluções nativamente distribuídas; será necessário planejar particionamento, retenção, compressão e, se preciso, estratégias de escala (réplicas, ajuste de hardware ou arquitetura multi-nó).
- **Crescimento de dados e custos de armazenamento:** mesmo com compressão e retenção, o volume histórico tende a crescer; políticas de retenção e agregação precisam ser definidas e revisadas continuamente.
- **Gestão de esquemas e compatibilidade:** a evolução dos formatos de mensagem exige governança (versionamento e validação de esquemas) para não quebrar consumidores existentes.
- **Segurança e conformidade ampliadas:** cada componente adicional amplia a superfície de ataque e exige gestão de credenciais, certificados, controle de acesso, backups e trilhas de auditoria.
- **Limitações do Grafana como interface de negócio:** o Grafana é excelente para monitoração técnica e operacional, mas não substitui uma aplicação de negócio completa (gestão de ordens de serviço, fluxos de aprovação etc.), que poderá demandar desenvolvimento adicional.
- **Dependência de ecossistema e licenciamento:** é necessário acompanhar os modelos de licenciamento e as diferenças entre edições open source e comerciais de cada produto (por exemplo, recursos avançados de clustering, integração e suporte), avaliando impacto de custo e de longo prazo.

---

## 7. Referências

- EMQX — https://www.emqx.io
- Apache Kafka — https://kafka.apache.org
- TimescaleDB — https://www.timescale.com
- Grafana — https://grafana.com
- Formato de ADR (Michael Nygard) — https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions