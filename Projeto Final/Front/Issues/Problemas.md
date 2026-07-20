
- Filtro de salas mockado 

- Bug real no backend mapa-sala — ao abrir o mapa como aluno (não admin), a chamada quebrava com 500 "Ocorreu um erro inesperado.". Causa: HttpHubClassAdapter.getClassIdForUser() (mapa-sala-backend) esperava que GET /users/{id}/class no core devolvesse um UUID isolado, mas o core devolve uma lista (List<ClassMembershipResponse> — suporte a múltiplas matrículas). A desserialização Jackson estourava MismatchedInputException. Corrigi para ler a lista e pegar a primeira turma (HubClassMembershipResponse, novo DTO), mantendo a assinatura da interface. Rebuildei a imagem Docker e reiniciei o container mapa — validei via docker logs que o erro sumiu e testei com token real do aluno.

- O problema não era um bug de dados — a alocação sempre esteve certa (Aluno Padrão no assento 1, Representante no assento 2, com selectedStudentId vindo corretamente do sub do JWT). O bug era de cor: em packages/mapa-salas/src/components/SeatCard/SeatCard.tsx, os estados occupied (qualquer colega alocado) e selected (o próprio usuário) usavam tons quase idênticos da mesma família de azul (interactive-default #01258F vs interactive-pressed #011142) — visualmente indistinguíveis, então qualquer assento ocupado parecia "o ponto azul".

Troquei:
- occupied → text-text-primary (neutro/escuro, não azul)
- selected → text-interactive-default (o azul de marca, agora exclusivo do seu próprio assento)






- 1. Dados de runtime (sem mudança de código)

Feito via API, não vive em nenhum arquivo — se resetarem os containers/banco isso se perde:
- Sala 101 vinculada ao layout_template "Lab Eletrotécnica 9x5" (Template E)
- Sala 201 vinculada ao layout_template "2º Pavimento 9x4" (Template A)
- room_map criado para turma MIDS1 nas salas 101 e 201, com Aluno Padrão e Representante Turma alocados nos assentos 1 e 2 de cada uma
- Task sugerida: decidir se isso deve virar seed automatizado (tipo o DevDataInitializer) em vez de setup manual via API, para não se perder a cada reset de ambiente.

2. Front-end — filtro de sala/turma (packages/mapa-salas)

Arquivos novos:
- types/hub.ts — tipos HubRoom/HubClass
- services/server/hubOptionsService.ts — busca /hub/rooms e /hub/classes
- services/client/hubOptionsClient.ts — mapeia para RoomFilterOption
- apps/root/src/app/api/mapa-salas/rooms/route.ts e .../turmas/route.ts — BFF novo

Arquivo alterado: pages/PageMapaSalasContent.tsx (removido MOCK_ROOMS/MOCK_TURMAS, agora busca via useEffect)

Tasks sugeridas:
- Substituir esse wiring temporário quando a RoomFilterBar oficial do squad (Figma 197-3019) sair do papel — tudo já está marcado com comentário TEMP(mapa-salas) para achar fácil.
- Hoje o filtro lista todas as salas ativas do Hub, mesmo as que não têm mapa configurado para a turma do aluno (ele pode selecionar uma sala sem mapa e cair na mensagem de "não configurado"). Vale filtrar só salas com mapa existente para a turma, cruzando com GET /api/mapas?turmaId=.
- listRoomMapsClient/BFF mapas/route.ts não repassam turmaId/salaId como filtro (só page/size) — precisa se for usar essa listagem para o filtro acima.

3. Front-end — cor do assento (packages/mapa-salas/src/components/SeatCard/SeatCard.tsx)

occupied (colega alocado) e selected (o próprio usuário) usavam tons quase idênticos de azul (interactive-default vs interactive-pressed). Troquei occupied para text-text-primary (neutro) e mantive selected em text-interactive-default, deixando o azul exclusivo do assento do usuário logado.

Task sugerida: validar com o squad de design se text-text-primary é o tom certo para "ocupado" no Design System, ou se merece um token dedicado (o comentário antigo já apontava "não há token dedicado a seleção no DS").

4. Backend — bug de integração mapa-sala-backend ↔ core-backend

Arquivo HttpHubClassAdapter.java (getClassIdForUser): esperava um UUID isolado de GET /users/{id}/class, mas o core devolve uma lista (List<ClassMembershipResponse>, suporte a múltiplas matrículas). Isso quebrava com 500 toda vez que um STUDENT/REPRESENTATIVE tentava ver o próprio mapa de sala (admin não era afetado porque não passa por essa checagem).

Corrigido: novo DTO HubClassMembershipResponse + adapter agora lê a lista e pega a primeira turma, mantendo a assinatura UUID getClassIdForUser(UUID userId).

Tasks sugeridas:
- Importante: essa correção mantém a semântica antiga de "primeira turma do usuário" — funciona para os seeds atuais (cada aluno em 1 turma), mas quebra a garantia real de autorização se um aluno estiver em mais de uma turma (ex.: o teacher1 do seed está em 2). O uso correto seria existsMembership(userId, classId) em vez de "pegar a turma", e os 3 call sites (RoomMapViewAuthorizationService, ListRoomMapHistoryUseCase, ListRoomMapsUseCase) merecem revisão — hoje todos assumem 1 turma por usuário.
- Rebuild + restart do container mapa já foi feito para aplicar o fix — só formalizar isso num commit/PR.