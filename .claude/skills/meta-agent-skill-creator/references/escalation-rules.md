# Regras de escalonamento (Fase 6)

Estas regras devem ser incorporadas (adaptadas ao contexto específico) em
todo agente/skill gerado por esta meta-skill.

## Classificação de falhas

**Recuperável** — a própria execução tenta resolver sem parar tudo:
- Arquivo de config existe mas está desatualizado -> reler/regerar.
- Ferramenta MCP não respondeu na primeira tentativa -> uma nova tentativa,
  não um loop infinito.

**Crítica** — bloqueia a execução, precisa de intervenção humana:
- Permissão de terminal/ferramenta negada pelo usuário.
- Variável de ambiente ou API key ausente/inválida.
- Dependência que não pode ser instalada ou resolvida automaticamente.
- Diretório/arquivo esperado que não existe e não pode ser criado com
  segurança (ambiguidade sobre o que deveria estar lá).

## Comportamento obrigatório numa falha crítica

1. Nunca continuar como se nada tivesse acontecido nem inventar um valor
   plausível para contornar a falta de informação.
2. Reportar, na própria conversa, de forma específica:
   - o que estava sendo feito quando travou;
   - o comando/arquivo/ferramenta exato que falhou;
   - o que o usuário precisa fornecer ou corrigir para destravar.
3. Se (e somente se) houver um canal de notificação externo já configurado
   e disponível na sessão (ex.: um MCP server de mensageria como WhatsApp/
   Evolution API, listado nas ferramentas disponíveis), oferecer/usar esse
   canal para alertar sobre a falha crítica — formulando a mensagem para
   ser lida por um humano fora do terminal (curta, específica, acionável).
4. Nunca tentar configurar, instalar ou inferir credenciais de um canal de
   notificação que não esteja já disponível na sessão. A ausência de canal
   de alerta externo é uma condição normal, não um erro adicional a reportar.
5. Nunca embutir tokens, chaves de API ou números de telefone diretamente
   no código ou no `SKILL.md` gerado — sempre referenciar variáveis de
   ambiente ou arquivos de configuração já existentes no projeto.
