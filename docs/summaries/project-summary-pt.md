# Resumo do Projeto

Este documento apresenta uma visão geral do projeto GreenThumb.

## Visão

O GreenThumb é um **sistema de produção vegetal em ambiente controlado** que combina um controlador Raspberry Pi, sensores ambientais, automação e um backend em nuvem para operar estufas de forma confiável e coletar dados de cultivo consistentes.

## Status Atual

### Origem

O GreenThumb começou como um projeto PIBITI de iniciação tecnológica de 12 meses no Insper (2025–2026). A fase de pesquisa terminou com o relatório final em agosto de 2026; o desenvolvimento do sistema continua.

### O que existe hoje

- Nó de borda em um Raspberry Pi 5, executado com Docker Compose (PostgreSQL, API do dispositivo, controlador, dashboard local, Watchtower)
- Drivers de sensores: AHT10, BMP280, TSL2561, DS18B20, boia de nível, sondas de pH e TDS/CE, câmera USB
- Atuadores acionados por relé: luz de cultivo, exaustor, bombas d'água e de ar, bombas dosadoras peristálticas
- Controlador com regras por limiar, agenda, intervalo e após-atuador, limites de segurança e watchdog de heartbeat (modo de segurança)
- Fotos periódicas e transmissão ao vivo da câmera
- Sincronização offline-first com uma nuvem própria (PostgreSQL 17 + TimescaleDB; fotos no Cloudflare R2)
- API na nuvem, serviços de autenticação e dashboard administrativo (ainda não públicos)
- CI/CD: GitHub Actions → Docker Hub

### Próximos passos

- 🔄 Finalização do protótipo físico
- 🔄 Calibração das bombas dosadoras e das sondas de pH e TDS
- Primeiro ciclo de cultivo (tomate cereja)

### Planejado

- **Visão Computacional**: OpenCV para análise de crescimento
- **Machine Learning**: Modelos de predição de crescimento

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
| Serviços na nuvem | FastAPI; Java (Spring Boot, Spring Cloud Gateway) |
| Dashboards | React |
| Dados na nuvem | PostgreSQL 17 + TimescaleDB; Cloudflare R2 (fotos) |
| Rede | WireGuard |

### Sensores

- **AHT10**: Temperatura e umidade
- **BMP280**: Pressão atmosférica e temperatura
- **TSL2561**: Intensidade luminosa
- **DS18B20**: Temperatura da água
- **Boia de nível**: Nível do reservatório (cheio ou vazio)
- **Sondas de pH e TDS/CE**: pH e condutividade da solução nutritiva
- **Câmera USB**: Fotos das plantas para visão computacional

### Atuadores

- **Luz de cultivo, exaustor, bombas d'água e de ar**: acionados por relé
- **Bombas dosadoras peristálticas**: dosagem de nutrientes e pH com limites de segurança

## Arquitetura do Sistema

O sistema utiliza uma arquitetura de microsserviços com gerenciamento centralizado de dispositivos:

```
Raspberry Pi 5
├── PostgreSQL (banco de dados)
├── microcontroller-api (API + controle de hardware)
├── controller (cliente com loop Sense-Think-Act)
├── local-dashboard (SPA React)
└── watchtower (atualizações automáticas)
```

Todos os serviços rodam em containers Docker e compartilham uma rede comum.

Uma nuvem própria (API, gateway, serviços de autenticação, dashboard administrativo, PostgreSQL + TimescaleDB) recebe os dados sincronizados. Veja [Cloud Backend](../components/cloud.md).

## Organização dos Repositórios

Veja [Repositórios](../architecture/repositories.md). A maioria dos repositórios é privada; esta documentação e o perfil da organização são públicos.

## Coleta de Dados

O sistema coleta:

- **Dados de sensores**, registrados periodicamente
- **Fotos**, capturadas periodicamente para futuros trabalhos de visão computacional

Os dados são armazenados primeiro no nó e sincronizados com a nuvem quando há conexão.

## Objetivos de Longo Prazo

1. **Múltiplos nós**: a nuvem já registra e lista vários dispositivos; o cadastro autônomo de novos nós ainda está por vir.
2. **Automação Aprimorada**: Refinar o controle e o monitoramento ambiental

## Contato

- **Desenvolvedor**: Henrique Bucci R. Netto
- **GitHub**: [GreenThumbProject](https://github.com/GreenThumbProject)
