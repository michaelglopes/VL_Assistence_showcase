# Sistema de Gestão VL Assistance (Showcase)

![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)
![Vite](https://img.shields.io/badge/vite-%23646CFF.svg?style=for-the-badge&logo=vite&logoColor=white)
![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)
![NodeJS](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white)
![Express.js](https://img.shields.io/badge/express.js-%23404d59.svg?style=for-the-badge&logo=express&logoColor=%2361DAFB)
![Prisma](https://img.shields.io/badge/Prisma-3982CE?style=for-the-badge&logo=Prisma&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)

> **Nota de Visibilidade e Status (Work in Progress):** Este é um repositório de vitrine (showcase). O código-fonte original é mantido em ambiente privado. O sistema encontra-se em **desenvolvimento ativo**. O objetivo desta documentação é expor a arquitetura de software atual, as decisões de engenharia e o roadmap de funcionalidades futuras da plataforma.

---

## Visão Geral e Arquitetura

O **VL Assistance** é um sistema de gestão desenhado para modernizar e estabelecer segurança jurídica no fluxo de entrada e saída de equipamentos em laboratórios de assistência técnica. 

A arquitetura foi estruturada para criar um ecossistema fluido entre a operação administrativa no desktop da bancada e a captura de evidências fotográficas via smartphone, exigindo uma integração cross-device precisa.

### Stack Tecnológico

* **Frontend:** Desenvolvido em React.js empacotado com Vite para otimização de build e agilidade no ambiente de desenvolvimento. Totalmente tipado com TypeScript para mitigar erros em tempo de compilação, especialmente no tráfego de dados de formulários.
* **Estilização e UI:** Implementada com Tailwind CSS, adotando o padrão Glassmorphism para uma interface limpa, dispensando a manutenção de grandes arquivos CSS tradicionais.
* **Backend:** API RESTful construída em Node.js com Express, também estritamente tipada com TypeScript. O recebimento e processamento de arquivos físicos (imagens do check-in) é gerenciado via middleware Multer.
* **Persistência e Banco de Dados:** Banco de dados PostgreSQL hospedado nativamente no Supabase para garantir alta disponibilidade. A modelagem de entidades (ex: Ordens de Serviço) e as transações no banco são abstraídas de forma declarativa pelo Prisma ORM.

---

## Módulos Principais (Em Produção)

### Dashboard Financeiro Integrado
O painel administrativo central consolida a saúde financeira do laboratório em tempo real. O sistema calcula e exibe métricas vitais da operação, como o caixa livre disponível, lucro líquido mensal e executa o provisionamento automático de reservas destinadas à compra de peças.

### Gestão Estrita de Ordens de Serviço (OS)
A funcionalidade core do sistema é a abertura de Ordens de Serviço. O formulário de entrada atua como a primeira camada de sanitização de dados, utilizando validações rigorosas e aplicações de máscaras em tempo real baseadas em expressões regulares (Regex) para CPF, telefones e e-mail. Isso garante que o banco de dados receba apenas informações formatadas e íntegras.

### Check-in Fotográfico Cross-Device
O principal diferencial operacional do VL Assistance. Ao registrar um equipamento, o frontend desktop gera dinamicamente um QR Code na tela (utilizando `react-qr-code`). O técnico escaneia o código com seu smartphone, o que invoca uma interface web mobile isolada. Esta interface ativa a câmera do dispositivo, captura as fotos das condições estruturais do equipamento e realiza o upload direto para o servidor Node.js, onde são automaticamente indexadas ao ID da Ordem de Serviço correspondente no banco de dados.

---

## Estudo de Caso: Resolução de Gargalos em Integração Cross-Device

O maior nível de complexidade técnica do projeto concentrou-se na comunicação bidirecional e na renderização de interfaces entre o desktop local e o dispositivo móvel. Durante o desenvolvimento do check-in fotográfico, dois gargalos críticos de arquitetura foram solucionados.

### 1. Refatoração de Roteamento e Isolamento de Layout (Layout Leak)

**O Problema:** Na iteração inicial, a transição de interfaces baseava-se em controle de estado do React. Quando o smartphone acessava a rota via QR Code, a árvore de componentes montava a aplicação inteira, injetando menus laterais e estruturas administrativas projetadas para desktop na tela do celular, inviabilizando a usabilidade da câmera.

**A Solução:** A arquitetura de navegação foi completamente refatorada utilizando o React Router DOM. Desenvolvi um componente de layout encapsulado para proteger e isolar as rotas administrativas de desktop. A rota de captura mobile foi extraída para o nível raiz da aplicação, garantindo que o smartphone renderizasse um ambiente limpo, em tela cheia, sem herdar a arvore de componentes do painel administrativo.

### 2. Descoberta de Rede e Fallback de Hardware (Network & Camera Access)

**O Problema:** Houve dois impedimentos na comunicação de rede. Primeiro, o gerador de QR Code utilizava `localhost` por padrão, resultando em falha de conexão quando lido pelo smartphone (que buscava o servidor em seu próprio loopback). Segundo, por operar em ambiente local sem certificado SSL (HTTPS), as políticas de segurança dos navegadores mobile bloqueavam o acesso à API de vídeo do HTML5 (`getUserMedia`).

**A Solução:** Para o problema de rede, implementei uma lógica utilizando a API do navegador para capturar o IP local da máquina na rede Wi-Fi dinamicamente, injetando esse endereço real na string de geração do QR Code. 

Para contornar o bloqueio de hardware, criei um sistema de renderização condicional atuando como fallback. Quando o sistema detecta que a API de vídeo do navegador recusa a conexão (devido à falta de HTTPS no ambiente de desenvolvimento local), a aplicação injeta atributos nativos do HTML5 (`<input type="file" accept="image/*" capture="environment">`) para invocar diretamente o aplicativo de câmera original do sistema operacional do smartphone, garantindo o funcionamento fluido da captura em qualquer cenário.

---

## Roadmap e Evolução do Sistema

O escopo do sistema continuará a ser expandido nas próximas sprints, visando transformar o VL Assistance em um ecossistema completo para a bancada técnica. As seguintes implementações já estão mapeadas para desenvolvimento:

* **Integração Fiscal (NFS-e):** Comunicação direta com webservices de prefeituras para automatizar a emissão de Notas Fiscais de Serviço eletrônica no momento do encerramento de uma OS.
* **Automação de Estoque e Compras:** Controle dinâmico de inventário com baixa automática de insumos e peças utilizadas nos reparos, diretamente atrelado ao provisionamento financeiro.
* **Gestão e Monitoramento de Garantias:** Rastreamento estruturado do ciclo de vida dos equipamentos entregues, controlando os prazos de garantia vigentes e extraindo métricas de reentrada por falha de serviço.
* **Arquivo Histórico Imutável:** Retenção a longo prazo de Ordens de Serviço finalizadas, assegurando a integridade dos laudos e das evidências fotográficas para respaldo jurídico em eventuais auditorias.
