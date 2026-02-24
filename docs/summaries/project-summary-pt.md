# Resumo do Projeto

Este documento fornece uma visão geral completa do projeto GreenThumb.

## Visão

O GreenThumb tem como objetivo criar uma **plataforma modular e escalável para agricultura em ambiente controlado**, possibilitando a coleta massiva de dados para otimização de cultivos baseada em aprendizado de máquina. Cada unidade de cultivo atua como um nó modular de um cluster de coleta, gerando datasets fenotípicos padronizados.

## Status Atual

### Fase de Pesquisa (2025-2026)

**Título**: Plataforma Modular e Escalável de Agricultura em Ambiente Controlado para Coleta de Dados Fenotípicos

**Resumo**: O desenvolvimento de modelos de aprendizado de máquina para otimização de cultivos enfrenta um desafio fundamental na escassez de datasets fenotípicos padronizados e de alta qualidade. Este trabalho propõe o desenvolvimento de um sistema distribuído para produção vegetal em ambiente controlado, deslocando o foco da infraestrutura física isolada para uma arquitetura orientada à coleta massiva de dados e à escalabilidade horizontal. O objetivo principal é consolidar uma plataforma onde cada unidade de cultivo atue como um nó modular de um cluster de coleta, possibilitando controle total das variáveis ambientais e a geração de alto volume de dados padronizados. A metodologia fundamenta-se na orquestração de microsserviços via contêineres em computadores de placa única, garantindo a paridade do ambiente de software entre nós, aliada à instrumentação com sensores de baixo custo, automação do controle ambiental e visão computacional para monitoramento contínuo. O sistema permite controle remoto inicialmente manual, estabelecendo a base para futura substituição por agentes de aprendizado de máquina. Até o final do projeto, será feito um protótipo físico para validar com o funcionamento do sistema.

**Objetivos Principais:**

1. Consolidar uma plataforma onde unidades de cultivo atuem como nós modulares de coleta de dados
2. Possibilitar controle total das variáveis ambientais
3. Gerar alto volume de dados fenotípicos padronizados
4. Estabelecer base para controle futuro por agentes de ML

### Trabalho Concluído

- ✅ Deploy no Raspberry Pi 5 com Docker Compose
- ✅ Pipeline CI/CD (GitHub Actions → Docker Hub → Watchtower)
- ✅ Integração de sensores I2C (AHT10, BMP280, TSL2561)
- ✅ Banco de dados PostgreSQL para armazenamento
- ✅ API REST FastAPI com gerenciamento centralizado de dispositivos
- ✅ Streaming de vídeo ao vivo
- ✅ Biblioteca compartilhada (`greenthumb-core`)
- ✅ Cliente controlador com loop Sense-Think-Act
- ✅ Sistema de atuadores (LED RGB, bomba d'água)
- ✅ Modo de segurança e mecanismo de heartbeat

### Em Andamento

- 🔄 Construção do protótipo físico da estufa
- 🔄 Integração de sensores de pH e CE
- 🔄 Sincronização com nuvem (Supabase + Cloudflare R2)
- 🔄 Visão computacional para análise de crescimento

## Stack Tecnológica

| Componente | Tecnologia |
|------------|------------|
| Controlador | Raspberry Pi 5 |
| Linguagem | Python 3.11+ |
| Framework Web | FastAPI |
| Banco de Dados | PostgreSQL 17 |
| ORM | SQLModel |
| Containers | Docker Compose |
| CI/CD | GitHub Actions → Docker Hub |

### Sensores

- **AHT10**: Temperatura e umidade
- **BMP280**: Pressão atmosférica e temperatura
- **TSL2561**: Intensidade luminosa
- **Câmera USB**: Fotos das plantas para visão computacional

### Atuadores

- **LED RGB**: Iluminação com controle PWM
- **Bomba d'água**: Irrigação com controle PWM

### Planejado

- **Supabase**: Banco de dados PostgreSQL em nuvem
- **Cloudflare R2**: Armazenamento de imagens
- **Visão Computacional**: OpenCV para análise de crescimento
- **Machine Learning**: Modelos de predição de crescimento

## Arquitetura do Sistema

O sistema utiliza uma arquitetura de microsserviços com gerenciamento centralizado de dispositivos:

```
Raspberry Pi 5
├── PostgreSQL (banco de dados)
├── microcontroller-api (API + controle de hardware)
├── controller (cliente com loop Sense-Think-Act)
├── cron (tarefas agendadas)
└── watchtower (atualizações automáticas)
```

Todos os serviços rodam em containers Docker e compartilham uma rede comum.

## Organização dos Repositórios

| Repositório | Propósito |
|-------------|-----------|
| `greenthumb-core` | Biblioteca Python compartilhada |
| `rasp5` | Deploy do Raspberry Pi 5 |
| `microcontroller-api-client` | Scripts de controle |
| `database` | Esquemas do banco de dados |
| `cron` | Tarefas agendadas |
| `docs` | Esta documentação |
| `research` | Artigos de pesquisa |

## Coleta de Dados

O sistema coleta:

- **Dados de sensores** via API sob demanda
- **Fotos** para análise de visão computacional

Os dados são armazenados localmente e serão sincronizados com a nuvem para treinamento de ML.

## Objetivos de Longo Prazo

1. **Múltiplas Estufas**: Escalar para gerenciar múltiplos nós de cultivo
2. **Geração de Datasets para ML**: Produzir datasets fenotípicos robustos para treinamento de modelos
3. **Controle por Agentes de ML**: Substituir controle manual por agentes treinados
4. **Produto Comercial**: Desenvolver um kit de estufa reproduzível e vendável

## Contato

- **Desenvolvedor**: Henrique Bucci R. Netto
- **Email**: henriquebrn@al.insper.edu.br
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)

---

*Última atualização: Fevereiro de 2026*
