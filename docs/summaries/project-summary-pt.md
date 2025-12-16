# Resumo do Projeto

Este documento fornece uma visão geral completa do projeto GreenThumb.

## Visão

O GreenThumb tem como objetivo criar um **ecossistema de estufa automatizada com inteligência artificial** que estabelece ambientes de cultivo ideais para qualquer espécie de planta, maximizando a eficiência de crescimento e minimizando o consumo de recursos.

## Status Atual

### Fase de Pesquisa (2025)

Estamos desenvolvendo uma **plataforma hidropônica de baixo custo** para cultivo de tomate-cereja como parte de um projeto de pesquisa PIBITI de 12 meses na universidade.

**Objetivos Principais:**

1. Construir uma mini-estufa com controle ambiental automatizado
2. Implementar monitoramento contínuo (pH, CE, temperatura, umidade, luz)
3. Desenvolver visão computacional para análise do crescimento das plantas
4. Validar o sistema através de um ciclo completo de cultivo

### Trabalho Concluído

- ✅ Deploy no Raspberry Pi 5 com Docker Compose
- ✅ Integração de sensores I2C (AHT10, BMP280, TSL2561)
- ✅ Banco de dados PostgreSQL para armazenamento
- ✅ API REST FastAPI com streaming de vídeo ao vivo
- ✅ Pipeline CI/CD com GitHub Actions
- ✅ Atualização automática de containers via Watchtower
- ✅ Biblioteca compartilhada (`greenthumb-core`)

### Em Andamento

- 🔄 Construção física da estufa
- 🔄 Integração de sensores de pH e CE
- 🔄 Controle PWM de LEDs e bomba d'água
- 🔄 Sincronização com nuvem (Supabase + Cloudflare R2)

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

### Planejado

- **Supabase**: Banco de dados PostgreSQL em nuvem
- **Cloudflare R2**: Armazenamento de imagens
- **Visão Computacional**: OpenCV para análise de crescimento
- **Machine Learning**: Modelos de predição de crescimento

## Arquitetura do Sistema

O sistema utiliza uma arquitetura de microsserviços:

```
Raspberry Pi 5
├── PostgreSQL (banco de dados)
├── FastAPI (API + dashboard)
├── data_collection (leituras de sensores + fotos)
├── cron (tarefas agendadas)
└── watchtower (atualizações automáticas)
```

Todos os serviços rodam em containers Docker e compartilham uma rede comum.

## Organização dos Repositórios

| Repositório | Propósito |
|-------------|-----------|
| `greenthumb-core` | Biblioteca Python compartilhada |
| `rasp5` | Deploy do Raspberry Pi 5 |
| `database` | Esquemas do banco de dados |
| `cron` | Tarefas agendadas |
| `docs` | Esta documentação |
| `research` | Artigos de pesquisa |

## Coleta de Dados

O sistema coleta:

- **Dados de sensores** a cada 30 minutos
- **Fotos** a cada 4 horas

Os dados são armazenados localmente e serão sincronizados com a nuvem diariamente.

## Objetivos de Longo Prazo

1. **Múltiplas Estufas**: Escalar para gerenciar múltiplas unidades
2. **Otimização com ML**: Usar dados coletados para otimizar condições de cultivo
3. **Produto Comercial**: Desenvolver um kit de estufa reproduzível e vendável
4. **Código Aberto**: Compartilhar conhecimento e ferramentas com a comunidade

## Contato

- **Desenvolvedor**: Henrique Bucci R. Netto
- **Email**: henriquebrn@al.insper.edu.br
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)

---

*Última atualização: Dezembro de 2025*
