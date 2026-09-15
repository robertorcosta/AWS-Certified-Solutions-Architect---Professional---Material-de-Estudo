# Guia do projeto — Material de estudo AWS SAP-C02

Contexto e convenções para trabalhar neste repositório. Leia antes de editar.

## O que é este projeto
Material de estudo em **Português do Brasil** para a certificação AWS Solutions Architect Professional (SAP-C02). É conteúdo estático (HTML/CSS/JS), sem backend.

## Arquivos principais
- `aws-sap-c02-guia.html` — guia completo (fonte da verdade do conteúdo). Seções por domínio (D1–D4), Aprofundamento, Questões menos prováveis, Novidades.
- `aws-sap-c02-mapa.html` — mapa mental "bate-pronto" (nós expansíveis por serviço).
- `aws-sap-c02-notebooklm.md` — versão texto para o NotebookLM.
- `README.md` — descrição do projeto.
- `docs/` — cópia para GitHub Pages. **Sempre manter idêntico** aos arquivos da raiz (guia e mapa).
- `MELHORIAS.md` — notas locais de ideias futuras (não commitado por escolha do dono).
- Artefatos antigos fora do versionamento: `extracted_insights.txt`, `headers.txt`, `pdf_text.txt`, `update_guide.py`. Não commitar sem pedir.

## Regras de edição

### Data de atualização (IMPORTANTE)
Sempre que fizer uma alteração de conteúdo, atualizar a nota do rodapé do guia para **a data atual** (mês e ano):
`Atualizado em <Mês> de <Ano>.` — no `<footer>` de `aws-sap-c02-guia.html` (e refletir em `docs/`).

### Idioma
- Todo o texto corrido em **PT-BR**.
- Nomes de serviços, APIs, recursos e termos técnicos consagrados ficam em **inglês** (ex.: Transit Gateway, AssumeRoleWithSAML, Gateway Endpoint).

### Sincronizar docs/
Depois de editar `aws-sap-c02-guia.html` ou `aws-sap-c02-mapa.html`, copiar para `docs/` antes do commit. Devem ficar byte a byte iguais.

### Validar HTML
Antes de commitar edições em HTML, checar que as tags estão balanceadas (nada de `<div>`/`<details>` sem fechar) e que não sobrou marcador de conflito de merge (`<<<<<<<`).

## Padrões de conteúdo (o que faz este material bom)
- **Foco em julgamento arquitetural**, não decoreba: cenários com trade-offs (custo, latência, RTO/RPO, esforço operacional, compliance).
- **Rodapé "📝 Como cai na prova"** em cada serviço do guia: até 5 cenários no formato *cenário → resposta → porquê* (por que essa e não a distratora).
- **Termos menos comuns** viram link para a doc oficial (classe `.doc-term` no guia).
- **Mapa**: tip **📝 Prova** (classe `.exam-tip`, cor vermelha/accent) separado da dica geral **💡** (classe `.tip`, azul).
- Não copiar enunciado literal de simulados (licenciamento) — sempre parafrasear.

## Princípio "estude para o blueprint, anote a realidade"
O exame testa o guia oficial vigente, não os lançamentos mais recentes. Um serviço novo leva meses a mais de um ano para virar resposta correta e nunca aparece em preview.
- A resposta que o material dá é a que **o exame espera**.
- Quando algo mudou no mundo real, adicionar **nota curta** (não reescrever o cenário).
- Novidades = contexto, não centro do estudo.

## Verificação factual (anti-alucinação)
É material de estudo: precisão importa. Ao adicionar afirmações técnicas (limites, comportamentos, APIs, escopos), confirmar contra a doc oficial da AWS quando houver dúvida. Fatos que já mudaram e devem ser tratados com cuidado:
- **API Gateway REST**: default 29s, mas ajustável até 300s (desde jun/2024).
- **Snowmobile**: descontinuado (mar/2024) — manter só como distrator.
- **Compute Optimizer** enhanced metrics: até 93 dias (default 14).
- **Aurora Global Database**: managed failover/switchover (planejado) vs detach-and-promote (desastre).

## Fluxo de trabalho de commit
- Commits pequenos e descritivos, em PT-BR, agrupados por tema/domínio.
- Fazer push para `main` (o repositório trabalha direto em main).
- Não commitar `MELHORIAS.md` nem os artefatos antigos sem pedido explícito.
- Ao terminar uma rodada de mudanças: sincronizar `docs/`, validar HTML, atualizar a data do rodapé, commitar e dar push.
