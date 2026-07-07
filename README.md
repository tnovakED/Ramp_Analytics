# Ramp Analytics

Pipeline de dados para monitoramento de operações aeroportuárias, construído com arquitetura medalhão (Bronze → Silver → Gold) no Databricks.

## Sobre o projeto

Operações de rampa geram um grande volume de dados operacionais (itens restritos, carga/bagagem, alocação de staff) que raramente são consolidados em uma base histórica confiável. Este projeto nasceu para resolver esse problema: transformar registros operacionais soltos em uma base analítica consistente, capaz de responder perguntas como *"esse padrão está dentro do esperado ou é uma exceção?"*.

O resultado final é uma base de dados em Delta Lake, pronta para consumo em ferramentas de BI, com histórico consistente e lógica de negócio bem definida por camada.

## Arquitetura

O pipeline segue o padrão de arquitetura medalhão, com responsabilidades bem separadas por camada:

| Camada | Responsabilidade |
|---|---|
| **Bronze** | Espelho fiel dos dados brutos de origem (JSON), ingeridos via Autoloader, sem transformação de negócio |
| **Silver** | Normalização de schema, explosão de estruturas aninhadas, deduplicação via MERGE |
| **Gold** | Tabelas de negócio, com regras e agregações prontas para consumo analítico |

### Por que essa estrutura

- **Rastreabilidade**: a camada Bronze preserva o dado exatamente como chegou, permitindo auditoria e reprocessamento a qualquer momento
- **Isolamento de problemas**: separar normalização (Silver) de regra de negócio (Gold) facilita identificar em qual etapa um erro de dado se origina
- **Carga incremental**: as tabelas Gold usam seu próprio histórico (`MAX(data_processamento)`) como referência para processar apenas dados novos, com escrita idempotente via MERGE
- **Flexibilidade futura**: a camada Gold não fixa filtros de unidade operacional, permitindo expansão para múltiplas bases sem refatorar a lógica de negócio

## Stack técnica

- **Databricks** (Free/Serverless Edition)
- **PySpark** para processamento distribuído
- **Delta Lake** + **Unity Catalog** para armazenamento e governança
- **Autoloader** (`cloudFiles`) para ingestão incremental de arquivos
- **Databricks Workflows** para orquestração (Bronze → Silver em paralelo → Gold)
- **Power BI** como camada de visualização, conectado via SQL Warehouse Serverless

## Estrutura do repositório

```
├── notebooks/
│   ├── bronze/          # Ingestão via Autoloader
│   ├── silver/          # Normalização e deduplicação
│   └── gold/            # Agregações de negócio
├── workflows/           # Definições de orquestração
└── docs/                # Documentação adicional
```

## Status

Pipeline Bronze, Silver e Gold implementado e funcional. Próxima etapa: consolidação do dashboard em Power BI.

## Observações

Este repositório contém apenas código e estrutura do pipeline. Nenhum dado real, sensível ou identificável de operações é versionado ou exposto aqui.