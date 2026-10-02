# Auditoria do estado atual — Prova Social

Data da auditoria: 02/10/2026
Repositório de código: `NAGCODE-Dev/Prova-Social`
Branch e revisão inspecionadas: `codex/p0-flutter-actions`, `89186ae`
Escopo: inspeção do código, documentação viva e testes não Flutter disponíveis neste ambiente. Flutter não está instalado; nenhum comando Flutter foi executado.

## Conclusão

O P0 ainda não está pronto para uma rodada humana. Há implementação de catálogo paginado, publicação validada no servidor e persistência local de tentativas, mas os fluxos do aplicativo, as migrations recentes e a integração ponta a ponta continuam sem validação no estado atual do código.

A documentação viva confirma que a suíte Flutter e os fluxos integrados são gates pendentes. O diagnóstico do repositório, datado de setembro, já não descreve completamente o código atual: a consulta do catálogo usa questões embutidas e cursor; há migrations posteriores de paginação e validação de publicação.

## Matriz de estado

| Área | Estado | Evidência e limite |
|---|---|---|
| Site e simulador | Funcional no escopo testado | `node --test site/tests/site.test.cjs`: 3 testes aprovados, cobrindo cálculo do simulador, link de download e estilos/contraste. Isso não valida o Flutter Web publicado. |
| Catálogo e busca | Parcial | `lib/core/backend/exam_repository.dart` consulta provas públicas, inclui questões na consulta e pagina por cursor. Busca dados do backend, mas não houve consulta ao Supabase remoto nem confirmação de conteúdo real disponível. |
| Visitante e onboarding | Parcial | Há onboarding e navegação do produto no código. Login contextual, retorno à intenção e comportamento em runtime não foram exercitados. |
| Focus Mode e persistência | Parcial | `lib/features/quiz/quiz_page.dart`, `attempt_draft_store.dart` e `attempt_sync_service.dart` implementam rascunho local, restauração e fila. Não foram testados encerramento do processo, perda de conexão, retomada nem sincronização sem duplicação. O cronômetro pode ser ocultado, mas não foram encontrados os controles configuráveis de tempo total e por questão exigidos no produto. |
| Importação e OCR | Parcial; risco P1 | Existem extração de texto nativo, OCR seletivo, roteador, ordenação heurística de colunas e avaliação de confiança. Não há validação comprovada contra a fixture real de 80 questões nem do ciclo completo até revisão e publicação. A documentação do projeto prioriza estabilizar o parser antes de ampliar o pipeline. |
| Publicação | Parcial | O cliente valida a entrada e `supabase/migrations/20260929110000_harden_exam_publication.sql` valida conteúdo no limite server-side. A migration e o comportamento atual não foram executados nesta auditoria. |
| Tentativas, idempotência e RLS | Parcial | Há migrations e código para persistência/reenvio. O histórico documenta validações SQL anteriores focadas em tentativas; isso não comprova a migration mais recente, concorrência nem integração do cliente no SHA auditado. |
| Discussões e comunidades | Ausente | Não foram encontradas features correspondentes em `lib/features`. |
| Flutter, Android e Flutter Web | Não verificável neste ambiente | Flutter ausente. Analyze, testes Flutter, builds, execução visual, responsividade e acessibilidade do app não foram verificados. |

## Riscos do CI e da distribuição

O workflow `.github/workflows/flutter-ci.yml` configura deploy do site em push para `main`. O job Android também usa `gh release upload --clobber` quando uma release da tag já existe. Esse comportamento permite substituir artefatos publicados e conflita com a diretriz do projeto de não sobrescrever uma release publicada silenciosamente. Rever esse fluxo antes de considerar a distribuição previsível.

## Próximos passos P0

1. Executar, em ambiente compatível com a versão Flutter fixada no workflow e no SHA revisado, os gates de format, analyze, testes, build Web e Browser QA. Os testes escritos não substituem essa execução.
2. Exercitar em aparelho ou Web o fluxo visitante/login e retorno à intenção; abrir prova, responder, sair e retomar; finalizar sem rede; sincronizar após reconexão; conferir que não há perda nem duplicação; revisar resultado e procedência.
3. Validar as migrations recentes e a concorrência de entrega em banco isolado. Separar esses resultados de verificações históricas ou de smoke tests em outros SHAs.
4. Revisar o uso de `--clobber` e o gatilho de deploy antes da distribuição.
5. Só depois dos gates e fluxos acima aprovados, marcar o P0 como pronto para rodada humana. Importação com fixture de 80 questões continua sendo prioridade P1.

## Limites desta auditoria

- Não executei Flutter ou Dart tests; Flutter não está disponível e respeitei essa restrição.
- Rodei apenas `node --test site/tests/site.test.cjs`; resultado: 3 aprovados, 0 falhas.
- Não acessei o Supabase remoto, não apliquei migrations e não fiz builds ou testes de browser do aplicativo.
- Código e testes presentes indicam implementação, mas não contam como aprovação de runtime.
- Não alterei o repositório do aplicativo. A alteração preexistente em `analysis_options.yaml` foi preservada.
