# Resumo do Projeto

Este documento fornece uma visão geral completa do projeto GreenThumb.

## Visão

O GreenThumb é um **sistema de produção vegetal em ambiente controlado** que combina um controlador Raspberry Pi, sensores ambientais, automação e um backend em nuvem para operar estufas de forma confiável e coletar dados de cultivo consistentes.

## Status Atual

### Fase de Pesquisa (2025-2026)

O sistema está sendo desenvolvido como parte de um projeto de iniciação científica PIBITI de 12 meses no Insper.

**Objetivos Principais:**

1. Construir uma plataforma confiável de cultivo em ambiente controlado
2. Possibilitar o controle das variáveis ambientais por meio de sensores e automação
3. Coletar dados de cultivo consistentes e bem estruturados
4. Validar o sistema com um protótipo físico

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

1. **Múltiplas Estufas**: Suportar o gerenciamento de mais de uma unidade de cultivo
2. **Automação Aprimorada**: Refinar o controle e o monitoramento ambiental

## Contato

- **Desenvolvedor**: Henrique Bucci R. Netto
- **Email**: henriquebrn@al.insper.edu.br
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)

---

*Última atualização: Fevereiro de 2026*
