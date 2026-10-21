# Assistente de Investimentos com RPA e IA Generativa

Pipeline de automação que extrai clientes de uma página web com Python (RPA), orquestra o processamento no **N8N**, cruza o perfil de cada cliente com opções de investimento e gera **mensagens personalizadas com IA generativa (Gemini)**, enviando o resultado por **e-mail**.

> Projeto desenvolvido para o desafio *Criando um Assistente de Investimentos com RPA e IA Generativa* (DIO).

## Demonstração

- 🎥 Vídeo: https://drive.google.com/file/d/1__Vgzj28eLXT5P3Q3y1HAHJHSDYsBlq9/view?usp=drivesdk




## Tecnologias

| Etapa | Ferramenta | Função |
|-------|-----------|--------|
| Hospedagem | GitHub Pages | Serve a página de clientes e o CSV de investimentos |
| Extração (RPA) | Python + BeautifulSoup | Coleta os dados dos clientes via web scraping |
| Orquestração | N8N Cloud | Recebe os dados, cruza perfis e gera as mensagens |
| IA generativa | AI Agent + Google Gemini | Escreve mensagens personalizadas por cliente |
| Comunicação | Gmail (N8N) | Envia a recomendação por e-mail |

## Como funciona

1. **RPA em Python:** o notebook `src/extrair_clientes.ipynb` acessa a página de clientes, lê a tabela (nome, e-mail, saldo e perfil) e envia a lista em JSON, via `POST`, para o Webhook do N8N.
2. **Webhook (N8N):** recebe `{"clientes": [...]}` e dispara o fluxo.
3. **HTTP Request + Extract from File:** baixa o `data.csv` e transforma cada linha em um item (`perfil`, `produto`, `minimo`, `rentabilidade`).
4. **Code (JavaScript):** para cada cliente, filtra os produtos do perfil dele e mantém apenas os que cabem no saldo.
5. **AI Agent (Gemini):** recebe nome, perfil, saldo e as opções elegíveis e escreve uma mensagem curta, cordial e personalizada.
6. **Edit Fields:** organiza a saída (`nome`, `email`, `perfil`, `mensagem_ia`).
7. **Saídas em paralelo:** o **Gmail** envia o e-mail e o **Respond to Webhook** devolve as mensagens ao Python, que as imprime.

## Decisões técnicas

- **Por que RPA?** O script faz o que uma pessoa faria manualmente (abrir a página, ler a tabela e passar os dados adiante). É útil quando não há API disponível ou ao integrar sistemas legados.
- **Tratamento do saldo:** o saldo chega como texto (`"R$ 12.500,00"`). Antes de comparar com o investimento mínimo, ele é convertido para número removendo símbolos e trocando a vírgula decimal por ponto.
- **Filtro por saldo:** além do perfil, só são sugeridos produtos cujo valor mínimo cabe no saldo do cliente. Se nenhum produto couber, o cliente recebe uma mensagem incentivando a montar uma reserva.
- **MVP estático e versão com IA:** a primeira versão gera mensagens por template, baseadas no perfil. A versão final adiciona o AI Agent para mensagens únicas e mais humanizadas.
- **Segurança da IA:** o *System Message* proíbe prometer retornos e inventar produtos, e exige o aviso de que rentabilidade passada não garante resultado futuro. A IA só pode usar as opções fornecidas.
- **Respond to Webhook em paralelo ao Gmail:** a resposta ao Python não depende do envio de e-mail. Assim, o Python sempre recebe os campos `nome`, `perfil` e `mensagem_ia`.
- **Timeout no Python:** como o agente faz uma chamada de IA por cliente, o fluxo leva de dezenas de segundos a alguns minutos. O `requests.post` usa `timeout=180`.


## Estrutura do repositório

```
📁 dio-lab-assistente-investimentos-rpa-n8n/
├── 📄 README.md
├── 📁 src/
│   └── 📄 extrair_clientes.ipynb    # RPA: extrai clientes e envia ao N8N
├── 📁 n8n/
│   ├── 📄 workflow.json             # Workflow final (IA + e-mail)
│   └── 📄 workflow-mvp.json         # Versão MVP com mensagens estáticas (opcional)
└── 📁 docs/
    ├── 📄 index.html                # Página de clientes
    └── 📄 data.csv                  # Opções de investimento
```




## Dificuldades e aprendizados

- **Test URL x Production URL:** a Test URL só funciona após clicar em *Listen for test event*, e a de produção só com o workflow ativo. Confundir as duas gera erro 404.
- **Respond to Webhook:** o nó Webhook precisa estar configurado em *Using 'Respond to Webhook' Node*; caso contrário o N8N retorna erro (500).
- **Timeout:** a geração das mensagens para todos os clientes demora; foi necessário aumentar o timeout da requisição no Python.

## Referências

- [Documentação do N8N](https://docs.n8n.io/)
- [BeautifulSoup: Web Scraping com Python](https://realpython.com/beautiful-soup-web-scraper-python/)
- [Gemini API: limites de uso](https://ai.google.dev/gemini-api/docs/rate-limits)
- [GitHub Pages: Guia Rápido](https://pages.github.com/)


