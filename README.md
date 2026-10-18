# Dominando o Armazenamento na Azure

Este repositório reúne um resumo dos conceitos estudados no laboratório **Dominando o Armazenamento na Azure**, da **DIO (Digital Innovation One)**.

O objetivo é compreender como escolher, configurar e proteger serviços de armazenamento no Microsoft Azure, considerando **durabilidade, redundância, disponibilidade, desempenho, segurança e custos**.

> Este documento é um resumo de estudos. Os exemplos de configuração são ilustrativos e não representam necessariamente recursos provisionados durante o laboratório.

## Objetivos do desafio

- Conhecer os tipos de armazenamento oferecidos pela Azure.
- Entender contas de armazenamento e seus principais serviços.
- Comparar opções de redundância de dados.
- Diferenciar camadas de acesso e seus custos.
- Compreender controles de segurança e autorização.
- Conhecer ferramentas de transferência e migração.
- Organizar as decisões para o uso de armazenamento na nuvem.

## Nove conceitos fundamentais de armazenamento

### 1. Storage Account

A **conta de armazenamento** é o recurso usado para agrupar e gerenciar serviços de armazenamento no Azure. Sua configuração envolve escolhas como região, redundância, desempenho e segurança.

Uma conta de armazenamento pode oferecer serviços como Blob Storage, Azure Files, Queue Storage e Table Storage, dependendo do tipo de conta e das funcionalidades disponíveis.

### 2. Azure Blob Storage

Serviço de armazenamento de **objetos**, adequado para arquivos não estruturados, como imagens, vídeos, backups, logs e arquivos de dados.

Organização lógica:

```text
Storage Account
└── Container
    ├── imagens/foto.png
    ├── documentos/relatorio.pdf
    └── dados/eventos.json
```

Os principais tipos de blobs são **Block Blobs**, **Append Blobs** e **Page Blobs**.

### 3. Azure Files

Oferece compartilhamentos de arquivos gerenciados, acessíveis por protocolos como **SMB** e, em cenários suportados, **NFS**.

Pode ser útil para arquivos compartilhados entre servidores, aplicações legadas e migrações de file servers.

### 4. Azure Queue Storage

Disponibiliza filas para comunicação assíncrona entre componentes de sistemas.

Exemplo:

```text
Aplicação → Fila → Worker → Processamento
```

Essa separação ajuda a desacoplar serviços e lidar com picos de demanda. A aplicação precisa tratar possíveis reentregas e processamento duplicado.

### 5. Azure Table Storage

Serviço NoSQL orientado a chave para armazenar dados estruturados e não relacionais.

Organiza entidades usando **PartitionKey** e **RowKey**, permitindo consultas eficientes quando o modelo de acesso é bem planejado.

### 6. Redundância dos dados

A Azure oferece estratégias para proteger os dados contra falhas:

| Opção | Conceito |
| --- | --- |
| **LRS** | Cópias sincronizadas em uma única localização física da região |
| **ZRS** | Cópias sincronizadas entre zonas de disponibilidade na região |
| **GRS** | LRS na região primária com replicação assíncrona para região secundária |
| **GZRS** | ZRS na região primária com replicação assíncrona para região secundária |
| **RA-GRS / RA-GZRS** | Opções com acesso de leitura à região secundária, quando disponíveis |

A escolha depende dos requisitos de continuidade, recuperação e orçamento. A replicação geográfica assíncrona pode apresentar atraso; redundância não substitui backups nem proteção contra exclusões acidentais.

### 7. Camadas de acesso do Blob Storage

As camadas ajudam a equilibrar frequência de acesso e preço:

| Camada | Uso típico |
| --- | --- |
| **Hot** | Dados acessados frequentemente |
| **Cool** | Dados acessados com menor frequência |
| **Cold** | Dados pouco acessados, com retenção mais longa |
| **Archive** | Dados raramente acessados, com recuperação que exige reidratação |

O custo não depende apenas da capacidade ocupada: operações, transferência, recuperação e período mínimo de retenção também podem influenciar. A disponibilidade das camadas varia conforme o tipo de conta e o recurso.

### 8. Segurança e controle de acesso

Boas práticas:

