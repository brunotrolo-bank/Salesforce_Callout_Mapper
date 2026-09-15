# Checklist de template limpo (Fase 3)

Use para comparar as 5 variações geradas e escolher a mais limpa, e depois
para validar a versão final antes de seguir para a Fase 4.

## Frontmatter

- [ ] `name` em kebab-case, sem espaços, condizente com o diretório.
- [ ] `description` descreve o QUÊ e o QUANDO (gatilhos concretos), não só
      um resumo genérico. Uma boa descrição permite que outra pessoa decida
      se deve invocar a skill/agente sem ler o corpo inteiro.
- [ ] Nenhum campo de frontmatter inventado que o Claude Code não reconhece.
- [ ] Para agentes: `tools` lista apenas o que foi validado em uso real —
      nunca copiado de outro agente "por garantia".

## Corpo do documento

- [ ] Sem seções vazias ou placeholders esquecidos (`<...>` residual).
- [ ] Sem instruções que já são comportamento padrão do Claude Code (não
      reexplique como usar Read/Edit/Bash — só o que é específico desta
      skill/agente).
- [ ] Passo a passo é determinístico onde possível; julgamento do modelo
      é reservado só para onde realmente precisa.
- [ ] Toda ferramenta/variável de ambiente/dependência externa citada tem
      uma verificação de existência antes do uso, não uma suposição.
- [ ] Falhas encontradas durante a validação manual (Fase 2) estão
      documentadas como regras explícitas, não perdidas no histórico do chat.

## Sinal de "template estável" (para parar de iterar)

Pare de gerar novas variações quando, entre uma rodada e a próxima:
- a estrutura de seções não muda;
- a lista de ferramentas não muda;
- só ajustes de texto/wording aparecem.

Isso normalmente acontece em 1–2 rodadas de refinamento, não exige rodar
o ciclo de 5 variações indefinidamente.