- Preferir **Microsoft Entra ID** e **Azure RBAC** para autorização quando suportados.
- Aplicar o princípio do menor privilégio.
- Evitar acesso público desnecessário a contêineres e dados.
- Utilizar HTTPS e conexões privadas quando apropriado.
- Controlar a emissão e validade de **Shared Access Signatures (SAS)**.
- Considerar versionamento, soft delete e políticas de imutabilidade.
- Usar criptografia em repouso e monitoramento de operações.

### 9. Ferramentas de transferência e migração

Algumas ferramentas disponíveis:

- **Azure Storage Explorer:** interface gráfica para administrar e movimentar dados.
- **AzCopy:** utilitário de linha de comando para copiar arquivos e blobs.
- **Azure File Sync:** sincronização de compartilhamentos do Azure Files com servidores Windows.
- **Azure Data Box:** transferência de grandes volumes de dados por meio de dispositivos físicos, em cenários aplicáveis.
- **Azure Migrate:** planejamento e avaliação de cenários de migração, conforme a carga.

## Exemplo de decisão técnica

Imagine um sistema que armazena imagens enviadas por usuários e mantém arquivos antigos para auditoria.

Uma arquitetura simplificada pode ser:

```text
Aplicação
   ↓
Azure Blob Storage
   ├── Container: uploads
   │   └── Camada Hot
   └── Container: historico
       └── Política de gerenciamento de ciclo de vida
             Hot → Cool → Cold/Archive
```

A escolha das transições depende da frequência de acesso, da necessidade de recuperação rápida e dos custos. A camada Archive não deve ser escolhida para arquivos que precisam ser lidos imediatamente.

## Gerenciamento do ciclo de vida

As políticas de **Lifecycle Management** ajudam a automatizar mudanças de camada ou exclusões de objetos conforme condições definidas.

Exemplo: mover blobs pouco acessados para uma camada mais econômica e eliminar arquivos após o período de retenção exigido.

Antes de implementar uma regra, é importante analisar obrigações de retenção, versionamento e custos envolvidos.

## Monitoramento e otimização de custos

O dimensionamento deve considerar:

- volume total armazenado;
- crescimento mensal dos dados;
- quantidade de requisições e operações;
- frequência de leitura e gravação;
- transferência de dados;
- redundância escolhida;
- camadas de acesso;
- retenção e exclusão.

Serviços como **Azure Monitor**, **Azure Cost Management** e métricas das contas de armazenamento ajudam a identificar comportamentos inesperados e oportunidades de otimização.

## Boas práticas

1. Separar ambientes e dados conforme os requisitos de segurança e operação.
2. Definir a redundância de acordo com os objetivos de recuperação.
3. Restringir acessos e evitar compartilhamento de credenciais.
4. Configurar proteção contra exclusão e versionamento quando necessário.
5. Criar políticas de ciclo de vida baseadas no padrão real de acesso.
6. Monitorar capacidade, operações, disponibilidade e custos.
7. Testar procedimentos de restauração e recuperação.
8. Documentar a finalidade de cada conta e contêiner.
9. Excluir recursos de estudo desnecessários para evitar cobranças.

## Conclusão

Armazenar dados na Azure envolve mais do que enviar arquivos para a nuvem. É necessário selecionar o serviço adequado, controlar acessos, planejar a redundância, acompanhar o crescimento dos dados e manter os custos compatíveis com os requisitos da aplicação.

O principal aprendizado é que **cada carga de trabalho precisa de uma estratégia própria de armazenamento**.

## Referências

- [Visão geral do Azure Storage](https://learn.microsoft.com/pt-br/azure/storage/common/storage-introduction)
- [Introdução ao Blob Storage](https://learn.microsoft.com/pt-br/azure/storage/blobs/storage-blobs-introduction)
- [Azure Files](https://learn.microsoft.com/pt-br/azure/storage/files/storage-files-introduction)
- [Redundância do Azure Storage](https://learn.microsoft.com/pt-br/azure/storage/common/storage-redundancy)
- [Camadas de acesso do Blob Storage](https://learn.microsoft.com/pt-br/azure/storage/blobs/access-tiers-overview)
- [AzCopy](https://learn.microsoft.com/pt-br/azure/storage/common/storage-use-azcopy-v10)
- [Azure Storage Explorer](https://learn.microsoft.com/pt-br/azure/vs-azure-tools-storage-manage-with-storage-explorer)
- [Documentação da DIO](https://www.dio.me/)

---

**Desafio de Projeto — DIO**
